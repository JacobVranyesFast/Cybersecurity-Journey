# Routing Lab, Nmap Scanning & Physical Firewall Setup

**Date:** June 4, 2026
**Platform:** Home Lab (Ubuntu, Kali) + Physical Hardware + Professor Messer N10-009
**Domain:** Network+ N10-009 Domain 1, 2

---

## Professor Messer: Routing & Switching

Watched and studied Domain 2 content covering:
- Static vs dynamic routing
- VLANs: segmenting networks logically
- Spanning Tree Protocol (STP): prevents switching loops

### Key Concepts

**Static vs Dynamic Routing:**
- Static: manually configured routes, admin sets each path. Predictable, no overhead, doesn't adapt to changes. Used in small networks or specific fixed paths.
- Dynamic: routers learn routes automatically via protocols. OSPF, BGP, and EIGRP share routing info between routers and adapt to network changes automatically. Used in large enterprise and internet routing.

**VLANs (Virtual Local Area Networks):**
Logically segment a physical network into separate networks. Devices on different VLANs cannot communicate without a router or Layer 3 switch, even on the same physical switch. Used to separate departments, isolate IoT devices, and segment lab networks by trust level.

**Spanning Tree Protocol (STP):**
Prevents Layer 2 switching loops. If two switches are connected by multiple paths, STP blocks redundant paths to prevent broadcast storms. Elects a root bridge, calculates shortest paths, blocks all other paths.

---

## Lab 1: Static Routing (Ubuntu VM)

Practiced manually adding and removing static routes on Ubuntu to understand how routing tables work.

### Commands Run
```bash
# View current routing table
ip route

# Add static route
sudo ip route add 10.10.10.0/24 via 10.0.2.1

# Verify route was added
ip route

# Remove static route
sudo ip route del 10.10.10.0/24 via 10.0.2.1

# Verify removal
ip route

# Trace route to google
traceroute google.com
```

### Routing Table Before Adding Route
```
default via 10.0.2.1 dev enp0s3 proto static
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.50
```

### After Adding Static Route
```
default via 10.0.2.1 dev enp0s3 proto static
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.50
10.10.10.0/24 via 10.0.2.1 dev enp0s3
```

### Key Concepts Learned
- Direct route: network directly attached, no gateway needed (proto kernel = OS added automatically)
- Default route: catch-all, everything unknown goes to gateway (proto static = manually configured)
- Static route: manually added path to a specific network

Route priority, most specific match wins:
- `10.10.10.0/24`: matches only 10.10.10.x (most specific)
- `10.0.2.0/24`: matches only 10.0.2.x (specific)
- `default`: matches everything else (least specific)

### Traceroute
First hop resolved to the CyberLab gateway. Subsequent hops redacted (ICMP blocked by ISP routers), which is normal behavior. Traffic still reached Google, confirming routing works end to end despite the redacted intermediate hops.

### SOC Relevance
Static route analysis is a key step when investigating a compromised host. Unexpected routes added to a routing table can redirect traffic through an attacker-controlled device, a man-in-the-middle technique at the network layer.

---

## Lab 2: Nmap Network Scanning (Kali VM)

### Command 1: Full Network Discovery
```bash
sudo nmap 10.0.2.0/24
```
Results:
- 10.0.2.1: Gateway, ports 135 (RPC), 445 (SMB) open
- 10.0.2.2: VirtualBox service, all ports closed
- 10.0.2.5: Kali (self), port 22 (SSH) open
- 10.0.2.50: Ubuntu, ports 22 (SSH), 80 (HTTP) open

### Command 2: SYN Scan
```bash
sudo nmap -sS 10.0.2.50
```
Same ports found. A SYN scan sends SYN, receives SYN-ACK, then sends RST, never completing the handshake. Stealthier than a TCP Connect scan, leaves less evidence in target logs.

### Command 3: Specific Port Scan
```bash
nmap -p 22,80,443 10.0.2.50
```
Results:
- 22/tcp: open, SSH
- 80/tcp: open, HTTP/Apache
- 443/tcp: filtered, HTTPS, UFW dropping probes silently

Filtered vs closed distinction:
- Closed: target sends RST back, port reachable but no service
- Filtered: no response, firewall silently dropping packets, reveals less to attacker, more secure response

### Command 4: Version Detection
```bash
sudo nmap -sV 10.0.2.50
```
Identifies exact service names and version numbers on open ports, used to find vulnerable software versions.

### SOC Relevance
nmap is used defensively to audit networks for unexpected open ports. Running regular scans and comparing against a known baseline immediately reveals unauthorized services or new attack surface. Filtered ports indicate firewall protection is working correctly.

---

## Physical Lab: Netgear FVS318 Firewall

### Hardware Acquired
Netgear ProSafe VPN Firewall FVS318 from the University of Minnesota reuse warehouse. 8 LAN ports, 1 WAN port, VPN capable. Also acquired a Netgear FS108Pv3 PoE switch.

### Power Supply
No power adapter included. Sourced an Axis Communications 12V 3.33A power supply. Spliced the bare wire output to a barrel connector cable via soldering. The firewall requires 12V 1.2A; a 3.33A supply is compatible since the device only draws what it needs.

### Configuration
Accessed the admin panel at http://192.168.0.1 after setting a static IP on the PC ethernet adapter (192.168.0.3) to reach the firewall's subnet directly.

Default firewall rules observed:
- Outbound default: ALLOW all (permissive by default)
- Inbound default: BLOCK all (restrictive by default)

### First Firewall Rule Written
Created an outbound block rule for port 23 (Telnet):
- Service: Telnet (port 23)
- Action: BLOCK
- Direction: Outbound
- Reason: Telnet transmits credentials in plaintext, should never be allowed on any network

### Troubleshooting: Default Gateway Conflict
Setting a static IP on Ethernet 3 for firewall access caused Windows to route all internet traffic through the Netgear instead of the Xfinity gateway. Resolved by setting the main ethernet adapter back to DHCP.

Key lesson: two network adapters with two default gateways causes routing conflicts. Windows sends traffic to the wrong gateway.

Correct lab architecture going forward:
- Jimba-PC main ethernet: Xfinity (home internet)
- USB ethernet adapter: Netgear firewall (lab access)
- No default gateway on the firewall adapter

### SOC Relevance
Understanding default firewall posture (deny inbound, allow outbound) is foundational SOC knowledge. Every enterprise firewall starts from this baseline. Custom rules add specific exceptions. Reviewing firewall rules for unexpected allows or missing denies is a standard SOC audit task.

---

## Flashcard Review
Reviewed Anki decks covering Network+ Domain 1 and 2 content: ports, protocols, OSI layers, subnetting, network devices, and routing concepts.

---

## What I Learned Today
- Static routes tell a machine exactly where to send traffic for a specific network: manually added, manually removed, immediately effective
- The most specific route always wins over the default route
- SYN scans are stealthier than TCP Connect because they never complete the handshake
- Filtered ports reveal less information to attackers than closed ports: silent drops are more secure
- Physical firewall default posture is allow all outbound, block all inbound; explicit rules override defaults
- Multiple default gateways on one machine causes routing conflicts, only one adapter should have a default gateway set
- Soldered a power supply connection for real hardware: voltage matching (12V) and polarity are critical before connecting to any device

**Big Picture:** Whether it's a routing table, an nmap scan, or a physical firewall's rule set, the job is the same: know what the baseline should look like so anything unexpected stands out immediately.
