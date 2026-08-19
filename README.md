# Jacob Vranyes-Fast's Cybersecurity Journey

## About
Aspiring Tier 1 SOC Analyst documenting my journey from zero 
to hired. This repository contains home lab write-ups, 
study notes, and security tool documentation.

## Certifications & Training
- [x] CompTIA Network+ N10-009: passed June 2026
- [x] CompTIA Security+ SY0-701: passed July 2026
- [x] [Let's Defend](https://app.letsdefend.io/user/JacobVranyesFast) SOC Analyst Path: completed, continuing with alert investigations and challenges on the platform
- [ ] Microsoft SC-200: in progress

## Home Lab
Started as a 3-VM VirtualBox setup, since rebuilt on Proxmox as the main hypervisor. Currently running:
- Kali Linux: recon, scanning, attack simulation
- Ubuntu Server running the Wazuh SIEM stack (manager, indexer, dashboard, Filebeat)
- Windows Server 2022: Active Directory, event log analysis
- FlareVM: Windows-side malware analysis tooling
- REMnux: Linux-side malware analysis tooling
- Pi-hole: network-wide DNS filtering, reachable remotely over Tailscale

Side project in progress: a 20+ node Raspberry Pi cluster, physical hardware fully wired up, currently getting K3s (lightweight Kubernetes) installed and workloads running on it. Right now only two of the Pis are doing active work (Pi-hole and a Wake-on-LAN relay for the main workstation); the rest are staged for the cluster build-out.

## Write-Ups
| Date | Topic | Link |
|------|-------|------|
| May 21, 2026 | Home Lab Setup: 3-VM VirtualBox Environment | [View](2026-05-21_home-lab-setup.md) |
| May 22, 2026 | Cryptography Fundamentals & Hashing | [View](2026-05-22_cryptography-fundamentals-hashing.md) |
| May 26, 2026 | Linux Labs, Network Troubleshooting & Subnetting | [View](2026-05-26_linux-labs-network-troubleshooting-subnetting.md) |
| May 27, 2026 | Wireshark, tcpdump, nmap & Subnetting | [View](2026-05-27_wireshark-tcpdump-nmap-subnetting.md) |
| May 28, 2026 | DNS Deep Dive & Network+ Quiz Practice | [View](2026-05-28_dns-deep-dive-network-plus-quiz.md) |
| May 29, 2026 | John the Ripper & Network+ Baseline Exam | [View](2026-05-29_john-the-ripper-network-plus-baseline-exam.md) |
| June 1, 2026 | Linux Fundamentals Lab, SCP & Process Monitoring | [View](2026-06-01_linux-fundamentals-scp-process-monitoring.md) |
| June 2, 2026 | Network+ Practice Exam: Domain Breakdown & Subnetting Review | [View](2026-06-02_network-plus-practice-exam-subnetting.md) |
| June 3, 2026 | Wireshark HTTP vs HTTPS, Ports Lab & Networking Concepts | [View](2026-06-03_wireshark-http-https-ports-lab.md) |
| June 4, 2026 | Routing Lab, Nmap Scanning & Physical Firewall Setup | [View](2026-06-04_routing-lab-nmap-physical-firewall.md) |
| June 7, 2026 | Network Tool Mastery Lab | [View](2026-06-07_network-tool-mastery-lab.md) |
| June 9, 2026 | Network+ Travel Study, KQL Training & Microsoft Sentinel Research | [View](2026-06-09_network-plus-travel-prep-kql-training-sentinel-research.md) |
| June 15, 2026 | Network+ Study Session: Tunneling Protocols, Network Architecture & SDN | [View](2026-06-15_network-plus-tunneling-architecture-sdn.md) |
| June 16, 2026 | Physical Installations, Disaster Recovery, DHCP, DNS & Time Protocols | [View](2026-06-16_network-plus-physical-installations-disaster-recovery-dns.md) |
| June 17, 2026 | Network Security Concepts & Attack Types | [View](2026-06-17_network-security-concepts-attack-types.md) |
| June 19, 2026 | Network+ Domain 5 Troubleshooting Review & Raspberry Pi Remote Access | [View](2026-06-19_network-plus-troubleshooting-raspberry-pi-remote-access.md) |
| June 20, 2026 | Network+ Practice Exam, KC7 Threat Hunting & Remote Access Lab | [View](2026-06-20_network-plus-practice-exam-kc7-threat-hunting-remote-access.md) |
| June 22, 2026 | Network+ Final Push: Ports & Protocol Review | [View](2026-06-22_network-plus-ports-protocols-review.md) |
| June 23, 2026 | Network+ Final Push: Cloud, Wireless & Final Memorization | [View](2026-06-23_network-plus-cloud-wireless-subnetting-review.md) |
| June 24, 2026 | Network+ Final Push: Light Review Day | [View](2026-06-24_network-plus-light-review-day.md) |
| June 25, 2026 | Network+ N10-009 Exam: Passed | [View](2026-06-25_network-plus-n10-009-exam-passed.md) |
| June 29, 2026 | Security+ Study Hard Start & MSSP Interviews | [View](2026-06-29_security-plus-study-start-mssp-interviews.md) |
| June 30, 2026 | Home Lab Build: Proxmox, Wazuh SIEM & Pi-hole DNS | [View](2026-06-30_home-lab-proxmox-wazuh-pihole.md) |
| June 30, 2026 | Security+ Study Log: Cryptography, Threat Actors & Vulnerability Types | [View](2026-06-30_security-plus-cryptography-threat-actors-vulnerabilities.md) |
| July 6, 2026 | Security+ Domains 3-4 Study & Remote Pi-hole Access via Tailscale | [View](2026-07-06_security-plus-domains-3-4-pihole-tailscale.md) |
| July 16, 2026 | CompTIA Security+ SY0-701: Certification Passed | [View](2026-07-16_security-plus-sy0-701-certification-passed.md) |
| Week of July 20, 2026 | LetsDefend: Phishing & Web Attack Alert Queue | [View](2026-07-20_letsdefend-phishing-web-attacks-week.md) |
| Week of July 27, 2026 | LetsDefend: Malware Alert Investigations | [View](2026-07-27_letsdefend-malware-week.md) |
| Week of August 3, 2026 | LetsDefend: SOC Analyst Learning Path, Training Focus Week | [View](2026-08-03_letsdefend-soc-analyst-path-training.md) |
| Week of August 10, 2026 | LetsDefend: Ransomware & Emotet Investigations | [View](2026-08-10_letsdefend-ransomware-emotet-week.md) |
| Week of August 17, 2026 | LetsDefend: Threat Intel & Malware Alerts (in progress) | [View](2026-08-17_letsdefend-threat-intel-malware-week.md) |
| August 19, 2026 | KC7 KQL 201: Aggregation, Time Filtering & Multi-Field Summarization | [View](2026-08-19_kc7-kql-201-aggregation-time-filtering.md) |

See [TEMPLATE.md](TEMPLATE.md) for the format every write-up above follows.

## Tools & Skills
- Proxmox, VirtualBox, Wazuh SIEM, Wireshark, nmap
- Linux (Ubuntu, Kali, REMnux), Windows Server, FlareVM
- Tailscale, Pi-hole, Raspberry Pi / K3s
- KQL (Microsoft Sentinel / Defender XDR query language)
- Platforms: [Let's Defend](https://app.letsdefend.io/user/JacobVranyesFast), [KC7](https://kc7cyber.com/profile/422f062b), [TryHackMe](https://tryhackme.com/p/JacobVranyesFast)

## Current Focus
No longer following a fixed week-by-week plan, this is a continuous learning phase at this point:
- **Done:** CompTIA Network+ (N10-009), CompTIA Security+ (SY0-701), Let's Defend SOC Analyst Path
- **In progress:** Microsoft SC-200, ongoing Let's Defend alert investigations, KC7 KQL / threat hunting practice, Raspberry Pi cluster build-out
- **Next:** additional Microsoft security certifications

## Contact
- GitHub: (https://github.com/JacobVranyesFast)
- LinkedIn: (https://www.linkedin.com/in/jacob-vranyes-fast-3040073b7/)
