# Building a ZFS Mirror NAS on Proxmox with Samba Over Tailscale

**Date:** 2026-08-27
**Platform:** Proxmox VE (Debian 12 LXC)
**Domain:** Systems Administration / Storage / Networking

---

## Overview

Built a NAS directly on an existing Proxmox host (HP ProDesk 400 G6 SFF) instead of a dedicated TrueNAS VM, to avoid the RAM overhead of a second hypervisor layer and give Proxmox native access to the pool. Storage lives in a ZFS mirror on the host itself; a privileged LXC container handles file sharing over Samba, reachable through Tailscale instead of the local VLAN.

## Hardware

- ProDesk 400 G6 SFF, existing Proxmox host, 32GB RAM, single free PCIe x16 slot
- GLOTRENDS SA3026-C, 6-port PCIe SATA III card, ASM1166 chipset, non-RAID/AHCI passthrough
- 3x drives connected through the card: 1x Seagate ST2000DM008 2TB, 2x WD20EARS (Caviar Green) 2TB
- A fourth 1.5TB drive is currently non-functional and excluded from this build

## Drive Health Verification

- Ran a quick `smartctl -a` pass on all three drives before touching anything
- Seagate ST2000DM008 came back "PASSED" on the overall health check, but with 8 Current_Pending_Sector and 8 Offline_Uncorrectable, both nonzero. Also confirmed as an SMR drive against Seagate's own spec sheet, not the CMR type ZFS expects in a redundant vdev
- Both WD20EARS drives came back clean: 0 reallocated, 0 pending, 0 uncorrectable, despite ~98,900 power-on hours each (~11.3 years continuous)
- Ran extended (`long`) self-tests on all three concurrently, since a "PASSED" quick check only means a drive hasn't crossed a failure threshold, not that it's actually clean
- Seagate failed the extended test outright: `Completed: read failure`, stopped 40% through, with a logged bad LBA
- Both WD drives completed the full extended test without error
- Decision: exclude the Seagate from the pool entirely. SMR sits in the write path of every mirror/RAIDZ write, not just resilvers, and combined with a confirmed unreadable sector it's not trustworthy for any tier of this build, including scratch space

## ZFS Pool

- Built a 2-way mirror on the two WD20EARS drives, using stable `/dev/disk/by-id/` paths rather than `/dev/sdX`, since drive-letter assignment isn't guaranteed to survive a reboot
- Forced `ashift=12` explicitly rather than trusting the drive's reported sector size. Getting this wrong isn't fixable later without destroying and rebuilding the whole pool
- `zpool status` confirmed `ONLINE` at the pool, vdev, and both individual drive levels, zero errors
- Created a dataset (`naspool/share`) with `compression=lz4` and `atime=off`
- Usable capacity: ~1.81TB. A mirror trades raw capacity for redundancy, one drive's worth of space with a full duplicate on the other

## Network Reachability

- The Proxmox host sits on Lab VLAN 40 (10.0.40.0/24). The primary workstation is on Main VLAN 1 (10.0.0.0/24)
- Confirmed with a direct ping test that VLAN 1 to VLAN 40 isn't currently routed at the router
- Rather than open new router/firewall paths for a single service, used the existing Tailscale mesh already running across the lab. Tailscale is an overlay network, so any two devices on the same tailnet reach each other directly regardless of the underlying VLAN
- Installed Tailscale inside the LXC container itself, giving it its own tailnet identity and IP, independent of the host's own Tailscale connection

## LXC Container (File Sharing)

- Debian 12 LXC container, built privileged rather than unprivileged. This was a deliberate call: privileged containers don't remap UIDs, so container UID 1000 is the same UID 1000 on the host, which matters the moment a host path gets bind-mounted in
- Had to explicitly pass `/dev/net/tun` into the container (cgroup device allow rule plus a mount entry), since LXC doesn't expose it by default and Tailscale needs it
- Hit a stall where `timedatectl` hung and timed out on a D-Bus call. Root cause was systemd 252 inside the container needing the `nesting` and `keyctl` LXC features to manage its own services properly. Enabled both, timezone set cleanly afterward
- Bind-mounted the ZFS dataset into the container with `pct set --mp0`, confirmed with a test file that writes from inside the container landed on the actual host-side dataset, not a separate copy

## Samba

- Installed Samba, appended a `[share]` block to `smb.conf` pointing at the bind mount: guest access off, single valid user, explicit create/directory masks
- Created a Samba-only Linux user (no shell, no home directory) separate from the container's root login, set its SMB password with `smbpasswd`
- `testparm -s` confirmed the config parsed clean with no syntax issues
- First connection attempt from Windows succeeded (browseable, authenticated), but writing a file failed with a "Destination Folder Access Denied" error
- Root cause wasn't Samba config, it was the underlying Unix directory ownership: the bind-mounted directory was still owned `root:root` from creation, and the Samba user wasn't root. `create mask`/`directory mask` in `smb.conf` only govern permissions on new files once a write is already allowed by the parent directory; they don't override the parent's own permissions
- Fixed with `chown` on the mount point inside the container, which (since the container is privileged) changed ownership on the real host-side directory too

## Verification

- Connected from Windows to the container's Tailscale IP over SMB, authenticated as the Samba user
- Created a test file from Windows, confirmed it landed on the actual ZFS dataset by checking from the Proxmox host directly, not just inside the container
- File ownership showed as a raw UID rather than a name on the host side, expected since the host's `/etc/passwd` has no entry for that UID, but it matched the container's UID exactly. Confirmed the privileged-container decision was correct, no UID remapping in the way

---

**What I Learned Today**

- SMART's "PASSED" is a threshold check, not a clean bill of health. A drive can carry pending and uncorrectable sectors and still pass, because those attributes haven't crossed the failure threshold yet. An extended self-test surfaces what a quick check misses.
- SMR drives don't belong in redundant ZFS vdevs. The problem isn't limited to resilver time, it's every write to the pool, since mirror and RAIDZ writes touch every member of the vdev.
- Privileged vs. unprivileged LXC containers change UID mapping, and that decision only becomes visible the moment you bind-mount a host path in. It's the difference between a one-line `chown` fix and chasing a UID offset table.
- Samba's create/directory masks apply after the fact. If the parent directory's real Unix permissions don't allow the write, a correct `smb.conf` (confirmed with `testparm`) won't matter.
- Tailscale as a mesh overlay solves cross-VLAN reachability without touching router config, useful when segmentation is intentional and opening a new routing path isn't worth it for one service.

**Big Picture:** A working NAS came out of verifying every layer independently, drive health, pool topology, container privilege model, network path, and file permissions, instead of assuming any one of them was fine because the previous one worked.
