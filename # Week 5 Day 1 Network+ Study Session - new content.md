# Network+ Study Session — June 15, 2026

## Practice Exam Results

Score: 58% (53/90) — 1 hour 30 minutes

| Domain | Score |
|--------|-------|
| 1 - Networking Concepts | 55% |
| 2 - Network Implementation | 61% |
| 3 - Network Operations | 50% |
| 4 - Network Security | 54% |
| 5 - Network Troubleshooting | 71% |

Target score before exam day: 75%+. Strongest domain: Troubleshooting.
Weakest: Network Operations and Network Security. Gaps in Domains 1 and 2
are content-driven — not all content videos completed yet. Need to get through more content first.

---

## Topics Reviewed

### Networking Devices

| Device | Layer | Key Detail |
|--------|-------|------------|
| Router | L3 | Forwards by IP address |
| Layer 3 Switch | L2+L3 | Switch + router in one device |
| Switch | L2 | Forwards by MAC address, hardware via ASIC |
| Firewall | L3 | Traditional = port filtering, NGFW = application filtering; also VPN/NAT |
| IDS | - | Detects and alerts only |
| IPS | - | Detects and blocks |
| Load Balancer | - | Traffic distribution + caching + SSL offload + TCP offloading + QoS + auto failover |
| Proxy | - | Requests on user's behalf; explicit vs transparent; caching + URL filtering + malware scanning |
| NAS | - | Network-Attached Storage; file-level access; whole file transfers |
| SAN | - | Storage Area Network; block-level access; only changed blocks transfer |
| Access Point | L2 | Bridges 802.11 wireless to 802.3 ethernet |
| Wireless LAN Controller | - | Centralized management of all APs from single console |

---

### Network Services

| Term | Definition |
|------|------------|
| CDN | Content Delivery Network; geographically distributed caching; serves content from nearest location |
| VPN Concentrator | Central point for all VPN connections; high-speed encryption/decryption; often integrated into NGFW |
| QoS | Quality of Service; also called traffic shaping or packet shaping; prioritizes voice/video over file transfers |
| TTL (IP) | Measured in hops; router decrements by 1 each hop; hits zero = packet dropped; Windows = 128, macOS/Linux = 64 |
| TTL (DNS) | Measured in seconds; controls how long DNS resolution stays cached locally |

---

### TCP vs UDP

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented | Connectionless |
| Handshake | SYN, SYN-ACK, ACK | None |
| Reliability | Acknowledged delivery | Unconfirmed delivery |
| Flow control | Yes | No |
| Error recovery | Yes | No |
| Use case | File transfer, web, email | Voice, video, DNS |

- Non-ephemeral ports: 0–1023 — servers, permanent, well-known
- Ephemeral ports: 1024–65535 — clients, temporary, randomly assigned
- TCP port 80 and UDP port 80 are different ports — separate namespaces
- Multiplexing: multiple apps between same two devices using different port numbers — Layer 4

---

### Traffic Types

| Type | Relationship | Example |
|------|-------------|---------|
| Unicast | One to one | Web browsing, file transfer, email |
| Multicast | One to many subscribers | Stock feeds, routing updates, streaming |
| Anycast | One to nearest of many | Anycast DNS |
| Broadcast | One to all | ARP requests, routing updates |

IPv6 removed broadcast completely — replaced with multicast.

---

### Tunneling and VPN Protocols

**ICMP** — not TCP or UDP; used by ping; also returns network unreachable and TTL exceeded messages

**GRE** — Generic Routing Encapsulation; creates tunnels; no encryption; requires additional VPN protocols on top

**IPSec**
- AH (Authentication Header) — integrity and authentication only, no encryption
- ESP (Encapsulation Security Payload) — encryption + authentication; used in most implementations
- Transport mode — original IP header unencrypted; destination visible if captured
- Tunnel mode — original IP header encrypted; new IP header added; most secure; most common

**IKE — Internet Key Exchange**
- Phase 1: Diffie-Hellman, shared secret, UDP port 500, ISAKMP tunnel
- Phase 2: cipher negotiation, key sizes, Security Associations established
- Data transfer happens after both phases complete

---

### Network Topologies

| Topology | Key Detail |
|----------|------------|
| Star / Hub-and-spoke | Central switch; all devices connect to it; most common LAN design |
| Mesh | Multiple paths; redundancy + load balancing; most common in WANs |
| Hybrid | Combination of multiple topologies |
| Spine and Leaf | Spine connects to all leafs; leafs don't connect to each other; top-of-rack switching |
| Point-to-point | Single link between two locations; T1/T3; campus building links |

---

### Network Architecture

**Three-Tier**
- Core — servers, applications, databases, critical resources
- Distribution — redundancy and connectivity between core and access
- Access — where users connect; per-floor switches

**Collapsed Core** — core + distribution combined; cheaper and simpler; less redundant

**Traffic flow**
- East-West — traffic within the same data center; lower security scrutiny
- North-South — traffic entering or leaving the data center; higher security scrutiny

---

### SDN and Cloud Networking

**Three Planes**
- Data plane — forwards traffic; NAT, encryption, trunking
- Control plane — routing/switching tables; forwarding decisions
- Management plane — SSH, console, web GUI

**SD-WAN** — application-aware WAN for cloud environments; zero-touch provisioning;
transport agnostic; central policy management

**VXLAN** — Virtual Extensible LAN; 16 million virtual networks vs VLANs 4,000;
runs over Layer 3; VTEP handles encapsulation/decapsulation across data centers

**Zero Trust** — never trust, always verify; every user/device/application untrusted
by default; adaptive identity; least privilege

**SASE** — Secure Access Service Edge; next-gen VPN; security moved to cloud;
client on every device; bundles zero trust + firewall as a service + DNS security + QoS

---

## Weak Areas Identified

- CDN definition — confused with authoritative DNS server
- IKE Phase 2 — thought it was data transfer, it is still setup
- Control plane vs data plane — mixed up definitions
- IPv6 removed broadcast — replaced with multicast
