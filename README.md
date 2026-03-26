# Wazuh SIEM — apextech.local

> Wazuh 4.9.2 deployed on Ubuntu Server 22.04 in a Proxmox home lab, monitoring a Windows Active Directory environment across 3 endpoints.

---

## What This Is

A fully operational SIEM built from scratch in the apextech.local home lab. Wazuh runs as an all-in-one deployment — Manager, Indexer (OpenSearch), and Dashboard — on a dedicated Ubuntu Server VM. Three Windows endpoints are enrolled as agents and actively reporting.

This documents the full build: every step, every failure, and every fix.

---

## Lab Environment

| Component | Role | IP |
|-----------|------|----|
| SIEM01 | Wazuh Manager + Indexer + Dashboard | 10.10.1.140 |
| DC01 | Domain Controller (AD DS, DNS) | 10.10.1.150 |
| WS01 | Domain Workstation (Windows 11) | 10.10.1.100 |
| HD01 | Helpdesk Server (osTicket) | 10.10.1.200 |
| pfSense | Firewall / Router | 10.10.1.1 |

**Hypervisor:** Proxmox VE 9.0.3 — HP Z2450 — Intel i7-6700 — 15.41GB RAM
**Domain:** apextech.local
**Internal subnet:** 10.10.1.0/24

---

## Stack

| Component | Version |
|-----------|---------|
| Wazuh Manager | 4.9.2 |
| Wazuh Indexer (OpenSearch) | 4.9.2 |
| Wazuh Dashboard | 4.9.2 |
| OS | Ubuntu Server 22.04 LTS |
| Agents | Windows Server 2022, Windows 11 Pro |

---

## Active Agents

| Agent ID | Name | OS | IP |
|----------|------|----|----|
| 001 | Apextech | Windows 11 Pro | 10.10.1.100 |
| 002 | DC01 | Windows Server 2022 | 10.10.1.150 |
| 003 | HD01 | Windows Server 2022 | 10.10.1.200 |

---

## Documentation

| Doc | Description |
|-----|-------------|
| [Deployment Runbook](docs/wazuh-siem-deployment.md) | Full step-by-step build guide |
| [Build Notes — What Actually Happened](docs/build-notes.md) | Raw notes: every hiccup, every fix |

---

## Capabilities Enabled

- Real-time endpoint alerting across the domain
- Windows Event Log collection (Security, System, Application)
- Active Directory monitoring (DC01)
- File Integrity Monitoring
- Vulnerability Detection
- MITRE ATT&CK mapping
- Configuration Assessment

---

## Part of the ApexTech Home Lab

This SIEM is one component of a larger enterprise-simulated home lab.
Full lab documentation: [apex-tech-homelab](https://github.com/diegomtz28/apex-tech-homelab)
