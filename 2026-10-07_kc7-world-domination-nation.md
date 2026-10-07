# Tracing a Phishing Campaign to Plink Tunnels: KC7 World Domination Nation

**Date:** October 7, 2026 **Platform:** KC7 (kc7cyber.com) **Domain:** Threat Hunting, KQL, Phishing Analysis

---

## Scenario

Alert on `freebitcoin.exe` created on host `QU09-DESKTOP`. Worked from the alert to the delivery URL, then pivoted through email and DNS data to scope the actor's campaign and find every host that tunneled out to their infrastructure.

## Host Timeline Reconstruction

- Pulled `ProcessEvents` for `QU09-DESKTOP` in a window around the `freebitcoin.exe` creation time (found via `FileCreationEvents`)
- Found a run of `cmd.exe`-spawned recon commands (`ipconfig /all`, `systeminfo`, `net user`, `netstat -ano`, `net view`, `tasklist`, `schtasks /query`, `net localgroup administrators`, a `reg query` of the Run key)
- Later in the window, a `plink.exe` reverse tunnel launched through `powershell.exe`, forwarding the host's RDP port (3389) to an external IP

```
let target = datetime(2023-06-05 13:10:22);
ProcessEvents
| where timestamp between ((target - 1h) .. (target + 1h))
| where hostname == "QU09-DESKTOP"
| sort by timestamp asc
```

- Reasoning: `process_name` was `powershell.exe`, not `plink.exe`. Filtering on `process_name` silently dropped the tunnel rows. Filtering on `process_commandline has "plink"` catches it however it was launched.

## Delivery Confirmation

- A browser likely created the file, so checked outbound browsing for that host just before the file appeared
- `OutboundNetworkEvents` keys on `src_ip`, not hostname, so looked up the host's IP in `Employees` first

```
let targetIP =
    Employees
    | where hostname == "QU09-DESKTOP"
    | project ip_addr;
OutboundNetworkEvents
| where timestamp between ((target - 10m) .. (target + 10m))
| where src_ip in (targetIP)
| project timestamp, src_ip, url
| sort by timestamp asc
```

## Campaign Scope

- Started from one known sender and expanded using the actor's own indicators (subject lines and links) to find every sender tied to the campaign
- Counted distinct recipients with `dcount(recipient)`, not rows, since one person can get several emails
- Ranked subject lines by distinct employees reached to identify the main lure

```
let knownSubjects =
    Email
    | where sender == "freedom@hotmail.com"
    | distinct subject;
let knownLinks =
    Email
    | where sender == "freedom@hotmail.com"
    | distinct link;
let actorSenders =
    Email
    | where subject in (knownSubjects) or link in (knownLinks)
    | distinct sender;
Email
| where sender in (actorSenders)
| summarize Employees = dcount(recipient)
```

- Reasoning: link-only filtering undercounted. Pivoting on both subject and link surfaced additional sender addresses.

## Infrastructure Pivot

- Extracted link domains with `parse_url()`, ranked them by use, then resolved them in `PassiveDns` to find the actor's IPs and every other domain hosted on those IPs

```
let susDomain =
    Email
    | where sender in (actorSenders)
    | where isnotempty(link)
    | extend LinkDomain = tostring(parse_url(link).Host)
    | distinct LinkDomain;
let ActorIps =
    PassiveDns
    | where domain in (susDomain)
    | distinct ip;
PassiveDns
| where ip in (ActorIps)
| summarize TotalDomains = dcount(domain)
```

## Tunneling Hunt

- Searched `ProcessEvents` for plink commands containing any actor IP
- `in (ActorIps)` returned nothing because `in` needs the whole value to equal a list item. `has_any` matches an IP appearing anywhere inside a long command line.

```
ProcessEvents
| where process_commandline has "plink"
| where process_commandline has_any (ActorIps)
```

- To rank actor IPs by number of hosts, pulled every IP out of each command line with `extract_all`, split them into one row per IP with `mv-expand`, then filtered to actor IPs. This drops `0.0.0.0` and `127.0.0.1` from the `-R` forwarding spec.

```
ProcessEvents
| where process_commandline has "plink"
| extend CmdIPs = extract_all(@"(\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})", process_commandline)
| mv-expand ActorIP = CmdIPs to typeof(string)
| where ActorIP in (ActorIps)
| summarize Hosts = dcount(hostname) by ActorIP
| sort by Hosts desc
```

- Flipped the grouping to `by hostname` for a per-host view of how many actor IPs each machine reached
- For a single user, mapped their internal IP to a hostname through `Employees`, then pulled that host's plink commands

## MITRE ATT&CK Mapping

| Tactic              | Technique                                  | Evidence                                              |
| ------------------- | ------------------------------------------ | ----------------------------------------------------- |
| Initial Access      | T1566.002 - Spearphishing Link             | Campaign using shared links and subject lines         |
| Execution           | T1204.002 - User Execution: Malicious File | `freebitcoin.exe` created after browser download      |
| Discovery           | T1082 / T1016 / T1087                      | `systeminfo`, `ipconfig /all`, `net user` on the host |
| Command and Control | T1572 - Protocol Tunneling                 | `plink.exe` reverse tunnel forwarding RDP to external IP |

---

## What I Learned Today

- **Chaining `let` statements:** one `let` can feed the next, building a combined list step by step. Subjects and links fed `actorSenders`, which fed `susDomain`, which fed `ActorIps`, and the final query only had to use the finished list. Each `let` that feeds `in ()` must return a single column (`distinct` or `project`), and every `let` ends with a semicolon. Once the chain exists, a new question only needs a new last few lines.
- **Searching by time:** `datetime()` takes `yyyy-MM-dd HH:mm:ss` in 24-hour time, so 1:10:22 PM is `13:10:22`. An exact `==` on a timestamp rarely hits, so anchor on a time in a `let` and use `between ((target - 1h) .. (target + 1h))` for a window. `ago()` counts from now and is useless on historical data like KC7. Start wide, then narrow the window once the timeline makes sense.
- **Using `summarize`:** the `by` clause decides the question being answered. `dcount(x) by y` answers "how many different x per y," and `count() by y` plus a sort answers "which y most often." Flipping the `by` column turns "hosts per IP" into "IPs per host" without rebuilding the query. `make_set()` lists the values next to the count, and any column not in the `summarize` or `by` is dropped.
- **Attacker infrastructure scopes a campaign better than one sender or subject.** Pivoting sender to indicators to every actor sender to domains to IPs to hosts caught what single-field filters missed.
- **`in` needs an exact match against one value.** For a value buried in a long string, use `has_any`, or extract it first with `extract_all` + `mv-expand` and then use `in`.
- **`process_name` is the launching binary.** When a shell starts the tool, the tool name is only in `process_commandline`.

**Big Picture:** Going from one alert to a full victim list and host list by chaining email, DNS, and process telemetry is the same pivoting a Tier 1 analyst does when scoping a phishing incident.
