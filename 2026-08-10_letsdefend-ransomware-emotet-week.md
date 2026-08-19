# LetsDefend: Ransomware & Emotet Investigations, Week of August 10

**Date:** Week of August 10, 2026
**Platform:** [LetsDefend](https://app.letsdefend.io/user/JacobVranyesFast)
**Domain:** SOC Alert Investigation (Malware, Ransomware)

---

## What I Did
The most serious week of alerts in this stretch, capped off by a Critical-severity ransomware case.

| Date | Severity | Rule | Type |
|------|----------|------|------|
| Aug 10 | Medium | SOC104 - Malware Detected | Malware |
| Aug 10 | Medium | SOC109 - Emotet Malware Detected | Malware |
| Aug 11 | Critical | SOC145 - Ransomware Detected | Malware |

## SOC145: Ransomware Detected, Full Investigation

**Role:** Security Analyst | **Difficulty:** Medium | **Result:** True Positive
**Playbook score:** 10 (67% success rate)

Playbook answers:
- Check If Someone Requested the C2: **Accessed** (answered wrong, correct answer was Not Accessed)
- Analyze Malware: Malicious
- Check if the malware is quarantined/cleaned: Not Quarantined

Host MarkPRD (`172.16.17.88`) executed `ab.exe` (MD5 `0b486fe0503524cfe4726a4022fa6a68`), confirmed as the last recorded event on the machine. Device action logged was Allowed, execution wasn't blocked.

Pivoting on the `ab.exe` hash surfaced an earlier proxy log (predating this alert) showing the same delivery pattern: `BAL_GB9684140238GE.doc` (MD5 `ac596d282e2f9b1501d66fce5a451f00`) spawned `powershell.exe`, which pulled the payload from `hxxp://thuening[.]de/cgi-bin/uo9wm/` (`ntweb.rzone[.]de`, `81.169.145[.]105:80`), proxy action Permitted. The date gap on that earlier log means it's threat intel on the hash's known delivery method rather than confirmed activity on this specific host, but it corroborates that `ab.exe` is malicious.

**Where I got it wrong:** I answered "Accessed" on whether the C2 was requested. The correct answer was Not Accessed, there's no C2 address in this scenario at all. The Editor Note spells it out: it's a True Positive because `ab.exe` is ransomware that encrypted files directly on the machine, not because it phoned home. Dynamic analysis would have shown the encryption behavior directly instead of me assuming a C2 callback had to be part of the picture. Ransomware doesn't always need a live C2 connection to do damage, encryption can run entirely locally, and I conflated "malicious behavior confirmed" with "C2 confirmed" when they're not the same finding.

**Verdict:** True Positive.
**Actions taken:** Host contained. Escalated immediately given confirmed execution plus ransomware indicators. Recommended blocklisting both hashes and the domain/IP, and checking the `172.16.17.0/24` subnet for lateral spread.

---

## MITRE ATT&CK Mapping
- T1204.002: User Execution: Malicious File (`ab.exe` execution)
- T1059.001: PowerShell (dropper chain from the `.doc`)
- T1105: Ingress Tool Transfer
- T1486: Data Encrypted for Impact (ransomware, per SOC145 classification)

---

## What I Learned Today
- Confirming "malicious" and confirming "C2 contacted" are two separate findings, not one. I answered a question about C2 activity based on the malware being confirmed malicious, when the actual evidence for C2 activity just wasn't there
- Ransomware can be a True Positive purely on execution and encryption behavior with zero network C2 involved, don't assume every malware case needs an outbound connection to be dangerous
- Hash pivoting into an older, unrelated-looking proxy log is what actually built the delivery chain here (`.doc` to PowerShell to `ab.exe`), even though that older log technically predates the alert and isn't proof of activity on this host

**Big Picture:** Getting the C2 question wrong on a Critical ransomware case was a useful mistake to make in a sandbox instead of live: it forced me to actually separate "is this malicious" from "how is it communicating," which are two different questions a real investigation has to answer independently.
