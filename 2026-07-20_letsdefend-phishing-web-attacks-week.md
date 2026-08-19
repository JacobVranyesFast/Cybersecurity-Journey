# LetsDefend: Phishing & Web Attack Alert Queue, Week of July 20

**Date:** Week of July 20, 2026
**Platform:** [LetsDefend](https://app.letsdefend.io/user/JacobVranyesFast)
**Domain:** SOC Alert Investigation (Phishing, Web Attacks)

---

## What I Did
Worked the closed-alert queue in LetsDefend's simulated SOC for the week, 12 alerts total, split almost evenly between email-based phishing (Exchange) and web application attacks. Every alert below was correctly triaged and closed as a True Positive.

| Date | Severity | Rule | Type |
|------|----------|------|------|
| Jul 20 | High | SOC146 - Phishing Mail Detected: Excel 4.0 Macros | Exchange |
| Jul 20 | Medium | SOC140 - Phishing Mail Detected: Suspicious Task Scheduler | Exchange |
| Jul 20 | Medium | SOC120 - Phishing Mail Detected: Internal to Internal | Exchange |
| Jul 21 | High | SOC114 - Malicious Attachment Detected: Phishing Alert | Exchange |
| Jul 21 | High | SOC165 - Possible SQL Injection Payload Detected | Web Attack |
| Jul 23 | High | SOC170 - Passwd Found in Requested URL: Possible LFI Attack | Web Attack |
| Jul 23 | High | SOC141 - Phishing URL Detected | Proxy |
| Jul 23 | Medium | SOC169 - Possible IDOR Attack Detected | Web Attack |
| Jul 23 | Medium | SOC166 - Javascript Code Detected in Requested URL | Web Attack |
| Jul 23 | High | SOC168 - Whoami Command Detected in Request Body | Web Attack |
| Jul 23 | High | SOC167 - LS Command Detected in Requested URL | Web Attack |
| Jul 24 | Medium | SOC282 - Phishing Alert: Deceptive Mail Detected | Exchange |

## SOC169: Possible IDOR Attack, Worked Investigation

This one got a full writeup at the time since it needed actual log analysis instead of a quick triage.

An external source contacted internal web server WebServer1006, traffic direction inbound (internet to company network). Five requests were sent in quick succession to the user info endpoint, each with a different user ID value. All five came back with a success response, and the response sizes varied between requests, which suggested distinct user data was being returned each time rather than the same error page. The user agent string was outdated and inconsistent with a real browser, pointing to automated tooling. The request was allowed through by the device.

Working it on the Log Management page: filtered by source IP address, pulled every request from that source, and confirmed the pattern. Since the request sizes differed per user ID and every status code came back 200, the attacker was successfully pulling different users' data by just changing an ID in the URL, no authorization check on that endpoint.

**Verdict:** True positive, confirmed unauthorized data disclosure via IDOR.
**Action taken:** Blocked the source IP, escalated to Tier 2/IR, flagged the endpoint for the missing authorization check.

## SOC104: Malware Detected (Jul 23 instance)

Separate from the deeper SOC104 investigation later in the month (see the following week's write-up), this earlier instance in the queue was closed correctly as part of the regular alert flow.

---

## MITRE ATT&CK Mapping
- T1190: Exploit Public-Facing Application (SOC169 IDOR)
- T1566: Phishing (SOC146, SOC140, SOC120, SOC114, SOC282)
- T1059: Command and Scripting Interpreter (SOC167 LS command, SOC168 whoami command, SOC166 JS in requested URL)

---

## What I Learned Today
- IDOR is easy to miss if you're only looking at status codes. The tell here was response size varying per request even though every status came back 200, that's what confirmed different user data was actually being returned each time
- A stack of near-identical Exchange phishing alerts in one week (4 of the 12) is realistic SOC volume, most of the actual work is fast pattern recognition, not deep investigation, on the majority of them
- Command injection style web attacks (whoami, LS command detected in requested URL) show up as a distinct cluster from credential/LFI-style attacks, worth telling apart quickly since the response playbook differs

**Big Picture:** This week was mostly volume: recognizing phishing and web attack patterns fast enough to keep the queue moving, while still slowing down on the one alert (IDOR) that actually needed log correlation to confirm impact.
