# Daily Log — Network+ Domain 5 Review + Home Lab: Remote Access

**Date:** June 19, 2026

## Network+ N10-009 — Domain 5: Network Troubleshooting

### Material Covered

Professor Messer N10-009, Domain 5:

- Network Troubleshooting Methodology
- 5.2 Physical Issues — Cable Issues, Interface Issues, Hardware Issues
- 5.3 Troubleshooting Network Services — Switching Issues, Routing and IP Issues
- 5.4 Performance Issues — Performance Issues, Wireless Issues

### Quiz Results

Self-quizzed against the video content immediately after watching, one question at a time, mixed format.

| Round | Topics | Score |
|---|---|---|
| 1 | Troubleshooting methodology, cabling/fiber, interface error states, PoE/transceiver mismatch | 6.5/8 (81%) |
| 2 | STP, VLAN configuration, routing tables/default routes, DHCP troubleshooting, performance metrics, wireless channel management | 5.5/8 (69%) |

### Concepts Solid

- CompTIA troubleshooting methodology step order
- Auto-MDIX limitations
- Bandwidth vs. throughput, jitter vs. latency
- STP BPDU timing (3 missed hello intervals at the 2-second default)
- APIPA identification
- Non-overlapping 2.4 GHz channel selection

### Gaps Identified

Layer 2 loop prevention logic (STP), crosstalk/EMI distinctions, switch port error states, VLAN misassignment as a connectivity cause, gateway-of-last-resort notation, and DHCP-side fixes for APIPA. Anki cards going in for all six.

## Home Lab: Raspberry Pi as a Remote Access Relay

Spent the rest of the day building out remote access to my main workstation (Jimba-PC) so I can reach it from my MacBook and laptop without leaving it powered on 24/7. The Pi now runs as an always-on relay in the lab — separate from the Pi-hole/syslog/honeypot units, this one's dedicated to Wake-on-LAN duty.

The setup layers a VPN mesh (Tailscale) for secure transport, the Pi for waking the PC on demand, and a streaming layer (Apollo/Moonlight) for the actual remote session, with RDP as fallback. Got the PC-side NIC and BIOS power settings configured correctly and the Pi deployed and broadcasting on the LAN. Remote streaming connectivity is still being diagnosed — confirmed the Tailscale tunnel itself is healthy, now narrowing down whether it's a firewall rule or addressing issue.

Good practical parallel to SOC work: relay/jump-host architecture, VPN mesh transport, and layer-by-layer connectivity triage (ping → port → firewall → application) are all things I'll be doing in a real environment.
