# Week 1 — Cryptography Fundamentals & Hashing

**Date:** May 22, 2026  
**Platform:** TryHackMe + Home Lab Terminal  
**Domain:** Security+ SY0-701 Domain 1 — Threats, Attacks & Vulnerabilities

## What I Studied
Worked through TryHackMe cryptography rooms covering the fundamentals 
of how data is protected and how attackers break that protection.

## Topics Covered

### Public Key & Private Key Cryptography
- How asymmetric encryption works — public key encrypts, 
  private key decrypts
- Why this solves the key distribution problem that symmetric 
  encryption has
- Real-world application: HTTPS, SSH authentication, digital signatures

### Hashing
- What hashing is — one-way function that produces a fixed-length 
  output from any input
- Common hashing algorithms: MD5, SHA-1, SHA-256
- Why hashes are used for password storage instead of 
  storing plaintext
- How even a small change to input produces a completely 
  different hash (avalanche effect)

### Password Cracking with Hashcat
- How Hashcat works — takes a hash and attempts to find the 
  original input using wordlists and rules
- Dictionary attacks vs brute force attacks
- Why weak passwords fall quickly even when hashed
- Defense takeaway: salting hashes makes cracking significantly harder

### Terminal & OpenVPN
- Connected to TryHackMe attack boxes via OpenVPN using the terminal
- Used SSH to connect to remote systems
- Practiced navigating and running tools directly from the command line
  rather than a GUI

## Commands Used
hashcat -m 0 hash.txt wordlist.txt       # MD5 crack
hashcat -m 1800 hash.txt wordlist.txt    # SHA-512 crack
ssh user@[target-ip]
sudo openvpn [file].ovpn

## Key Takeaways
- Hashing is not encryption — you cannot reverse a hash, 
  only attempt to crack it
- Public key cryptography is the foundation of almost every 
  secure connection on the internet
- Hashcat shows exactly why password complexity and salting matter — 
  simple passwords crack in seconds
- Connecting via OpenVPN + SSH is the standard way SOC analysts 
  access remote systems and lab environments

## SOC Relevance
- Hash analysis appears in malware investigations — file hashes 
  are used to identify known malicious files in VirusTotal
- Password cracking knowledge helps understand attacker TTPs 
  and why account lockout policies matter
- SSH log monitoring (Event ID equivalent on Linux: /var/log/auth.log) 
  is a daily SOC task

## Challenges
- Getting OpenVPN connected and SSH working from the terminal 
  took some troubleshooting — good hands-on practice with 
  real tool usage
