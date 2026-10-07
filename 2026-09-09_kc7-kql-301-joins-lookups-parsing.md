# Joins, Lookups and Parsing: Tracing a Phishing Attack Across Tables in KC7 KQL 301

**Date:** September 9, 2026 **Platform:** KC7 (kc7cyber.com) **Domain:** Threat Hunting, KQL, Phishing Analysis

---

## Scenario

Phishing attack at Whiskers and Wonders, followed from email delivery through link clicks and credential theft to the attacker's access with stolen accounts. The room teaches joining `Email`, `Employees`, `ProxyEvents`, and `AuthenticationEvents`, since no single table shows the whole attack.

## Enrichment: `let` + `in` vs `lookup`

- `let` + `in` finds who clicked but returns columns from one table only, so the click time and URL are lost
- `lookup` adds employee columns directly to each click, so each row shows who, when, and what

```
ProxyEvents
| where domain in ("api-sync-updates.top", "update-cdn-service.xyz")
| lookup Employees on $left.src_ip == $right.ip_addr
| project-reorder timestamp, src_ip, name
```

- Reasoning: `lookup` fits one-to-one enrichment (IP to employee, hostname to asset). It only returns matches and gives no control over non-matches, so `join` is the next step up.

## Join Types

- `leftouter` keeps every left row and adds right-side data where it matches. Diego's outbound emails kept external recipients with empty employee fields.
- `inner` drops rows with no match, which filters out external recipients without a separate `where`
- `rightouter` keeps every right row. Useful for flipped questions like "which employees did this sender not email?" with `isempty()` on a left-side column.
- `leftanti` returns left rows with no match on the right, with no right-side columns added
- `leftsemi` returns left rows that do have a match, as a filter with full join syntax

```
Employees
| join kind=leftanti (
    AuthenticationEvents
    | where timestamp between (datetime(2025-06-01) .. datetime(2025-06-07))
) on $left.username == $right.username
```

## Who Clicked and Who Didn't

- Chained two `leftouter` joins: emails to `Employees` for each recipient's IP, then to `ProxyEvents` to see if that IP visited the phishing URL
- `mv-expand links` first, since `Email.links` is an array and needs one row per link before it can match a URL

```
Email
| where links has "update-cdn-service.xyz"
| mv-expand links to typeof(string)
| join kind=leftouter Employees on $left.recipient == $right.email_addr
| join kind=leftouter (
    ProxyEvents
    | where url has "update-cdn-service.xyz"
) on $left.ip_addr == $right.src_ip, $left.links == $right.url
| project timestamp, sender, recipient, links, clicked_at = timestamp1
```

- Non-clickers: add `| where isempty(clicked_at)`
- Reasoning: joining on the IP alone duplicated rows, because one employee can visit several URLs on the same domain. Joining on both IP and link ties each click to the specific email.

## Parsing

- `parse` pulls named columns out of a string using anchors, such as the domain from a URL or the username from an email address
- `split()` breaks a string into an array, such as sender domain from `split(sender, "@")[1]`
- `array_length()` finds the last element when path depth varies, such as the filename in a path
- `parse_url()` extracts the host from a URL

```
Email
| extend sender_domain = tostring(split(sender, "@")[1])
| summarize count() by sender_domain
| sort by count_ desc
```

## Final Challenge

- Identified every phishing recipient with name and role
- Calculated click rate by joining emails to proxy events
- Found compromised accounts through ongoing C2 communication from their workstations
- Located the most recent attacker activity and measured the time from the first phishing email to the last C2 contact

## MITRE ATT&CK Mapping

| Tactic              | Technique                          | Evidence                                          |
| ------------------- | ---------------------------------- | ------------------------------------------------- |
| Initial Access      | T1566.002 - Spearphishing Link     | Emails with links to phishing domains             |
| Execution           | T1204.001 - User Execution: Malicious Link | Employees clicked links, seen in proxy logs |
| Credential Access   | T1056 - Input Capture              | Credentials entered on the phishing site          |
| Initial Access      | T1078 - Valid Accounts             | Attacker access using stolen accounts             |
| Command and Control | T1071 - Application Layer Protocol | Ongoing C2 traffic from compromised workstations  |

---

## What I Learned Today

- **Choosing the right tool:** `let` + `in` for quick filtering with one table's columns, `lookup` for one-to-one enrichment, `join` when I need non-matches or columns from both sides.
- **Join kinds:** `leftouter` keeps all left rows, `inner` keeps only matches, `leftanti` finds what is missing, `leftsemi` filters to what exists. `leftanti` is cleaner than `leftouter` + `isempty()` when I only need to identify non-matches.
- **Join keys matter:** joining on IP alone duplicated rows. Adding the second key (the link) fixed it. Overlapping column names also come back as `timestamp1`, so I rename columns as I go.
- **Arrays before joins:** `mv-expand` has to turn a list column into rows before it can be matched against another table.
- **`parse` vs `split`:** `parse` for anchored patterns that create named columns, `split` for simple delimiters, `parse_url` for URLs.

**Big Picture:** Connecting emails to the people who clicked and the accounts that were compromised is the cross-table correlation a Tier 1 analyst does to scope a phishing incident.
