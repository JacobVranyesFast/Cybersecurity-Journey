# Wireshark HTTP vs HTTPS, Ports Lab & Networking Concepts

**Date:** June 3, 2026
**Platform:** Home Lab + Professor Messer N10-009
**Domain:** Network+ N10-009 Domain 1, 2 | Security+ SY0-701 Domain 3

---

## Lab 1: Wireshark HTTP vs HTTPS Analysis

Captured live traffic using curl from the Ubuntu VM to both the http and https versions of example.com.

### HTTP Capture (Plaintext)
Applied filter: `http`. Observed the complete request/response cycle:
- GET request visible in plaintext, including host header
- HTTP 200 OK response with readable HTML content
- Ethernet layer showed a Gigabyte Tech NIC MAC as source (onboard NIC identified by manufacturer OUI prefix)
- IP layer showed local 10.x.x.x source, 104.x.x.x destination
- TCP layer showed ephemeral source port, destination port 80

### HTTPS Capture (Encrypted)
Applied filter: `tls`. Observed the TLS 1.3 handshake:
- Client Hello (SNI=example.com): initiates connection, SNI visible in plaintext even on encrypted traffic
- Server Hello, Change Cipher Spec: server responds, switches to encrypted mode
- Application Data: encrypted website content, unreadable in Wireshark

After the handshake, all application data is completely unreadable. In TLS 1.3 the Finished message is encrypted inside the handshake itself, so there's no visible separate Finished packet like there is in older TLS 1.2.

### Key Finding: SNI Exposure
Server Name Indication (SNI) is not encrypted. The domain being visited is visible even on HTTPS traffic, so a network monitor can see which domains a device visits without decrypting the traffic content.

### SOC Relevance
- HTTP traffic is readable in full: credentials, content, requests
- HTTPS protects content but not the destination domain (SNI)
- Enterprises use SSL inspection (TLS decryption proxy) to inspect HTTPS traffic for threats
- Unencrypted HTTP on a network where HTTPS is expected is an immediate finding

---

## Lab 2: Ports Lab (Ubuntu + Kali)

### netstat -tulpn on Ubuntu
- Port 22: SSH, listening on 0.0.0.0 (all interfaces, network reachable)
- Port 53: DNS resolver, listening on 127.0.0.53 (loopback only, not network reachable)
- Port 323: chronyd NTP, loopback only

Key distinction learned:
- `0.0.0.0:22` = reachable from network (any interface)
- `127.0.0.x:53` = loopback only, invisible to network scanners

### nmap -sV localhost on Ubuntu
Only port 22 was visible. Port 53 didn't show up because it's bound to 127.0.0.53, a different loopback address than 127.0.0.1. Demonstrates the difference between the internal view (netstat) and the external view (nmap).

### Apache Web Server Install: Port 80
```bash
sudo apt install apache2 -y
sudo systemctl start apache2
```
Port 80 immediately appeared in the nmap scan from Kali. Apache default page confirmed accessible from the Windows host browser at http://10.0.2.50.

### nmap Full Lab Network Scan from Kali
```bash
nmap -sV 10.0.2.0/24
```
- 10.0.2.1: Gateway, ports 135 (RPC), 445 (SMB) open
- 10.0.2.2: VirtualBox service, all ports closed
- 10.0.2.5: Kali, port 22 (SSH)
- 10.0.2.50: Ubuntu, ports 22 (SSH), 80 (HTTP/Apache)

Port 445 (SMB) on the gateway is worth noting since it's historically exploited by WannaCry ransomware. Confirmed here as a VirtualBox internal service, not a real misconfiguration.

### netstat Flags Comparison
- `-tulpn` = listening ports only (what doors are open)
- `-tupn` = established connections (who is connected now)

Both get used together during incident response: tulpn finds backdoors listening for attacker connections, tupn finds active exfiltration or C2 beaconing in progress.

### VM Timezone Fix
All VMs set to Central Time (America/Chicago):
```bash
sudo timedatectl set-timezone America/Chicago
sudo timedatectl set-ntp true
```
Consistent timestamps across all VMs are critical for log correlation during an incident investigation.

---

## Professor Messer N10-009: Domain 2 Videos

### Ethernet Standards
- Ethernet uses CSMA/CD (Carrier Sense Multiple Access with Collision Detection) on half-duplex connections
- Modern switched networks are full-duplex, eliminating collisions entirely
- BASE = baseband transmission, T = twisted pair copper cable, the number before BASE is speed in Mbps/Gbps

| Standard | Speed | Cable |
|----------|-------|-------|
| 10BASE-T | 10 Mbps | Cat3 |
| 100BASE-TX | 100 Mbps (Fast Ethernet) | Cat5 |
| 1000BASE-T | 1 Gbps (Gigabit) | Cat5e/Cat6 |
| 10GBASE-T | 10 Gbps | Cat6a/Cat7 |

### Copper Cabling

| Category | Speed | Notes |
|----------|-------|-------|
| Cat5 | 100 Mbps | 100m max, largely obsolete |
| Cat5e | 1 Gbps | 100m max, most common legacy install |
| Cat6 | 1 Gbps (10G up to 55m) | better crosstalk protection |
| Cat6a | 10 Gbps | 100m max, augmented Cat6 |
| Cat7 | 10 Gbps | shielded, proprietary connectors |
| Cat8 | 25/40 Gbps | short runs, data center use |

All use RJ-45 connectors except Cat7/8 variants. Crosstalk is interference between wire pairs; higher category means better crosstalk protection.

Straight-through vs crossover cables:
- Straight-through: connects different device types (PC to switch, switch to router)
- Crossover: connects same device types (PC to PC, switch to switch)

Modern switches auto-detect via MDIX.

### Optical Fiber

| | Single-mode (SMF) | Multi-mode (MMF) |
|---|---|---|
| Core | 8-10 microns (very narrow) | 50-62.5 microns (wider) |
| Light | single laser beam, one path | multiple light paths (modes) |
| Distance | up to 100km | up to 2km |
| Use | long distance, WAN, between buildings | short distance, within buildings, data centers |
| Color | yellow jacket typically | orange or aqua jacket typically |

Fiber is immune to electromagnetic interference (EMI) and can't be tapped without physical interruption, which makes it more secure than copper for sensitive runs.

### Wireless Networking

| Standard | Band | Max Speed | Year | Notes |
|----------|------|-----------|------|-------|
| 802.11a | 5 GHz | 54 Mbps | 1999 | |
| 802.11b | 2.4 GHz | 11 Mbps | 1999 | |
| 802.11g | 2.4 GHz | 54 Mbps | 2003 | |
| 802.11n | 2.4/5 GHz | 600 Mbps | 2009 | MIMO |
| 802.11ac | 5 GHz | 3.5 Gbps | 2013 | MU-MIMO, Wi-Fi 5 |
| 802.11ax | 2.4/5/6 GHz | 9.6 Gbps | 2019 | Wi-Fi 6 |

2.4 GHz vs 5 GHz:
- 2.4 GHz: longer range, slower, more interference, only 3 non-overlapping channels (1, 6, 11)
- 5 GHz: shorter range, faster, less interference, 23 non-overlapping channels

Key wireless security standards:
- WEP: broken, never use
- WPA: improved but weak
- WPA2: current standard, AES encryption
- WPA3: newest, stronger, mandatory for Wi-Fi 6

WPS vulnerability: allows PIN-based brute force attack, should be disabled on all access points.

---

## Concepts Studied

### DNS Deep Dive
Full DNS resolution chain: local cache → router cache → recursive resolver → root servers → TLD servers → authoritative server. Each layer caches for the TTL duration.

DNS server types:
- Recursive resolver: does the lookup work (8.8.8.8, 1.1.1.1)
- Root servers: top of hierarchy, knows TLD locations only
- TLD server: knows authoritative servers per domain
- Authoritative: owns the actual DNS records, final answer
- Forwarding server: passes queries to another server
- Caching server: stores answers for the TTL duration

CNAME rule: aliases a hostname to another hostname, never to an IP. Cannot coexist with other record types at the same name. Cannot be used at the root domain.

MX records: specify mail servers with priority numbers. Lower number means higher preference. Multiple MX records provide failover redundancy; equal priority means load balancing.

### Traffic Types
- Unicast: one to one, specific destination
- Broadcast: one to all local devices, never crosses a router
- Multicast: one to a subscribed group (224.0.0.0/4)
- Anycast: one to the nearest server sharing the same IP, used by DNS root servers, Cloudflare, Google DNS

### Linux Log Analysis Tools
- `find`: locate files by name, size, permissions, date
  ```bash
  find / -name "*.pcap" 2>/dev/null
  find / -perm 777 2>/dev/null
  ```
- `grep`: search text content inside files
  ```bash
  grep "Failed password" /var/log/auth.log
  ```
- `cut`: extract specific columns by delimiter
  ```bash
  cut -d: -f1 /etc/passwd
  ```
- `awk`: process text by columns with logic and conditions
  ```bash
  grep "Failed" auth.log | awk '{print $11}'
  ps aux | awk '$3 > 1.0 {print $2, $11}'
  ```

Real SOC query: find and rank brute force IPs
```bash
grep "Failed password" /var/log/auth.log \
| awk '{print $11}' | sort | uniq -c | sort -rn
```

### Ports Memorized

| Port | Service | Protocol |
|------|---------|----------|
| 22 | SSH | TCP |
| 25 | SMTP (server to server) | TCP |
| 53 | DNS | UDP primary, TCP for large/zone transfer |
| 80 | HTTP | TCP |
| 110 | POP3 | TCP |
| 143 | IMAP | TCP |
| 443 | HTTPS | TCP |
| 445 | SMB | TCP |
| 587 | SMTP submission (client to server) | TCP |
| 3306 | MySQL | TCP |
| 3389 | RDP | TCP |
| 161 | SNMP | UDP |
| 123 | NTP | UDP |
| 67/68 | DHCP | UDP |
| 20/21 | FTP data/control | TCP |

---

## What I Learned Today
- SNI exposes the destination domain even on HTTPS: full traffic encryption doesn't mean full privacy
- netstat and nmap answer different questions: internal view vs external attacker view
- Loopback bound services (127.x.x.x) are invisible to network scanners, which matters for both security hardening and SOC investigation
- Fiber optic cable can't be tapped without physical interruption, making it more secure than copper for sensitive runs
- 802.11ax (Wi-Fi 6) supports 2.4, 5, and 6 GHz bands
- WPS should always be disabled: the PIN brute force vulnerability affects all WPA2 implementations
- cut extracts columns, awk adds logic and conditions, and combined with grep they enable powerful log analysis without any external tools

**Big Picture:** Reading traffic at the packet level and knowing which ports and services should be listening where is the foundation of triage. Half of a Tier 1 investigation is just being able to tell what's normal from what isn't.
