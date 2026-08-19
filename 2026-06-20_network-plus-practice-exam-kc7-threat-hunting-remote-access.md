# Network+ Practice Exam, KC7 Threat Hunting & Remote Access Lab

**Date:** June 20, 2026
**Platform:** Udemy (Jason Dion) + KC7 + Raspberry Pi / Apollo / Moonlight home lab
**Domain:** Network+ N10-009 Exam Prep | SOC Threat Hunting (KQL) | Home Lab Infrastructure

---

## What I Did
- Took a full Jason Dion N10-009 practice exam
- Continued KC7's Valordira investigation: KQL queries across ProcessEvents, FileEvents, EmailEvents, and inbound/outbound network event tables
- Got remote access to Jimba-PC fully working end to end: Tailscale + Raspberry Pi (UpSnap) + Apollo/Moonlight

## Network+ Practice Exam
- Score: 66%
- Dion's exams run intentionally harder than the real N10-009, not a direct readiness signal on its own
- Real benchmark: consistent 75%+ scores plus closing specific gaps
- Weak areas to drill before booking:
  - Layer 2 loop prevention (why STP exists, not just BPDU timers)
  - NEXT crosstalk vs alien crosstalk vs EMI
  - Switch port error states (suspended / error-disabled / administratively down)
  - VLAN misassignment as a cause of connectivity failure despite valid IP config
  - 0.0.0.0/0 notation and gateway-of-last-resort logic
  - DHCP-side fixes for APIPA (pool exhaustion, scope expansion, lease time)

## KC7: Valordira Module
- Confirmed a SHA256 file hash for a suspicious document on a specific user's machine
  - Verified the username-to-hostname mapping first via authentication/login events, rather than assuming
  - Scoped the FileEvents query to the confirmed host
- File metadata (hashes) lives in FileEvents, not ProcessEvents: a document's filename won't show up in a process's own name field, only potentially in a child process's command line if a macro spawned one
- Practiced identity-to-network correlation: pulled a user's IP from Employees, then scoped OutboundNetworkEvents by src_ip
- Same pattern confirms a phishing click: match a URL from EmailEvents against outbound traffic to that URL

## Home Lab: Remote Access Fully Functional

Stack: Tailscale (WireGuard mesh transport) + Raspberry Pi (`pi-upsnap`) running UpSnap as WoL relay + Apollo/Moonlight for streaming, RDP as fallback.

Two issues fixed today:
- **Pi lost Tailscale auth + DHCP lease overnight.** `tailscaled` was running but stuck on "Needs login" (browser auth from initial setup never completed), and the Pi's LAN IP had drifted after a power event. Found the live IP via nmap, completed Tailscale auth at the console, created the UpSnap superuser via a one-off Docker container against the existing data volume (the documented CLI command for this didn't work, even with an entrypoint override).
- **Apollo losing pairing on every host reboot.** Known issue with Apollo/Sunshine running as a Windows Service: pairing data doesn't reliably persist across reboots in that mode. Fix: switch to the portable build, autostarted via Task Scheduler under the user account instead of the installed service.

End-to-end test passed: powered Jimba-PC fully off, triggered wake from the UpSnap web UI, PC powered back on.

---

## What I Learned Today
- The Pi's IP drift is the same class of problem as a stale lease in production: the fix is a DHCP reservation, not just finding the new address. Setting that up next for the whole Pi fleet and Jimba-PC
- The Apollo issue was a good reminder that "broke after a reboot" is its own diagnostic category: service execution context matters as much as config
- Verifying the username-to-hostname mapping before scoping a query, instead of assuming it, is a small habit that prevents investigating the wrong machine entirely

**Big Picture:** Confirming identity before scoping a query is a habit that transfers directly to real investigations: assuming a username maps to the obvious hostname is exactly the kind of shortcut that sends an investigation in the wrong direction.
