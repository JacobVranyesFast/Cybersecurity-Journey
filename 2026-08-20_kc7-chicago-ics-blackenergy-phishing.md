# Critical Compromise in Chicago: Tracing an ICS Intrusion from BlackEnergy to Phishing Origin

**Date:** 2026-08-20
**Platform:** KC7 (kc7cyber.com)
**Domain:** Threat Hunting, KQL, Incident Response, Phishing Analysis

---

## Scenario

Widespread power outage in Chicago, tied to a suspected cyberattack on the city's power grid. Investigated via KQL across process, network, email, and DNS telemetry to trace the attack from SCADA compromise back to initial phishing delivery.

## Malware Identification

- Pivoted on process activity referencing SCADA, found `blackenergy.exe` spawning:
  - `ICSScanner.exe` scanning the internal network for SCADA hosts
  - `curl` pulling a second-stage payload from C2
  - A PsExec loop using stolen credentials to move laterally across discovered SCADA IPs
- All activity traced to host `BDC0-DESKTOP`, user `jisaetang`

```kusto
ProcessEvents
| where process_name =~ "blackenergy.exe" or parent_process_name =~ "blackenergy.exe"
| sort by timestamp asc
```

- Reasoning: initial `has "black"` filter missed a hit where the filename had no path separator (`C:\ProgramDataBlackEnergy.exe`). `has` only matches clean delimited tokens, `contains` catches raw substrings regardless of formatting. Switched to `=~` against `process_name` directly once I had a confirmed clean field to match exactly.

## Delivery Confirmation

- File creation activity around the BlackEnergy timestamp confirmed the payload dropped immediately after `jisaetang` opened a downloaded archive, `Urgent_Cyber_Threat_Alert.zip`
- BlackEnergy beaconed to C2 domain `chicagogridupdates.com` immediately after execution

```kusto
FileCreationEvents
| where hostname == "BDC0-DESKTOP"
| where timestamp >= datetime(2024-08-29T08:28:01Z)
| take 2
```

## Phishing Campaign Scope

- Filtering `Email` on the zip filename or a single subject line only surfaced the final, narrow wave sent directly to `jisaetang` from spoofed internal-looking addresses
- Pivoted on infrastructure instead: found two additional domains (`citygridsolutions.net`, `infrastructurewatch.org`) sharing IPs with the confirmed C2 domain via PassiveDNS

```kusto
PassiveDns
| where ip in ("104.244.42.129", "87.250.252.242")
| distinct domain
```

- Ranked senders by distinct employees targeted across all three domains, using `has_any` since these are substrings inside full URLs, not exact field values

```kusto
Email
| where link has_any ("chicagogridupdates.com", "citygridsolutions.net", "infrastructurewatch.org")
| summarize targeted_employees = dcount(recipient) by sender
| order by targeted_employees desc
```

- Reasoning: an earlier attempt ranking all senders with no infrastructure filter returned a false positive, a legitimate high-volume internal employee, not the attacker. Anchoring on confirmed C2 infrastructure instead of guessing at sender patterns fixed it.

- Flipped the same query to group by recipient instead of sender, to get a full per-victim view (every subject line, link, and sender each employee actually saw)

```kusto
Email
| where link has_any ("chicagogridupdates.com", "citygridsolutions.net", "infrastructurewatch.org")
| summarize targeted_employees = dcount(recipient),
            subjects = make_set(subject),
            recipients = make_set(recipient),
            links = make_set(link)
            by recipient
```

- Joined the recipient list back against `Employees` to attach names, roles, and hostnames to every phished person, since `Email` only has addresses, not job context

```kusto
Email
| where link has_any ("chicagogridupdates.com", "citygridsolutions.net", "infrastructurewatch.org")
| distinct recipient
| join kind=inner Employees on $left.recipient == $right.email_addr
| project recipient, name, role, hostname, username
```

- Reasoning: needed the un-summarized recipient list first since `join` can't match a table's field against an array sitting inside a summarized column. `distinct` first, then join, gets a clean one-row-per-person shape.

## SCADA Operator Targeting

- Filtering `Employees` for `role contains "SCADA"` only returns two people company-wide, one Operator and one Administrator. Dead end for finding a second Operator by job title.

```kusto
Employees
| where role contains "SCADA"
```

- Reasoning: reconnaissance is attacker behavior, not an HR record, so a second recon target isn't necessarily going to show up as a named SCADA role. Pivoted to network telemetry instead, checking whether the attacker's known infrastructure touched more than one internal host before settling on `BDC0-DESKTOP`.

```kusto
let bad_ips = 
    PassiveDns
    | where domain has_any ("chicagogridupdates.com", "citygridsolutions.net", "infrastructurewatch.org")
    | distinct ip;
InboundNetworkEvents
| where src_ip in (bad_ips)
```

- Status: identifying the second recon target is still in progress as of this write-up. Next step is comparing distinct destination hosts from this query against the full employee list to find who else got touched.

## Post-Exploitation Discovery

- Hunted for domain controller enumeration on the compromised host, based on known attacker tradecraft (MITRE T1018 / T1482) rather than guessing at syntax first

```kusto
ProcessEvents
| where hostname == "BDC0-DESKTOP"
| where process_commandline has_any ("nltest", "dclist", "Get-ADDomainController", "_msdcs", "domain controllers")
| sort by timestamp asc
```

- Hunted for event log clearing activity, same approach, based on known technique (MITRE T1070.001) rather than searching blind

```kusto
ProcessEvents
| where hostname == "BDC0-DESKTOP"
| where process_commandline has_any ("wevtutil", "Clear-EventLog", "clear-eventlog")
| project timestamp, process_commandline, parent_process_name, process_name
| sort by timestamp asc
```

- When targeted keyword searches didn't immediately resolve, fell back to a broad, unfiltered pull of every process this user ran across the full incident window, to scan by eye for anything the targeted searches missed

```kusto
ProcessEvents
| where hostname == "BDC0-DESKTOP"
| where username == "jisaetang"
| where timestamp between (datetime(2024-08-28) .. datetime(2024-09-12))
| project process_commandline, parent_process_name, process_name, timestamp
```

- Reasoning: targeted keyword hunts are fast but assume you already know the attacker's exact syntax. A wide unfiltered pull is the fallback when that assumption fails, at the cost of having to manually scan more output.

## MITRE ATT&CK Mapping

| Tactic | Technique | Evidence |
|---|---|---|
| Reconnaissance | T1595 - Active Scanning | Internal SCADA IP sweep via ICSScanner.exe |
| Initial Access | T1566.002 - Spearphishing Link | Phishing campaign across 3 shared-infra domains |
| Execution | T1204.002 - User Execution: Malicious File | User opened Urgent_Cyber_Threat_Alert.zip |
| Command and Control | T1071 - Application Layer Protocol | BlackEnergy beacon to chicagogridupdates.com |
| Discovery | T1018 / T1482 - Remote System / Domain Trust Discovery | Domain controller enumeration commands, hunt in progress |
| Credential Access | T1552.001 - Credentials In Files | Password file search on compromised host |
| Lateral Movement | T1021.002 - SMB/Windows Admin Shares | PsExec loop using stolen credentials |
| Defense Evasion | T1070.001 - Clear Windows Event Logs | Log clearing command hunt in progress |
| Impact | T1485 - Data Destruction | KillDisk deployment against SCADA systems |

## What I Learned Today

- Attacker infrastructure is a more reliable pivot than sender names or subject lines. Anchoring queries to confirmed IPs/domains via PassiveDNS scoped the real campaign correctly, where content-based filters kept surfacing false positives.
- `has`, `contains`, and `=~` are not interchangeable. Match the operator to the field: `=~` for clean structured fields, `has` for reliably delimited paths, `contains` for messy or attacker-controlled strings.
- `summarize` collapses rows and drops any field not explicitly kept. `distinct` first, then `join`, is the fix when you need to match a table's field against something that got bundled into an array.
- When a targeted keyword hunt comes up empty, a wide unfiltered pull scoped tightly by host/user/time window is a legitimate fallback, not a failure, it just trades speed for coverage.

**Big Picture:** Real-world ICS intrusions rarely announce themselves through one obvious IOC. This one only came together by chaining process telemetry, DNS infrastructure, and phishing metadata, the same layered pivoting a Tier 1 SOC analyst does when triaging a live incident.
