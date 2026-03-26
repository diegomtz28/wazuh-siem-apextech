# Build Notes — What Actually Happened

> The unfiltered version. Every hiccup, every fix, every moment of "why is this not working."

---

## Hiccup 1 — Couldn't Get Into Proxmox to Start

Changed ISPs. New Deco mesh router. Proxmox had a hardcoded static IP from the old network that no longer existed — couldn't hit the web UI at all.

Had to plug a monitor directly into the HP Z2450, log into the bare metal console, and edit `/etc/network/interfaces` by hand to update the IP to the new 192.168.68.0/22 subnet.

First thought the ISP was assigning public IPs directly to devices because the Deco app was showing 71.83.x.x. Turned out that was just the WAN IP displayed in the router app. Home LAN was 192.168.68.x the whole time. Classic trap.

---

## Hiccup 2 — RAM Was at 98% Before We Even Started

16GB total on the host. Every VM running. Nothing left for a new build.

Shut down FS01 and MGMT01. While doing this, realized the jump box VM (MGMT01 — Windows Server used for RDP) was completely unnecessary. Proxmox has a built-in noVNC console that does the same thing with zero RAM cost. Killed it. Never looked back.

---

## Hiccup 3 — Ubuntu Netplan YAML Indentation Hell

Spent 30-40 minutes fighting indentation errors setting a static IP. The `routes:` block kept throwing:

```
did not find expected dash indicator via 10.10.1.1
```

YAML does not forgive tabs. Does not forgive inconsistent spacing. Does not give useful error messages.

Fix: scrapped the `routes:` block entirely and used `gateway4` — one line, no nesting, no drama. Deprecated in newer Ubuntu but works perfectly on 22.04 for a home lab.

---

## Hiccup 4 — Wazuh Installer Filled the Disk

Ubuntu LVM allocates roughly half the virtual disk by default. We gave it 50GB — Ubuntu took 24GB for LVM, left the rest unallocated. The Wazuh all-in-one installer (Manager + OpenSearch + Dashboard) ate every byte of that 24GB before it finished.

OpenSearch hit the flood-stage watermark (95% disk usage) and locked all indexes read-only. Dashboard threw errors on every page.

---

## Hiccup 5 — Disk Resize Took Four Attempts

This one hurt. The sequence that actually worked:

1. Resize the virtual disk in Proxmox (hardware level — disk goes from 50GB to 120GB)
2. `sudo growpart /dev/sda 3` — grow the partition to use the new space
3. `sudo pvresize /dev/sda3` — tell LVM about the bigger partition
4. `sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv` — extend the LV and resize the filesystem in one shot

What we tried before figuring out step 2: `lvextend` alone, `resize2fs` alone, rebooting. All of it said "nothing to do" or succeeded but `df -h` still showed 100% full. The missing piece was `growpart` — Proxmox growing the virtual disk doesn't automatically grow the partition inside it.

---

## Hiccup 6 — All Three Agents Registered as the Same Name

Generated one deploy command from the Wazuh dashboard and ran it on WS01, DC01, and HD01. All three tried to enroll with the agent name "Apextech."

Wazuh log on DC01:
```
ERROR: Duplicate agent name: Apextech (from manager)
ERROR: Unable to add agent (from manager)
```

Fix: fully uninstall the agent on DC01 and HD01 via Add/Remove Programs, then go back to the dashboard, generate a fresh deploy command with a unique name for each machine (DC01, HD01), and reinstall.

Each agent needs its own name. One deploy command does not work for multiple machines.

---

## What Got Us Through It

Read every error message. Looked at `lsblk` output when `df` didn't make sense. Checked logs when agents weren't showing up. Didn't assume — verified.

Total time: ~3.5 hours including a 30-minute Ubuntu install.
