# Home Lab Build: Proxmox, Wazuh SIEM & Pi-hole DNS

**Date:** June 30, 2026
**Platform:** Home Lab (Proxmox, Ubuntu Server, Windows Server 2022, Kali, Raspberry Pi)
**Domain:** Home Lab Build

---

## What I Did
- Migrated Proxmox node (HP ProDesk 400 G6) from 192.168.100.x to the home network (10.0.0.50) by editing /etc/network/interfaces
- Built and installed a Kali Linux VM (VM 100, 10.0.0.71) on Proxmox
- Built and installed an Ubuntu Server VM (VM 101, 10.0.0.60) on Proxmox
- Built a Windows Server 2022 VM (VM 103, 10.0.0.70) on Proxmox
- Deployed the full Wazuh SIEM stack on Ubuntu Server using wazuh-install.sh, including wazuh-manager, wazuh-indexer, wazuh-dashboard, and Filebeat
- Set static IPs on all VMs via netplan (Ubuntu) and SConfig (Windows Server)
- Installed Pi-hole on a Raspberry Pi 3B+ at 10.0.0.100
- Changed the Pi's network config from dhcpcd to NetworkManager using nmcli
- Pointed personal-PC and lab VMs' DNS to Pi-hole (10.0.0.100)
- Set router DHCP range to 10.0.0.151-254 to reserve 10.0.0.50-150 for static lab devices
- Enabled RDP on Windows Server via SConfig

## Lab Network Map

| Device | IP |
|--------|-----|
| Proxmox Node | 10.0.0.50 |
| Ubuntu / Wazuh SIEM | 10.0.0.60 |
| Windows Server 2022 | 10.0.0.70 |
| Kali Linux | 10.0.0.71 |
| Pi-hole DNS | 10.0.0.100 |
| personal-PC | 10.0.0.243 |

---

## What I Learned Today
- Proxmox network config lives in /etc/network/interfaces: vmbr0 is the bridge interface VMs attach to
- Ubuntu Server uses netplan for network config: YAML indentation is strict, spaces not tabs
- Wazuh's full stack includes four components: indexer (search), manager (analysis engine), dashboard (web UI), Filebeat (log shipper)
- LVM volumes don't automatically use all allocated disk space: need lvextend + resize2fs to expand
- Debian 12+ uses NetworkManager by default, not dhcpcd: nmcli is the right tool
- Pi-hole acts as a DNS sinkhole: devices send DNS queries to it, it blocks known ad/malware domains before the request leaves the network
- Setting the DHCP pool start address on the router reserves lower IPs for static lab devices
- Router DHCP reservations only work within the DHCP pool range

**Big Picture:** This is the lab moving from single VirtualBox VMs to a real hypervisor with a working SIEM ingesting logs, which is the actual environment a Tier 1 analyst works in day to day.
