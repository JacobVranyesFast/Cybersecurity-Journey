# Week 3 Day 1 — Linux Fundamentals Labs & Network+ Messer Study

**Date:** june 1, 2026
**Platform:** Home Lab (Ubuntu VM) + Professor Messer N10-009
**Domain:** Network+ N10-009 Domain 1 | Security+ SY0-701 Domain 3

---

## Linux Fundamentals Lab (Ubuntu VM)

Hands-on practice covering essential Linux commands 
for file management, process monitoring, and log analysis.

### File Management
```bash
touch file.txt          # create file
mkdir foldername        # create directory
cp file.txt /path/      # copy file
mv file.txt /path/      # move file
rm file.txt             # remove file
rm -r foldername        # remove directory recursively
```

### Copying Files Between Machines — SCP
```bash
# Copy file FROM remote machine TO local
scp user@10.0.2.50:/path/to/file .

# Copy file FROM local TO remote machine
scp file.txt user@10.0.2.50:/home/user/

# Copy directory recursively
scp -r folder/ user@10.0.2.50:/home/user/
```
SCP (Secure Copy Protocol) transfers files over SSH — 
encrypted, uses port 22. Standard method for moving 
files between Linux systems in enterprise environments.

### Process Management
```bash
ps aux          # list all running processes
top             # live updating process monitor (q to exit)
sleep 300 &     # run process in background
jobs            # list background processes
kill %1         # kill job number 1
kill -9 [PID]   # force kill by process ID
```

### Service Management
```bash
sudo systemctl status ssh     # check service status
sudo systemctl stop ssh       # stop service
sudo systemctl start ssh      # start service
sudo systemctl enable ssh     # start on boot automatically
```

Note: Ubuntu uses both ssh.service and ssh.socket — 
must stop both to fully disable SSH:
```bash
sudo systemctl stop ssh.socket
sudo systemctl stop ssh.service
```

### Log Monitoring
```bash
# Watch syslog live — service events
sudo tail -f /var/log/syslog

# Watch auth.log live — login attempts, sudo, user creation
sudo tail -f /var/log/auth.log

# Search for failed login attempts
sudo grep "Failed password" /var/log/auth.log

# Count failed attempts
sudo grep -c "Failed password" /var/log/auth.log

# Search for sudo usage
sudo grep "sudo" /var/log/auth.log
```

Key distinction:
/var/log/syslog   — service starts/stops, system events
/var/log/auth.log — authentication, SSH logins, sudo, user creation

### SOC Relevance
- ps aux — spot suspicious processes running as root
- auth.log — detect brute force attempts and privilege escalation
- systemctl — verify services haven't been tampered with
- SCP — transfer evidence files off compromised systems for analysis
- tail -f — monitor logs in real time during an active incident

---

## Professor Messer N10-009 — Study Sessions

Watched and answered questions on 4 video segments 
covering core Network+ Domain 1 concepts.

### OSI Model
Layer 7 — Application  — HTTP, HTTPS, DNS, FTP, SMTP
Layer 6 — Presentation — SSL/TLS encryption/decryption
Layer 5 — Session      — connection management
Layer 4 — Transport    — TCP, UDP, port numbers
Layer 3 — Network      — IP addresses, routing
Layer 2 — Data Link    — MAC addresses, switching
Layer 1 — Physical     — cables, signals, bits

Key mappings:
Switch        = Layer 2 (MAC addresses)
Router        = Layer 3 (IP addresses)
Firewall      = Layer 3-4 (IP + ports)
Access Point  = Layer 2 (bridges wireless to ethernet)

### Network Devices
Router        — routes between IP subnets, Layer 3
Switch        — forwards by MAC address, Layer 2
Layer 3 switch — switch with built-in routing
NGFW          — filters by application not just port
IDS           — detects and alerts on attacks
IPS           — detects AND blocks attacks
Load balancer — distributes traffic, removes failed servers
Proxy         — sits between user and internet, filters/caches
NAS           — file-level network storage
SAN           — block-level storage, faster for large files
Access point  — wireless to wired bridge, Layer 2

### CDN, VPN, QoS, TTL
CDN    — caches content geographically, reduces latency
VPN    — encrypted tunnel across insecure networks
VPN concentrator — handles encryption for many simultaneous users
QoS    — prioritizes traffic (VoIP > file transfers)
TTL (IP)  — measured in hops, prevents routing loops
TTL (DNS) — measured in seconds, controls cache duration

Default TTL values:
Windows:    128 hops
Mac/Linux:  64 hops
Typical internet path: 12-16 hops

### Cloud Networking
NFV            — virtual routers, switches, firewalls
VPC            — isolated private cloud network
Transit gateway — connects multiple VPCs (cloud router)
Internet gateway — two-way VPC to public internet
NAT gateway    — outbound only, private resources
VPC endpoint   — connects VPCs across cloud providers
Security group — cloud firewall, rules by port and IP

Key distinction:
Internet gateway = public facing, two-way traffic
NAT gateway      = private resources, outbound only

---

## What I Learned Today

- SCP is the standard Linux tool for secure file transfer 
  between systems — uses SSH encryption on port 22
- Ubuntu uses both ssh.service and ssh.socket — 
  stopping only the service leaves the socket still listening
- auth.log and syslog serve different purposes — 
  both critical for SOC log monitoring
- OSI model maps directly to real devices — 
  switches at Layer 2, routers at Layer 3, applications at Layer 7
- Cloud networking mirrors physical networking — 
  VPCs replace physical networks, transit gateways replace routers,
  security groups replace firewalls
- NAT gateway vs internet gateway is a common cloud 
  misconfiguration — exposing private resources publicly
  