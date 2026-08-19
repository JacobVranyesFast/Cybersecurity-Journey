# Wireshark, tcpdump, nmap & Subnetting

**Date:** May 27, 2026
**Platform:** TryHackMe (tcpdump + nmap rooms), Home Lab
**Domain:** Network+ N10-009, Security+ SY0-701 Domain 3

---

## Wireshark: Live Traffic Analysis (Windows 11 Host)
Captured live traffic browsing to http://example.com. Applied filters to isolate traffic at each OSI layer:

- **dns**: observed DNS query/response pair. Note: Brave uses DNS over HTTPS by default, so nslookup from Command Prompt was used instead to generate visible port 53 traffic
- **tcp**: observed full 3-way handshake (SYN, SYN-ACK, ACK)
- **http**: captured plaintext GET request and 200 OK response, demonstrating why HTTP is insecure
- **arp**: observed Layer 2 MAC address resolution
- **icmp**: captured ping request/reply pairs

### SOC Relevance
Wireshark is used in SOC investigations to detect unencrypted traffic, unusual DNS queries, port scans, data exfiltration, and malware C2 traffic.

---

## tcpdump: Command Line Traffic Analysis (Kali VM)
Completed the TryHackMe tcpdump room and ran hands-on captures on the home lab.

### Key Commands
```bash
sudo tcpdump -i eth0 -c 50              # capture 50 packets
sudo tcpdump -i eth0 port 80            # filter by port
sudo tcpdump -i eth0 -w capture.pcap    # save to file
tcpdump -r traffic.pcap                 # read from file
tcpdump -r traffic.pcap icmp | wc -l    # count matching packets
tcpdump -r traffic.pcap 'tcp[tcpflags] == tcp-rst'  # filter by flag
tcpdump -r traffic.pcap 'len > 15000'   # filter by packet size
tcpdump -r traffic.pcap arp -e          # show MAC addresses
```
Note: complex filter expressions require single quotes. Double quotes cause syntax errors.

### TCP Flag Reference
- `[S]` = SYN
- `[S.]` = SYN-ACK
- `[.]` = ACK
- `[P.]` = PSH-ACK
- `[F.]` = FIN
- `[R]` = RST

---

## nmap: Network Scanning (TryHackMe Room)
Completed the TryHackMe nmap room. Scanned 10.67.137.11.

### Scan Types
| Type | Command | Sudo | Method |
|------|---------|------|--------|
| TCP Connect | -sT | No | Full 3-way handshake, slower and detectable |
| SYN Scan | -sS | Yes | Sends SYN, gets SYN-ACK, sends RST, stealthier |
| Version Scan | -sV | Yes | Detects service name and version |
| Ping Sweep | -sn | Yes | Discovers live hosts, no port scan |

Key: running nmap without sudo forces TCP Connect because unprivileged users can't send raw packets.

### Target Results: 10.67.137.11
```
22/tcp   open  ssh      OpenSSH 9.6p1 Ubuntu
8008/tcp open  http     lighttpd 1.4.74
```
4 additional open ports (echo, tcpwrapped, daytime, qotd). Web server found on 8008. Accessed via browser to retrieve flag.

---

## Static IP Configuration
Fixed the persistent DHCP issue on both lab VMs:
- **Ubuntu**: edited /etc/netplan/ config, ran `sudo netplan apply`
- **Kali**: edited /etc/network/interfaces directly

Lab network now fully static:
- 10.0.2.5: Kali (static)
- 10.0.2.15: Windows Server
- 10.0.2.50: Ubuntu (static)

---

## Subnetting
Continued further subnetting practice.

---

## What I Learned Today
- Wireshark filters isolate traffic at each OSI layer. dns, tcp, http, arp, and icmp filters each surface a different piece of a connection
- tcpdump does everything Wireshark does from the command line, and pcap files can be filtered after capture with the same flag syntax
- SYN scans (-sS) are stealthier than full TCP connect scans (-sT) because the handshake never completes
- Static IP configuration differs by distro: Ubuntu uses netplan, Kali edits /etc/network/interfaces directly

**Big Picture:** Wireshark and tcpdump are usually the first tools a SOC analyst reaches for when traffic looks wrong. Today built fluency in both, plus the nmap scan types that show up in recon investigations.
