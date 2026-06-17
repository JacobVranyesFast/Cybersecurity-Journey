# Network+ Study Log — June 16, 2025

## Topics Covered
- Physical installations (MDF, IDF, rack standards, power, cooling)
- Disaster recovery (RTO, RPO, MTTR, MTBF, site types, redundancy)
- DHCP (DORA process, relay, reservations, lease timers)
- IPv6 (SLAAC, NDP, DAD)
- DNS (record types, recursive queries, security extensions)
- Time protocols (NTP, NTS, PTP)
- KQL basics (KC7 lab — PassiveDns queries)

---

## Physical Installations

**MDF (Main Distribution Frame)** — central point of the network. WAN connections terminate here alongside internal LAN connections. Single room, often the data center.

**IDF (Intermediate Distribution Frame)** — located on separate floors or in separate buildings. Contains switches, patch panels, and connects back to the MDF via uplinks.

Traffic path: end user → IDF switch → MDF core switch → internet

**Rack standards:**
- Standard width: 19 inches
- 1U = 1.75 inches
- Standard rack height: 42U

**Cooling — hot aisle / cold aisle:**
- Cold aisle = front of servers, intake side
- Hot aisle = back of servers, exhaust side
- HVAC pulls hot air from ceiling, recools it, pushes cold air back under raised floor or from above

**Power:**
- UPS (Uninterruptible Power Supply) — three types:
  - Offline/Standby — activates after power loss, small gap
  - Line-interactive — compensates for brownouts (voltage drop)
  - Online/Double-conversion — always running from battery, preferred in data centers
- PDU (Power Distribution Unit) — rack-mountable, remotely managed via IP, can power cycle individual devices

**Fiber:**
- Service loop = extra coiled fiber left in distribution panel for future moves
- Bend radius must not be exceeded or fiber breaks

---

## Disaster Recovery

| Term | Definition |
|------|------------|
| RTO (Recovery Time Objective) | How long it takes to restore service after an outage |
| RPO (Recovery Point Objective) | How much data was lost, measured in time back to last backup |
| MTTR (Mean Time to Repair) | Average time to fix a failed component |
| MTBF (Mean Time Between Failures) | Average expected lifespan before a component fails |

**Site types:**
- **Cold site** — empty building, no hardware or data. Cheapest, longest recovery time.
- **Warm site** — partial infrastructure (racks, some hardware). Needs data recovery before operational.
- **Hot site** — exact replica of production. Continuous replication. Fastest recovery, most expensive.

**Redundancy configurations:**
- **Active-passive** — one device active, one on standby. Passive monitors active constantly, takes over on failure. Configs must be identical including session and routing tables.
- **Active-active** — both devices running simultaneously. More complex to engineer but uses full capacity of both devices.

**FHRP (First Hop Redundancy Protocol)** — umbrella protocol for router redundancy.
- HSRP = Cisco proprietary
- VRRP = open standard
- Both use a **VIP (Virtual IP)** that floats between devices — end devices always point to the VIP as default gateway.

**Testing:**
- **Tabletop exercise** — conference room walkthrough of DR plan, no physical movement
- **Validation test** — full physical DR test, proves the plan actually works

---

## DHCP

**DORA process:**
1. **Discover** — client broadcasts from 0.0.0.0 to 255.255.255.255 (UDP 68 → 67)
2. **Offer** — DHCP server broadcasts offer (UDP 67 → 68)
3. **Request** — client broadcasts requesting specific offer (UDP 68 → 67)
4. **Acknowledge** — server confirms, client applies IP config (UDP 67 → 68)

**DHCP Relay** — needed when DHCP server is on a different subnet. Router configured as relay intercepts local broadcast, converts to unicast, forwards to DHCP server IP. Required because broadcasts don't cross routers.

**Address reservation** — ties a specific IP to a device's MAC address in the DHCP server. Device always receives the same IP. Managed centrally. Different from exclusion (which just removes an IP from the pool entirely).

**Lease timers:**
- **T1 = 50% of lease time** — client attempts renewal with original DHCP server
- **T2 = 87.5% (7/8) of lease time** — client attempts renewal with any available DHCP server

**DHCP options** — additional config pushed with IP assignment (DNS servers, default gateway, VoIP server, HTTP proxy, etc.)

---

## IPv6 — SLAAC and NDP

**SLAAC (Stateless Address Autoconfiguration)** — IPv6 device assigns itself an IP without a DHCP server.

Process:
1. Device sends **router solicitation** via NDP to find local router
2. Router responds with **router advertisement** containing 64-bit subnet prefix
3. Device generates last 64 bits (interface ID) from MAC address with FF:FE inserted, or randomized
4. **DAD (Duplicate Address Detection)** confirms no address conflict before use

**NDP (Neighbor Discovery Protocol)** — IPv6 replacement for ARP. Uses multicast instead of broadcast — more efficient.

Key difference from IPv4: no DHCP server required, no lease time, no central address tracking.

---

## DNS

**Record types:**

| Record | Purpose |
|--------|---------|
| A | IPv4 address for a hostname |
| AAAA | IPv6 address for a hostname |
| CNAME | Alias pointing to another hostname (canonical name) |
| MX | Mail exchanger — points to mail server hostname |
| PTR | Reverse lookup — IP address → hostname |
| TXT | Human-readable text — used for SPF and DKIM |
| NS | Name server records — where DNS zone is hosted |
| SOA | Start of Authority — zone metadata |

**Email security records (stored as TXT):**
- **SPF (Sender Policy Framework)** — specifies which mail servers are authorized to send on behalf of a domain
- **DKIM (Domain Keys Identified Mail)** — digitally signs outgoing email; public key stored in DNS TXT record for verification

**Forward vs reverse lookup:**
- Forward: name → IP (uses A/AAAA records)
- Reverse: IP → name (uses PTR records, configured separately)

**Recursive DNS query flow:**
1. Client queries local DNS server
2. Local DNS queries root server → gets .com nameserver location
3. Local DNS queries .com nameserver → gets authoritative nameserver for domain
4. Local DNS queries authoritative nameserver → gets IP
5. Answer returned to client and cached

**Authoritative vs non-authoritative:**
- Authoritative = primary DNS server that holds the actual zone records
- Non-authoritative = cached copy (secondary server or resolver cache)

**DNS security:**
- **DNSSEC** — digitally signs DNS responses so clients can verify authenticity
- **DoT (DNS over TLS)** — encrypts DNS traffic, TCP port 853
- **DoH (DNS over HTTPS)** — encrypts DNS traffic over HTTPS, TCP port 443

**TTL** — time in seconds a non-authoritative server caches a DNS record before expiring

---

## Time Protocols

| Protocol | Port | Accuracy | Notes |
|----------|------|----------|-------|
| NTP (Network Time Protocol) | UDP 123 | ~10ms | Standard time sync, sends in clear |
| NTS (Network Time Security) | — | ~10ms | Adds authentication to NTP via TLS key exchange |
| PTP (Precision Time Protocol) | — | Nanoseconds | Hardware-based, used in industrial/financial environments |

**NTS flow:** TLS handshake with key exchange server → receive cookie → NTP request with cookie → trusted timestamp received

**Security note:** Kerberos authentication fails if client/server clocks differ by more than 5 minutes — correct time is a security requirement, not just operational preference.

---

## KQL Lab — KC7

Practiced basic KQL queries against PassiveDns table:

```kql
// Find distinct domains containing "hire"
PassiveDns
| where domain contains "hire"
| distinct domain
| count
```

```kql
// Find employee IP address
Employees
| where name contains "Lois Lane"
| project name, ip_addr
```

Key KQL operators used today:
- `where` — filter rows
- `contains` — substring match
- `distinct` — deduplicate
- `count` — count rows
- `project` — select specific columns