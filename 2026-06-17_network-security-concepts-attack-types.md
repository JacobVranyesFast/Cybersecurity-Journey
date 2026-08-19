# Network Security Concepts & Attack Types

**Date:** June 17, 2026
**Platform:** Professor Messer N10-009 + KC7
**Domain:** Network+ N10-009 Domain 4: Network Security

---

## What I Did
- Watched Professor Messer N10-009 Domain 4.1: Network Security Concepts
- Watched Professor Messer N10-009 Domain 4.2: Attack Types
- Self-quizzed on today's content, no formal practice exam score recorded
- Continued KQL practice in KC7's Valadoria room

## Topics Covered

### 4.1: Network Security Concepts
- States of data, digital certificates, certificate authorities
- Identity and access management (IAM)
- Authentication methods: SSO, RADIUS, LDAP, SAML
- Honeypots and honeynets
- Risk, vulnerability, exploit, threat, and the CIA triad
- Regulatory compliance: data localization, GDPR, PCI DSS
- Segmentation: IoT, SCADA, OT, guest networks, BYOD

### 4.2: Attack Types
- Denial of Service and DoS amplification
- VLAN hopping: switch spoofing, double tagging
- MAC flooding: turning a switch into a hub
- ARP poisoning and DNS poisoning: on-path attacks
- Rogue DHCP servers, rogue access points, evil twins
- Social engineering: phishing, shoulder surfing, tailgating, piggybacking, dumpster diving
- Malware types, including ransomware

## KC7: Valadoria Room

Continued hands-on KQL practice through KC7's Valadoria scenario.

---

## What I Learned Today
- The CIA triad (confidentiality, integrity, availability) is the lens the rest of Domain 4's concepts map back to
- Attack types finally clicked once mapped onto protocols already covered, not as standalone material. DNS poisoning only makes sense against the backdrop of normal DNS resolution: the attack inserts itself into that exact query/response flow and alters the response or spoofs the source
- Same pattern applies to physical infrastructure concepts covered earlier (switches, access points): most Domain 4 attacks are abuses of how a protocol or device is supposed to work, not a separate category of exotic technique

**Big Picture:** Almost every attack type in Domain 4 is a normal protocol being used against its own trust assumptions, which is exactly the mindset a SOC analyst needs when triaging an alert that looks like "normal" traffic doing something it shouldn't.

## Next Steps
- Take a Domain 4 practice question set to get an actual score instead of a self-quiz
- Continue KC7 Valadoria
