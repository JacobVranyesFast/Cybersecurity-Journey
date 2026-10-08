# KC7 Azure Crest: Lookalike-Domain Phishing, SSH Staging & Ransomware Investigation

**Date:** October 8, 2026 **Platform:** KC7 (Azure Crest Hospital, Azure Data Explorer) **Domain:** Threat Hunting

---

## What I Did

- Investigated a multi-stage intrusion at Azure Crest Hospital in ADX (AzureCrest database) across Email, FileCreationEvents, ProcessEvents, InboundNetworkEvents, SecurityAlerts and PassiveDns
- Traced the phishing delivery, file download, attacker staging folder, SSH activity and database server compromise
- Identified two lookalike domains impersonating Azure Crest partners
- Extracted attacker IPs from command lines with `parse` and pivoted them into inbound web traffic
- Decoded obfuscated strings (Base64, ROT13, reversed text) and cleaned defanged command-line IOCs

## Investigation Steps

1. **Delivery:** Email table showed links to two macro-enabled files, `New_Healthcare_Protocols.docm` and `Pediatric_Care_Update.docm`. Example: `medstaffinfo@hospitalcomm.org` emailing Jerry Jones (Resident Doctors).
2. **Lookalike domain:** The link domain `takeyatimecarepartners.com` mimics partner Emergency Care Partners (`emergencycarepartners.com`) from the company's partner list.
3. **Download confirmed:** FileCreationEvents showed `New_Healthcare_Protocols.docm` created on ZQHM-LAPTOP at 2024-03-14T10:38:36Z by `chrome.exe`. Delivery alone doesn't prove a click, a file creation event does.
4. **Staging:** On the first affected machine (P3EX-DESKTOP), I pivoted on a time window around the file creation and found the attacker folder `C:\ProgramData\Heartburn`.
5. **SSH activity:** ProcessEvents showed `cmd.exe` launching `C:\ProgramData\Heartburn\putty.exe -ssh` to external IPs with credentials in plaintext on the command line. `distinct hostname` gave the compromised machine count.
6. **Recon:** Parsed the SSH destination IPs and searched InboundNetworkEvents for those IPs hitting the hospital website.
7. **Database server:** Pivoted to Roy Trenneman's mailbox and host SUPER-DB-SERVER-9000. Searched ProcessEvents for `dbhunter` and archive commands, and FileCreationEvents for encryption artifacts.
8. **Second lookalike:** A defanged `curl.exe` command pulled `anydesk_automation.ps1` from `unhealthyrecordsystems[.]tech` into the Heartburn folder. The domain impersonates partner Health Records Systems (`healthrecordsystems.tech`).
9. **Decoding:** One challenge string decoded as Base64, then ROT13, then reversed, giving a path ending in `medical_inventories.db.scholopendra`.

## Query Patterns

Verbatim string for Windows paths (backslashes are literal with `@`):

```kusto
FileCreationEvents
| where hostname == "P3EX-DESKTOP"
| where path contains @"C:\ProgramData\Heartburn"
```

Domain extraction from a link column:

```kusto
Email
| where link contains "Pediatric_Care_Update.docm"
    or link contains "New_Healthcare_Protocols.docm"
| extend LinkDomain = tostring(parse_url(link).Host)
| summarize count() by LinkDomain
```

Pull IPs out of command lines with `parse`, then pivot with `let` and `in`:

```kusto
let SshIPs =
    ProcessEvents
    | where process_commandline contains @"C:\ProgramData\Heartburn\putty.exe -ssh"
    | parse process_commandline with * "putty.exe -ssh " SshIP " -" *
    | distinct SshIP;
InboundNetworkEvents
| where src_ip in (SshIPs)
```

Time-window pivot around a known event:

```kusto
let target = datetime(2024-03-01 11:58:33);
ProcessEvents
| where hostname == "P3EX-DESKTOP"
| where timestamp between ((target - 30m) .. (target + 7d))
```

## IOCs (defanged)

- `takeyatimecarepartners[.]com`
- `unhealthyrecordsystems[.]tech`
- `hxxps://unhealthyrecordsystems[.]tech/anydesk_automation.ps1`
- `medstaffinfo@hospitalcomm[.]org`
- `New_Healthcare_Protocols.docm`, `Pediatric_Care_Update.docm`
- `C:\ProgramData\Heartburn\` (`putty.exe`, `anydesk_automation.ps1`)
- File extension `.scholopendra`

## MITRE ATT&CK Mapping

- T1583.001: Acquire Infrastructure: Domains (lookalike partner domains)
- T1566.002: Phishing: Spearphishing Link
- T1204.002: User Execution: Malicious File
- T1059.003: Command and Scripting Interpreter: Windows Command Shell
- T1021.004: Remote Services: SSH
- T1105: Ingress Tool Transfer (`curl.exe` download)
- T1219: Remote Access Software (AnyDesk automation script)
- T1083: File and Directory Discovery (`dbhunter.exe`)
- T1560.001: Archive Collected Data: Archive via Utility
- T1486: Data Encrypted for Impact

---

## What I Learned Today

- A file creation event proves a download, an email record only proves delivery
- Compare lookalike domains against the company's real partner list: same TLD or ending with a small change is the pattern
- `has` matches whole terms, `contains` matches substrings; `or` needs a full condition on both sides
- Use `@"..."` for Windows paths so backslashes aren't treated as escapes
- `parse_url()` returns dynamic values, wrap in `tostring()` before grouping
- `distinct` or `summarize count() by` answers "how many unique", a plain `count` answers "how many records"
- `let` plus `in` carries a list of IOCs from one table into another
- Defanged IOCs (`[.]`, `hxxps`, bracketed slashes) are safe to store in notes, clean them only to read
- KC7 to Microsoft table mapping: ProcessEvents → DeviceProcessEvents, FileCreationEvents → DeviceFileEvents, Email → EmailEvents

**Big Picture:** Tracing one phishing link through email, file, process and network logs is the same pivot chain a Tier 1 analyst runs on a real alert.
