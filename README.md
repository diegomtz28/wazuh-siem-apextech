# 🛡️ Wazuh SIEM — apextech.local

![Wazuh](https://img.shields.io/badge/Wazuh-4.9.2-blue?style=flat-square&logo=wazuh)
![Ubuntu](https://img.shields.io/badge/Ubuntu_Server-22.04_LTS-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![Proxmox](https://img.shields.io/badge/Proxmox-VE_9.0.3-E57000?style=flat-square&logo=proxmox&logoColor=white)
![Agents](https://img.shields.io/badge/Active_Agents-3-brightgreen?style=flat-square)
![Status](https://img.shields.io/badge/Status-Live-success?style=flat-square)

Wazuh all-in-one SIEM deployed from scratch in the apextech.local home lab. Monitors 3 Windows endpoints in an Active Directory environment — real-time alerting, vulnerability detection, FIM, and MITRE ATT&CK mapping. Built on a fresh Ubuntu Server VM inside Proxmox.

---

## Network Architecture

```
Internet
    │
    ▼
┌─────────────────────┐
│  TP-Link Deco Mesh  │  192.168.68.0/22
│  (Home Router)      │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│  Proxmox VE 9.0.3   │  HP Z2450 · i7-6700 · 15.41GB RAM
│  (Hypervisor)       │
└────────┬────────────┘
         │
         ▼  10.10.1.0/24 (internal lab network)
┌────────────────────────────────────────────────────────┐
│                    apextech.local                      │
│                                                        │
│  pfSense ──── DC01 ──── FS01                          │
│  10.10.1.1    10.10.1.150  10.10.1.110                │
│                   │                                    │
│              ┌────┴────┐                               │
│           WS01       HD01                             │
│        10.10.1.100  10.10.1.200                       │
│                                                        │
│  ┌─────────────────────────────────┐                  │
│  │  SIEM01 — Wazuh 4.9.2           │                  │
│  │  10.10.1.140                    │  ◄── YOU ARE HERE│
│  │  Manager · Indexer · Dashboard  │                  │
│  └─────────────────────────────────┘                  │
└────────────────────────────────────────────────────────┘
```

---

## Active Agents

| ID | Name | Machine | OS | IP | Status |
|----|------|---------|----|----|--------|
| 001 | Apextech | WS01 | Windows 11 Pro | 10.10.1.100 | 🟢 Active |
| 002 | DC01 | DC01 | Windows Server 2022 | 10.10.1.150 | 🟢 Active |
| 003 | HD01 | HD01 | Windows Server 2022 | 10.10.1.200 | 🟢 Active |

---

## What's Running

| Component | Details |
|-----------|---------|
| **Wazuh Manager** | Receives and processes agent data |
| **Wazuh Indexer** | OpenSearch — stores and indexes all events |
| **Wazuh Dashboard** | Web UI at `https://10.10.1.140` |
| **OS** | Ubuntu Server 22.04 LTS |
| **VM Specs** | 4 vCPU · 4GB RAM · 118GB disk |

---

## Capabilities

- **Real-time Alerting** — Windows Event Logs across all 3 endpoints
- **Active Directory Monitoring** — logons, group changes, policy events from DC01
- **Vulnerability Detection** — CVE scanning across enrolled agents
- **File Integrity Monitoring** — detects unauthorized file changes
- **MITRE ATT&CK Mapping** — alerts mapped to ATT&CK framework
- **Configuration Assessment** — CIS benchmark checks on Windows endpoints

---

## Docs

| File | What's in it |
|------|-------------|
| [Deployment Runbook](docs/wazuh-siem-deployment.md) | Step-by-step build — every command, every config |
| [Build Notes](docs/build-notes.md) | What actually went wrong and how it got fixed |

---

## The Honest Build Summary

Nothing went cleanly. Fought YAML indentation for 40 minutes. Ubuntu LVM ate the disk before Wazuh even finished installing. The disk resize required four different commands in the right order before it worked. Every agent tried to register with the same name.

Read every error. Fixed every one. That's IT.

---

## Part of apextech.local

This SIEM is one layer of a larger enterprise-simulated home lab.
Full lab: [apex-tech-homelab](https://github.com/diegomtz28/apex-tech-homelab)
