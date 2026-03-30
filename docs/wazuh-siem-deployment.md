# Wazuh SIEM Deployment — apextech.local

> **Lab:** apextech.local | **Date:** 2026-03-25 | **Status:** Complete

## Overview

Deployed a Wazuh 4.9.2 all-in-one SIEM on a new Ubuntu Server 22.04 VM (SIEM01) in the apextech.local Proxmox lab. Wazuh provides centralized log collection, real-time alerting, vulnerability detection, and file integrity monitoring across the domain. All three agents — DC01, WS01, and HD01 — are active and reporting. This is the first security monitoring capability in the lab.

---

## Environment

| Component | Role | Details |
|-----------|------|---------|
| SIEM01 | Wazuh Manager + Indexer + Dashboard | Ubuntu Server 22.04 LTS — 10.10.1.140 — 4 vCPU / 4GB RAM / 118GB disk |
| DC01 | Domain Controller | Windows Server 2022 — 10.10.1.150 — Wazuh Agent 002 |
| WS01 | Domain Workstation | Windows 11 Pro — 10.10.1.100 — Wazuh Agent 001 |
| HD01 | Helpdesk / osTicket | Windows Server 2022 — 10.10.1.200 — Wazuh Agent 003 |
| pfSense | Firewall / Router | 10.10.1.1 — internal routing for 10.10.1.0/24 |
| Proxmox VE | Hypervisor | HP Z240 — i7-6700 — 16GB RAM |

---

## Steps Taken

1. **Freed RAM on Proxmox host**
   Host was at 98% RAM utilization. Shut down FS01 and MGMT01 to bring utilization to ~70%. Identified that MGMT01 (jump box) was unnecessary — Proxmox noVNC console provides direct VM access without a dedicated jump box.

2. **Created SIEM01 VM in Proxmox**
   Downloaded Ubuntu Server 22.04.5 LTS ISO to Proxmox local storage. Created VM:
   - VM ID: 110
   - CPU: 4 vCores
   - RAM: 4096 MB
   - Disk: 50GB on local-lvm (later expanded to 118GB)
   - Network: default bridge

3. **Installed Ubuntu Server 22.04**
   Minimal server install via Proxmox noVNC console. OpenSSH enabled during install. No GUI. Hostname: `siem01`. User: `diegomtz`.

4. **Configured static IP via netplan**
   Edited `/etc/netplan/50-cloud-init.yaml`:

   ```yaml
   network:
     version: 2
     ethernets:
       ens18:
         dhcp4: false
         addresses:
           - 10.10.1.140/24
         gateway4: 10.10.1.1
         nameservers:
           addresses:
             - 10.10.1.150
             - 8.8.8.8
   ```

   ```bash
   sudo netplan apply
   ```

5. **Expanded LVM volume to use full disk**
   Ubuntu LVM only allocated 24GB of the 50GB virtual disk by default. Wazuh installer filled it completely. Expanded Proxmox virtual disk to 120GB, then:

   ```bash
   sudo growpart /dev/sda 3
   sudo pvresize /dev/sda3
   sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
   ```

   Final usable disk: ~118GB.

6. **Installed Wazuh all-in-one**

   ```bash
   curl -o wazuh-install.sh https://packages.wazuh.com/4.9/wazuh-install.sh
   sudo bash wazuh-install.sh -a
   ```

   Install completed in ~15 minutes. Admin credentials printed at end of install summary — save these immediately.

7. **Deployed Wazuh agents on Windows endpoints**
   For each machine: Wazuh Dashboard → Server Management → Agents → Deploy new agent → Windows → set unique agent name → copy PowerShell command → run on target machine as admin → start WazuhSvc.

   | Agent ID | Name | Machine | IP |
   |----------|------|---------|-----|
   | 001 | Apextech | WS01 | 10.10.1.100 |
   | 002 | DC01 | DC01 | 10.10.1.150 |
   | 003 | HD01 | HD01 | 10.10.1.200 |

---

## Key Decisions

- **All-in-one over distributed:** 3-4 agents doesn't justify separate VMs for each Wazuh component. All-in-one is standard for small environments.
- **gateway4 over routes block:** Avoids multi-level YAML indentation in netplan. Functionally identical for a single default route.
- **growpart before pvresize:** Proxmox disk resize only grows the virtual hardware. Partition, PV, LV, and filesystem each require explicit extension in sequence.
- **Unique agent names:** Wazuh requires a unique name per enrolled agent. Running the same deploy script on multiple machines causes duplicate enrollment failures.

---

## Validation

- **Dashboard accessible:** Navigated to `https://10.10.1.140` — loaded successfully, admin login works
- **Agents active:** Wazuh Dashboard → Endpoints → `status=active` — 3 agents showing, all v4.9.2
- **Port 1514 open:** `Test-NetConnection -ComputerName 10.10.1.140 -Port 1514` from DC01 → `TcpTestSucceeded: True`

---

## Next Steps

- Start FS01 back up and deploy a 4th agent
- Simulate attack events (failed logins, account lockouts) and validate alerts fire
- Enable File Integrity Monitoring on FS01 file shares
- Consider RAM upgrade for host (16GB → 32GB DDR4) to run full lab without shutting VMs down
