# Week 3 Day 3 — Wireshark, Ports Lab & Networking Concepts

**Date:** June 3, 2026
**Platform:** Home Lab + Professor Messer N10-009
**Domain:** Network+ N10-009 Domain 1, 2 | Security+ SY0-701 Domain 3

---

## Lab 1 — Wireshark HTTP vs HTTPS Analysis

Captured live traffic using curl from Ubuntu VM to
http and https versions of example.com.

### HTTP capture (plaintext)
Applied filter: http
Observed complete request/response cycle:
- GET request visible in plaintext including host header
- HTTP 200 OK response with readable HTML content
- Ethernet layer showed Gigabyte Tech NIC MAC as source
  (onboard NIC identified by manufacturer OUI prefix)
- IP layer showed local 10.x.x.x source, 104.x.x.x destination
- TCP layer showed ephemeral source port, destination port 80

### HTTPS capture (encrypted)
Applied filter: tls
Observed TLS 1.3 handshake:
Client Hello (SNI=example.com) — initiates connection,
SNI visible in plaintext
even on encrypted traffic
Server Hello, Change Cipher Spec — server responds,
switches to encrypted mode
Application Data               — encrypted website content,
unreadable in Wireshark
After handshake all application data completely unreadable.
Finished message encrypted inside handshake in TLS 1.3 —
no visible separate Finished packet unlike older TLS 1.2.

### Key finding — SNI exposure
Server Name Indication (SNI) is not encrypted — the domain
being visited is visible even on HTTPS traffic. A network
monitor can see WHICH domains a device visits without
decrypting traffic content.

### SOC Relevance
- HTTP traffic readable in full — credentials, content, requests
- HTTPS protects content but not destination domain (SNI)
- Enterprises use SSL inspection (TLS decryption proxy) to
  inspect HTTPS traffic for threats
- Unencrypted HTTP on a network where HTTPS is expected
  is an immediate finding

---

## Lab 2 — Ports Lab (Ubuntu + Kali)

### netstat -tulpn on Ubuntu
Port 22  — SSH, listening on 0.0.0.0 (all interfaces,
network reachable)
Port 53  — DNS resolver, listening on 127.0.0.53
(loopback only, not network reachable)
Port 323 — chronyd NTP, loopback only

Key distinction learned:
0.0.0.0:22    = reachable from network (any interface)
127.0.0.x:53  = loopback only, invisible to network scanners

### nmap -sV localhost on Ubuntu
Only port 22 visible — port 53 not found because it is
bound to 127.0.0.53, a different loopback address than
127.0.0.1. Demonstrates difference between internal view
(netstat) and external view (nmap).

### Apache web server install — port 80
```bash
sudo apt install apache2 -y
sudo systemctl start apache2
```
Port 80 immediately appeared in nmap scan from Kali.
Apache default page confirmed accessible from Windows
host browser at http://10.0.2.50.

### nmap full lab network scan from Kali
nmap -sV 10.0.2.0/24
10.0.2.1   — Gateway: ports 135 (RPC), 445 (SMB) open
10.0.2.2   — VirtualBox service: all ports closed
10.0.2.5   — Kali: port 22 (SSH)
10.0.2.50  — Ubuntu: ports 22 (SSH), 80 (HTTP/Apache)

Port 445 (SMB) on gateway noted — historically exploited
by WannaCry ransomware. Confirmed as VirtualBox internal
service, not a real misconfiguration.

### netstat flags comparison
-tulpn  = listening ports only (what doors are open)
-tupn   = established connections (who is connected now)
Both used together during incident response:
tulpn finds backdoors listening for attacker connections.
tupn finds active exfiltration or C2 beaconing in progress.

### VM Timezone Fix
All VMs set to Central Time (America/Chicago):
```bash
sudo timedatectl set-timezone America/Chicago
sudo timedatectl set-ntp true
```
Consistent timestamps across all VMs critical for
log correlation during incident investigation.

---

## Professor Messer N10-009 — Domain 2 Videos

### Ethernet Standards
- Ethernet uses CSMA/CD (Carrier Sense Multiple Access
  with Collision Detection) on half-duplex connections
- Modern switched networks are full-duplex,
  eliminating collisions entirely
- Standards define speed and cable requirements:
10BASE-T    — 10 Mbps, Cat3
100BASE-TX  — 100 Mbps (Fast Ethernet), Cat5
1000BASE-T  — 1 Gbps (Gigabit), Cat5e/Cat6
10GBASE-T   — 10 Gbps, Cat6a/Cat7
- BASE = baseband transmission
- T = twisted pair copper cable
- Number before BASE = speed in Mbps/Gbps

### Copper Cabling
Cable categories and key specs:
Cat5    — 100 Mbps, 100m max, largely obsolete
Cat5e   — 1 Gbps, 100m max, most common legacy install
Cat6    — 1 Gbps (10G up to 55m), better crosstalk protection
Cat6a   — 10 Gbps, 100m max, augmented Cat6
Cat7    — 10 Gbps, shielded, proprietary connectors
Cat8    — 25/40 Gbps, short runs, data center use
All use RJ-45 connectors (except Cat7/8 variants).
Crosstalk = interference between wire pairs — higher
category = better crosstalk protection.

Straight-through vs crossover cables:
Straight-through — connect different device types
PC to switch, switch to router
Crossover        — connect same device types
PC to PC, switch to switch
(modern switches auto-detect via MDIX)

### Optical Fiber
Single-mode fiber (SMF)
Core: 8-10 microns (very narrow)
Light: single laser beam, one path
Distance: up to 100km
Use: long distance, WAN, between buildings
Color: yellow jacket typically
Multi-mode fiber (MMF)
Core: 50-62.5 microns (wider)
Light: multiple light paths (modes)
Distance: up to 2km
Use: short distance, within buildings, data centers
Color: orange or aqua jacket typically
Fiber immune to electromagnetic interference (EMI)
and cannot be tapped without physical interruption —
more secure than copper for sensitive runs.

### Wireless Networking
802.11 standards:
802.11a   — 5 GHz, 54 Mbps, 1999
802.11b   — 2.4 GHz, 11 Mbps, 1999
802.11g   — 2.4 GHz, 54 Mbps, 2003
802.11n   — 2.4/5 GHz, 600 Mbps, MIMO, 2009
802.11ac  — 5 GHz, 3.5 Gbps, MU-MIMO, 2013 (Wi-Fi 5)
802.11ax  — 2.4/5/6 GHz, 9.6 Gbps, 2019 (Wi-Fi 6)

2.4 GHz vs 5 GHz:
2.4 GHz — longer range, slower, more interference
only 3 non-overlapping channels (1, 6, 11)
5 GHz   — shorter range, faster, less interference
23 non-overlapping channels

Key wireless security standards:
WEP   — broken, never use
WPA   — improved but weak
WPA2  — current standard, AES encryption
WPA3  — newest, stronger, mandatory for Wi-Fi 6

WPS vulnerability — allows PIN-based brute force attack,
should be disabled on all access points.

---

## Concepts Studied

### DNS Deep Dive
Full DNS resolution chain: local cache → router cache →
recursive resolver → root servers → TLD servers →
authoritative server. Each layer caches for TTL duration.

DNS server types:
Recursive resolver  — does the lookup work (8.8.8.8, 1.1.1.1)
Root servers        — top of hierarchy, knows TLD locations only
TLD server          — knows authoritative servers per domain
Authoritative       — owns actual DNS records, final answer
Forwarding server   — passes queries to another server
Caching server      — stores answers for TTL duration

CNAME rule: aliases a hostname to another hostname,
never to an IP. Cannot coexist with other record types
at the same name. Cannot be used at root domain.

MX records: specify mail servers with priority numbers.
Lower number = higher preference. Multiple MX records
provide failover redundancy. Equal priority = load balancing.

### Traffic Types
Unicast    — one to one, specific destination
Broadcast  — one to all local devices, never crosses router
Multicast  — one to subscribed group (224.0.0.0/4)
Anycast    — one to nearest server sharing same IP
used by DNS root servers, Cloudflare, Google DNS

### Linux Log Analysis Tools
find  — locate files by name, size, permissions, date
find / -name "*.pcap" 2>/dev/null
find / -perm 777 2>/dev/null
grep  — search text content inside files
grep "Failed password" /var/log/auth.log
cut   — extract specific columns by delimiter
cut -d: -f1 /etc/passwd
awk   — process text by columns with logic and conditions
grep "Failed" auth.log | awk '{print $11}'
ps aux | awk '$3 > 1.0 {print $2, $11}'

Real SOC query — find and rank brute force IPs:
```bash
grep "Failed password" /var/log/auth.log \
| awk '{print $11}' | sort | uniq -c | sort -rn
```

### Ports Memorized
22   — SSH (TCP)
25   — SMTP server to server (TCP)
53   — DNS (UDP primary, TCP for large/zone transfer)
80   — HTTP (TCP)
110  — POP3 (TCP)
143  — IMAP (TCP)
443  — HTTPS (TCP)
445  — SMB (TCP)
587  — SMTP submission client to server (TCP)
3306 — MySQL (TCP)
3389 — RDP (TCP)
161  — SNMP (UDP)
123  — NTP (UDP)
67/68 — DHCP (UDP)
20/21 — FTP data/control (TCP)

---

## What I Learned Today

- SNI exposes the destination domain even on HTTPS —
  full traffic encryption doesn't mean full privacy
- netstat and nmap answer different questions —
  internal view vs external attacker view
- Loopback bound services (127.x.x.x) are invisible
  to network scanners — important for both security
  hardening and SOC investigation
- Fiber optic cable cannot be tapped without physical
  interruption — more secure than copper for sensitive runs
- 802.11ax (Wi-Fi 6) supports 2.4, 5, and 6 GHz bands
- WPS should always be disabled — PIN brute force
  vulnerability affects all WPA2 implementations
- cut extracts columns, awk adds logic and conditions —
  combined with grep enables powerful log analysis
  without any external tools
  