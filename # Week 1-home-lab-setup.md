# Week 1 — Home Lab Setup

**Date:** May 21, 2026
**Platform:** Home Lab — VirtualBox

## What I Built
Set up a 3-VM cybersecurity home lab running on VirtualBox 
on Windows 11. All VMs connected on an internal NAT Network 
called CyberLab (10.0.2.0/24).

## Lab Inventory
| VM | OS | IP Address | Purpose |
|----|----|------------|---------|
| Ubuntu-Lab | Ubuntu 22.04 LTS | 10.0.2.3 | Linux practice, Splunk |
| WinServer-Lab | Windows Server 2022 | 10.0.2.15 | Active Directory, Event logs |
| Kali-Linux | Kali Linux | 10.0.2.5 | Recon, nmap, attacks |

## What I Did
- Installed VirtualBox and Extension Pack on Windows 11
- Created Ubuntu VM (4096MB RAM, 2 CPUs, 50GB storage)
- Created Windows Server 2022 Eval VM (4096MB RAM, 2 CPUs, 60GB storage)
- Imported Kali Linux VirtualBox image
- Created CyberLab NAT Network (10.0.2.0/24) with DHCP enabled
- Assigned all 3 VMs to CyberLab network
- Enabled ICMP firewall rule on Windows Server to allow ping

## Commands Used
ip addr
ping -c 5 10.0.2.15
ping -c 5 10.0.2.3
whoami


## Confirmed Working
- Ubuntu pings Windows Server — 0% packet loss
- Kali pings Ubuntu — 0% packet loss
- All VMs on 10.0.2.x range

## What I Learned
- How hypervisors work and why VirtualBox is used in cybersecurity labs
- How NAT Networks allow VMs to communicate internally
- Basic Linux terminal commands (ip addr, ping)
- How Windows Server firewall blocks ICMP by default and how to enable it

## Challenges
- Accidentally installed Windows Server directly on my physical PC 
  instead of in a VM — had to reinstall Windows 11 from a USB boot drive
- Learned the difference between installing an OS natively vs inside a VM
