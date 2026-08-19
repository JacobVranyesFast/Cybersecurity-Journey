# John the Ripper & Network+ Baseline Exam

**Date:** May 29, 2026
**Platform:** TryHackMe (John the Ripper room) + Dion N10-009 Practice Exam
**Domain:** Security+ SY0-701 Domain 1 | Network+ N10-009

---

## TryHackMe: John the Ripper Room

Completed full John the Ripper room covering wordlist attacks, custom rules, and cracking Linux password files.

### Core Commands
```bash
# Basic crack
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Specify format
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt

# Show cracked results
john hash.txt --show

# Custom rules
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --rules=THMRules

# Crack zip/rar files
zip2john file.zip > zip.hash
john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt

rar2john file.rar > rar.hash
john rar.hash --wordlist=/usr/share/wordlists/rockyou.txt

# SHA256 hash of a file
sha256sum document.pdf
```

### Unshadowing /etc/shadow
John requires passwd and shadow combined before cracking Linux password hashes:
```bash
unshadow /etc/passwd /etc/shadow > unshadowed.txt
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
john unshadowed.txt --show
```

### Custom Rule Syntax
- `Az"[A-Z]"` - append capital letter to end of word
- `Az"[0-9]"` - append number to end
- `c` - capitalize first letter

### SOC Relevance
- Weak passwords crack in seconds against rockyou.txt
- Salting makes offline cracking significantly harder
- Any process reading /etc/shadow outside normal login activity is a critical detection alert
- Understanding cracking techniques informs why password policies and MFA exist

---

## Network+ N10-009 Practice Exam

**Score: 65%, 59/90**

First full baseline exam. Passing score is around 80% (720/900). Target for next attempt: 75%+.

### Weak Areas to Review
- Subnetting: /28 host count, network/broadcast addresses
- Port numbers: NTP (123), SNMP (161), TFTP (69)
- nmap scan types: no sudo means TCP Connect, not SYN
- APIPA: 169.254.x.x means DHCP server unreachable
- Filtered vs closed ports
- Whaling vs spear phishing vs phishing
- Private IP range boundaries

### Next Steps
- Daily Anki review of all missed topics
- Professor Messer videos for weak domains
- Continue subnetting on subnetipv4.com
- Target 75%+ on next practice exam

---

## What I Learned Today
- Rockyou.txt cracks weak passwords almost instantly, which is exactly why password policy and MFA exist
- unshadow has to combine passwd and shadow before John can work on Linux hashes
- A 65% baseline on the first full practice exam is a normal starting point, not a red flag
- Filtered vs closed, and the different phishing variants, are the kind of terms that need cold recall on exam day

**Big Picture:** Password cracking tools like John the Ripper show exactly why weak passwords fail so fast in a breach, and that's the same reasoning behind every password policy a SOC enforces.
