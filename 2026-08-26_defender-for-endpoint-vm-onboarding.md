# Onboarding a New Lab VM to Microsoft Defender for Endpoint

**Date:** August 26-27, 2026
**Platform:** Microsoft Defender XDR / Microsoft Entra ID
**Domain:** Endpoint Detection and Response (EDR), Identity, Attack Surface Reduction

---

## What I did

- Built a new Windows 11 Enterprise VM in Oracle VirtualBox using Microsoft's official evaluation ISO, isolated on its own NAT network separate from the rest of my home lab
- Joined the VM to Microsoft Entra ID (JacobsCyberLab tenant) directly through Settings, confirmed the join with `dsregcmd /status` rather than trusting the UI text alone
- Onboarded the VM to Microsoft Defender for Endpoint using the local script deployment method, aware that Intune, GPO, and Configuration Manager exist for onboarding at scale beyond a handful of machines
- Verified onboarding actually succeeded by confirming the device showed up under Assets > Devices in the Defender portal, rather than assuming a clean script run meant it worked
- Created two separate groups for this environment, a Microsoft Entra ID group and a Defender device group, kept them intentionally distinct rather than collapsing them into one
- Built an Attack Surface Reduction (ASR) policy scoped to the device group, set every rule to Audit rather than Block, since I'm still actively building this environment out and don't want to break something I haven't observed yet. Plan is to review the audit telemetry and move specific rules to Block once I know what's actually normal here.

## Detection testing: two different approaches, one clear result

Tried Microsoft's official EDR detection test script first (a PowerShell one-liner that simulates a download-then-execute chain from `127.0.0.1`). Getting this running end to end surfaced several real troubleshooting steps: nothing was listening on port 80 by default, so the download failed silently every time; different Microsoft doc pages reference two different folder names (`test-MDATP-test` vs `test-WDATP-test`) left over from the product's rename history; and typing the one-liner directly into a live PowerShell session (instead of cmd.exe, which it's written for) breaks the `$ErrorActionPreference=` assignment due to how PowerShell parses it.

Fixed all three (enabled IIS as a local listener, placed a legitimate signed Windows binary at the expected download path, corrected the folder name, ran the core logic directly in PowerShell) and confirmed the full technical chain worked, the downloaded program visibly launched. But this did not generate a Defender alert. Most likely reason: the official test's detection is tied to Microsoft's actual placeholder file and its known reputation/hash, not to the general behavior of "PowerShell downloads and runs something." Substituting a trusted, signed system binary completed the mechanics but gave Defender nothing suspicious to actually flag.

Switched to the EICAR test string method instead, a standardized string every antivirus engine is built to recognize, specifically so testing doesn't require real malware. This is a signature-detection test, not a behavioral one, so it doesn't depend on file reputation or process ancestry the way the first test did.

**Result:** Incident ID 1, `'EICAR_Test_File' malware was prevented`, Informational severity, first activity Aug 27, 2026 6:08 AM.

Confirmed the detection at the engine level with `Get-MpThreatDetection`, not just the portal UI:

| Field | Value | Meaning |
|---|---|---|
| DetectionSourceTypeID | 3 | Caught by real-time protection specifically |
| CurrentThreatExecutionStatusID | 1 | Blocked |
| CleaningActionID | 2 | Quarantine |
| ThreatStatusID | 3 | Quarantined |
| InitialDetectionTime → RemediationTime | 6:08:16 AM → 6:08:39 AM | 23 seconds, detection to remediation |

Also separately confirmed `AMRunningMode: Normal` via `Get-MpComputerStatus`, ruling out a passive/degraded AV engine as a factor. Checking `Test-Path` on the EICAR file at two different points returned `True` then later `False`, initially confusing until the quarantine action explained it: Defender doesn't just flag a detected file in place, it removes it, so the file disappearing was the remediation itself, not a separate issue to chase down.

Also caught a portal UI gotcha along the way: the default Incidents and Alerts view filters to `High, Medium, Low` severity only, silently hiding Informational alerts like this one. Confirmed the alert was actually there via the device's Overview tab instead of trusting the filtered list view.

## MITRE ATT&CK mapping

| Technique | ID | Notes |
|---|---|---|
| Ingress Tool Transfer | T1105 | Attempted via the official download-then-execute test script; completed mechanically but did not trigger detection, likely due to using a trusted signed binary as the payload |
| Command and Scripting Interpreter: PowerShell | T1059.001 | PowerShell used as the delivery mechanism for the download-then-execute test |

*Note: the EICAR test itself isn't mapped to an ATT&CK technique, since EICAR isn't an emulated adversary behavior, it's a standardized artifact for validating that an AV engine's signature detection is actually active.*

## What I learned today

- Entra ID join and Defender for Endpoint onboarding are two separate tracks. Being Entra joined doesn't automatically enroll a device in Defender, they have to be set up independently, though they reinforce each other once both are in place
- Audit mode exists for a reason. Rolling out ASR rules in Block on a system I don't fully understand yet is how you break your own environment. Audit first, review what actually fires, then tighten
- Not every "test" script actually tests what it looks like it tests. The official download-execute script completing successfully told me the mechanics worked, not that Defender would flag it, those turned out to be separate questions, and only the file's reputation determined the second one
- Signature-based detection (EICAR) and behavioral detection (the download-execute test) are different tools for different questions. EICAR proves the AV engine is alive and scanning. It doesn't prove EDR is catching real adversary behavior patterns, that needs a different kind of test
- Portal UI defaults can hide real results. A default severity filter meant a genuine alert sat invisible in the main incidents list until I checked the device page directly
- A file disappearing after a detection isn't a mystery to chase, it's the remediation action itself. Learned to read `Test-Path` going from `True` to `False` as confirmation, not confusion
- The portal tells you *that* something happened. `Get-MpThreatDetection` tells you *exactly* what the engine did and when, down to the second. Worth checking both, not just the one that's easier to look at

**Big Picture:** Getting a script to run without errors and getting a detection to actually fire are two different milestones, and conflating them is how you end up with a false sense of coverage. Today was as much about learning what each test actually validates as it was about getting an alert to show up.
