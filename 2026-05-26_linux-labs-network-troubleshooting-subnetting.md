# Linux Labs, Network Troubleshooting & Subnetting

**Date:** May 26, 2026
**Platform:** Home Lab (Ubuntu VM, Kali VM, VirtualBox), subnetipv4.com, Professor Messer N10-009
**Domain:** Network+ N10-009 Domain 1, Security+ SY0-701 Domain 3

---

## What I Did Today
Full hands-on lab session covering Linux fundamentals, network troubleshooting, and subnetting calculations.

---

## Lab 1: File Permissions (Ubuntu via SSH)
Practiced the Linux file permission system using chmod on the Ubuntu VM.

### Commands Run
```bash
mkdir permissions-lab
cd permissions-lab
touch file1.txt file2.txt file3.txt
chmod 777 file1.txt
chmod 644 file2.txt
chmod 600 file3.txt
ls -la
```

### Permission Breakdown
- `-rwxrwxrwx` = 777: owner/group/others all have read+write+execute
- `-rw-r--r--` = 644: owner read/write, everyone else read only
- `-rw-------` = 600: owner read/write only, nobody else can access

### How to Read ls -la Output
```
rwx rwx rwx
^ ^   ^   ^
| |   |   └── others (everyone else)
| |   └────── group
| └────────── owner
└──────────── file type (- = file, d = directory)
```

### Number System
- r (read) = 4
- w (write) = 2
- x (execute) = 1
- Add them up: 7=rwx, 6=rw-, 4=r--, 0=---

### SOC Relevance
Finding a script with 777 permissions in an unusual location is an immediate red flag during incident response. SSH private keys should always be 600. If they're not, the system may have been tampered with.

---

## Lab 2: User Management & /etc/shadow (Ubuntu via SSH)
Demonstrated the Linux privilege model using user creation and /etc/shadow access control.

### Commands Run
```bash
sudo useradd testuser
sudo passwd testuser
su - testuser
cat /etc/shadow        # Permission denied as testuser
exit
sudo cat /etc/shadow   # Works as root via sudo
sudo userdel testuser  # Cleanup
```

### Key Findings
- Regular users cannot read /etc/shadow: permission denied
- Root access via sudo can read /etc/shadow
- /etc/shadow stores hashed passwords, not plaintext
- Hash format: $6$ = SHA-512 (current standard), $1$ = MD5 (weak)

### Why /etc/shadow Is Security-Critical
/etc/shadow stores every user's hashed password. If an attacker gets this file, they can run it through Hashcat offline. Weak passwords crack in seconds. Strong passwords plus salting makes cracking significantly harder.

### Attack Chain
1. Attacker gains limited system access
2. Escalates privileges to root
3. Copies /etc/shadow off the system
4. Runs Hashcat against hashes with a GPU and wordlist
5. Cracks passwords and logs in as those users
6. If the root password cracks, full system compromise

### SOC Relevance
Monitor /var/log/auth.log for unusual sudo attempts and privilege escalation. Any process reading /etc/shadow outside normal login activity is a serious incident indicator.

---

## Lab 3: Network Troubleshooting & SSH Setup

### Problem
Kali Linux losing its IP address after VM resume. DHCP lease expiring when the VM is paused/suspended.

### Diagnosis
```bash
ip addr          # showed no IPv4 on eth0
ip link show     # confirmed eth0 existed but had no IP
```

### Fix Applied
```bash
sudo ip link set eth0 down
sudo ip link set eth0 up
sudo dhcpcd eth0
```

dhcpcd output confirmed DHCP lease acquired:
```
eth0: offered 10.0.2.6 from 10.0.2.2
eth0: leased 10.0.2.6 for 600 seconds
eth0: adding route to 10.0.2.0/24
eth0: adding default route via 10.0.2.1
```

### SSH Setup on Ubuntu
Installed and enabled OpenSSH server on the Ubuntu VM so Kali can connect remotely, matching a real-world SOC workflow.

```bash
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh
sudo ufw allow 22
```

### SSH Connection from Kali
```bash
ssh user@10.0.2.3
```
Now controlling Ubuntu from the Kali terminal with full scrolling. This is how SOC analysts manage remote Linux systems in production environments.

### tcpdump: Live Traffic Capture
```bash
sudo tcpdump -i enp0s3 icmp
```
Captured live ICMP ping packets from Kali hitting Ubuntu in real time, the command line equivalent of Wireshark.

### Full Lab Network Scan Result
```bash
sudo nmap -sn 10.0.2.0/24
```
- 10.0.2.1: CyberLab gateway (virtual router)
- 10.0.2.2: VirtualBox DHCP/DNS service
- 10.0.2.3: Ubuntu-Lab
- 10.0.2.6: Kali-Linux
- 10.0.2.15: WinServer-Lab

---

## Lab 4: Subnetting Practice

### Resources Used
- Professor Messer N10-009 subnetting videos (Classful Subnetting + IPv4 Subnetting)
- subnetipv4.com: hours of hands-on calculation practice

### Subnet Ranges Practiced
/17, /18, /19, /20, /21, /22, /23, /24, /25, /26, /27, /28

### Core Formula
Usable hosts = 2^(host bits) - 2
- /24 = 8 host bits = 2^8 - 2 = 254 hosts
- /25 = 7 host bits = 2^7 - 2 = 126 hosts
- /26 = 6 host bits = 2^6 - 2 = 62 hosts
- /27 = 5 host bits = 2^5 - 2 = 30 hosts
- /28 = 4 host bits = 2^4 - 2 = 14 hosts

### Subnet Mask Reference
| Prefix | Mask |
|--------|------|
| /17 | 255.255.128.0 |
| /18 | 255.255.192.0 |
| /19 | 255.255.224.0 |
| /20 | 255.255.240.0 |
| /21 | 255.255.248.0 |
| /22 | 255.255.252.0 |
| /23 | 255.255.254.0 |
| /24 | 255.255.255.0 |
| /25 | 255.255.255.128 |
| /26 | 255.255.255.192 |
| /27 | 255.255.255.224 |
| /28 | 255.255.255.240 |

### Example Worked: 192.168.1.0/26
- Subnet mask: 255.255.255.192
- Network address: 192.168.1.0
- First usable host: 192.168.1.1
- Last usable host: 192.168.1.62
- Broadcast: 192.168.1.63
- Usable hosts: 62

### Key Insight
Each time you go up one prefix (e.g. /24 to /25) you cut the number of hosts in half. Each time you go down one prefix you double them.

---

## Challenges
- Kali kept losing its IP address due to DHCP lease expiry on VM resume. Troubleshot through multiple approaches (ip link bounce, dhcpcd, nmcli static IP attempt) before finding a working solution
- SSH connection to Ubuntu required installing openssh-server, enabling a UFW rule for port 22, and troubleshooting firewall configuration
- Static IP configuration via nmcli caused connection timeouts. Learned that DHCP vs static IP configuration requires careful network manager handling on Kali

## Next Steps
- Take first full Network+ N10-009 practice exam
- Continue subnetting practice, target: calculate any subnet without hesitation
- Begin Professor Messer routing and switching videos

---

## What I Learned Today
- Linux file permissions are a three-layer security model (owner/group/others). Understanding this is essential for both Linux administration and SOC investigations
- /etc/shadow is one of the most security-critical files on any Linux system. Privilege escalation attacks often target it
- DHCP leases expire. Understanding why a host loses its IP and how to recover it is core network troubleshooting
- SSH is the standard way to manage remote Linux systems. SOC analysts rarely interact with servers directly
- tcpdump is the command line equivalent of Wireshark, essential for live traffic analysis
- Subnetting is mechanical once you understand the bit math. Practice makes it automatic

**Big Picture:** File permissions, /etc/shadow, DHCP troubleshooting, and subnetting all map directly to Tier 1 work: spotting bad permissions, reading auth logs, and placing an alert's source IP on the network.
