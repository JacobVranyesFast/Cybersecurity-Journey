# Network Tool Mastery Lab

**Date:** June 7, 2026
**Platform:** Home Lab (Ubuntu, Kali, Windows host)
**Domain:** Network+ N10-009 | SOC Tool Familiarization

---

## Objective
Hands-on practice with six core network analysis tools used daily in SOC environments. All commands run across CyberLab VMs (Ubuntu 10.0.2.50, Kali 10.0.2.5) and the Windows host.

## Environment
- Kali Linux 10.0.2.5
- Ubuntu-Lab 10.0.2.50
- Windows 11 Host
- VirtualBox NAT Network: 10.0.2.0/24

---

## Tools Covered

### 1. ping -c 4 8.8.8.8
Tests Layer 3 connectivity via ICMP echo requests.
- Confirmed external connectivity from the Ubuntu VM
- TTL=64 confirms a Linux-based response from Google DNS
- Zero packet loss, avg ~15ms response time
- **SOC use:** first step in any connectivity triage, host discovery, OS fingerprinting via TTL values

### 2. traceroute / tracert
Maps every router hop to the destination using incremental TTL values. Each router that drops a TTL=0 packet returns ICMP Time Exceeded, revealing its IP.

**From Kali VM:**
- Hop 1: 10.0.2.1 (VirtualBox NAT gateway)
- Hops 2-30: timed out (routers blocking ICMP, expected in a NAT environment)

**From Windows host:**
- Full path visible: home router, Comcast Roseville MN, Comcast backbone, Chicago (350 E Cermak), Google network, 8.8.8.8 in 12 hops, ~22ms
- **SOC use:** detect unexpected routing paths, BGP hijacking, traffic redirection

### 3. nmap -sn 10.0.2.0/24
Host discovery scan across the entire subnet. No port scanning, alive/dead detection only.

| Host | Status | Notes |
|------|--------|-------|
| 10.0.2.1 | Up | VirtualBox NAT gateway |
| 10.0.2.2 | Up | VirtualBox internal DNS/DHCP |
| 10.0.2.5 | Up | Kali Linux, port 22 open |
| 10.0.2.50 | Up | Ubuntu, ports 22, 80 open |

**Finding:** Port 80 open on Ubuntu from a previous UFW lab, not intentional, should be closed.
- **SOC use:** compare scan results against asset inventory, any unlisted host is an immediate investigation

### 4. netstat -an
Displays all active connections and listening ports on the local machine.
- Run on Kali: showed only port 22 listening
- Run on Ubuntu: showed ports 22, 80 listening plus an active ESTABLISHED SSH session from Kali
- Key finding: ESTABLISHED connection visible, 10.0.2.50:22 ← 10.0.2.5:43594
- **SOC use:** detect unauthorized outbound connections, C2 beaconing, unexpected listeners

### 5. ss -tulpn / ss -tupn
Modern replacement for netstat. Faster, shows process names with sudo.

**ss -tulpn (listening only):**
- Ports 22, 80, 53 (systemd-resolved, localhost only), 323 (chrony time sync)
- Note: ESTABLISHED connections filtered out by the -l flag

**ss -tupn (active connections):**
- Confirmed active SSH session: 10.0.2.50:22 ESTAB 10.0.2.5:43594
- **SOC use:** -tulpn for listening port audit, -tupn for active connection investigation

**C2 detection workflow:**
1. `ss -tupn`: find suspicious ESTABLISHED connection
2. `nmap -sV [foreign IP]`: fingerprint the destination
3. `curl -v [foreign IP]`: inspect response headers
4. VirusTotal / AlienVault OTX: threat intel lookup
5. Escalate with documented findings

### 6. curl -v https://google.com
Verbose HTTP/HTTPS request showing the full transaction including the TLS handshake and response headers.

**Key observations:**
- IPv6 attempted first, failed (NAT network limitation), fell back to IPv4 automatically
- TLSv1.3 handshake completed: Client Hello, Server Hello, Certificate, Verified, Finished
- Certificate: CN=*.google.com, issuer Google Trust Services, valid through Aug 2026
- Response: HTTP 301 redirect to https://www.google.com
- Headers noted: x-frame-options, CSP policy, server: gws

**SOC use:** manual URL investigation, certificate inspection during phishing analysis, header analysis, verify TLS version on suspicious connections

**Certificate red flags to check:**
- Expired date
- Issuer is unknown or untrusted CA
- Subject doesn't match domain being accessed
- Self-signed on a site claiming to be a major service

---

## Key Takeaways
- netstat/ss perspective is inside looking out: your machine's own connections
- nmap perspective is outside looking in: what other hosts expose to the network
- These tools form the first 5 minutes of any Linux host triage in a SOC investigation
- Port 80 left open on Ubuntu-Lab, remediation needed

## Remediation
```bash
sudo ufw deny 80
sudo ufw status
```

---

## What I Learned Today
- ping, traceroute, nmap, netstat, ss, and curl each answer a different question, and knowing which one to reach for first is most of the job
- netstat/ss show what your own machine is doing, nmap shows what the network sees from outside
- Leftover open ports from earlier labs are exactly the kind of thing a real asset inventory scan would catch, which is why the UFW cleanup mattered

**Big Picture:** These six tools cover the first five minutes of almost any host triage in a SOC. Getting fast and comfortable with all of them together matters more than mastering any one in isolation.
