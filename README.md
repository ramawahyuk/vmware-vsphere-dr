# 🏛️ VMware vSphere Clustered System with Disaster Recovery

> **Clustered system with disaster recovery capabilities to minimize downtime using virtual machines**  
> This is the documentation of my Academic research project which I design, implement, and validate a dual-datacenter VMware cluster with vSphere Replication, Site Recovery Manager (SRM), Fault Tolerance, and automated failback across two geographically separated sites.

[![VMware ESXi](https://img.shields.io/badge/VMware_ESXi-8.0-blue?logo=vmware)](https://www.vmware.com/products/esxi-and-esx.html)
[![vCenter](https://img.shields.io/badge/vCenter_Server-8.0-blue?logo=vmware)](https://www.vmware.com/)
[![SRM](https://img.shields.io/badge/SRM-8.7-orange?logo=vmware)](https://www.vmware.com/products/site-recovery-manager.html)
[![vSphere Replication](https://img.shields.io/badge/vSphere_Replication-8.7-green?logo=vmware)](https://www.vmware.com/)
---

## 📋 Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Network Configuration](https://github.com/ramawahyuk/vmware-vsphere-dr/blob/main/network-map.md)
- [Hardware Configuration](https://github.com/ramawahyuk/vmware-vsphere-dr/blob/main/hardware-requirements.md)
- [Hostname & FQDN Convention](#hostname--fqdn-convention)
- [Fault Tolerance Design](#fault-tolerance-design)
- [Replication & Failback Scenarios](https://github.com/ramawahyuk/vmware-vsphere-dr/blob/main/fault-tolerance.md)
- [Results](https://github.com/ramawahyuk/vmware-vsphere-dr/blob/main/results.md)
- [Key Concepts](#key-concepts)

---

## Overview

This project implements a clustered virtual machine environment across two simulated datacenters — **HK Datacenter (Production)** and **SG Datacenter (Disaster Recovery)** — with the following capabilities:

| Capability | Implementation |
|-----------|---------------|
| High Availability | VMware vSphere HA across ESXi cluster |
| Fault Tolerance | vSphere FT with real-time VM mirroring (zero downtime) |
| Disaster Recovery | VMware SRM 8.7 + vSphere Replication 8.7 |
| Shared Storage | VMware vSAN (420 GB across 3 ESXi hosts) |
| Replication RPO | As low as 5 minutes |
| Full Failback | Automated via SRM planned migration |
| Domain Management | Windows Server 2022 domain controller (DNS + AD) |

**End-to-end DR cycle validated:** Test → Cleanup → Recovery I → Reprotect I → Recovery II → Reprotect II — completed in under 4 minutes 15 seconds for two Linux VMs.

---

## System Architecture
<img width="3341" height="3251" alt="Untitled Diagram-Page-14" src="https://github.com/user-attachments/assets/503d4cd0-0b5a-43e7-8cbf-1f2c5dafba44" />

**Component roles:**

| Component | Role |
|-----------|------|
| vSphere Client | Administrator UI — manages both vCenter instances |
| vCenter Server (VCSA) | Centralized VM, resource, and cluster management per site |
| ESXi | Type-1 bare-metal hypervisor hosting all VMs |
| vSphere Replication | Hypervisor-level VM replication from HK → SG |
| Site Recovery Manager (SRM) | Orchestrates DR plans, failover, reprotect, and failback |
| vSAN | Software-defined shared storage across 3 ESXi hosts |
| Domain Controller | DNS resolution for FQDN lookups (critical for SRM replication) |

---

## Hostname & FQDN Convention

All nodes follow a structured FQDN naming scheme: `XX-ZZ-VVV-NN.domain.TLD`

| Segment | Meaning | Examples |
|---------|---------|---------|
| `XX` | Datacenter location | `hk` = Hong Kong, `sg` = Singapore |
| `ZZ` | OS / environment | `esx` = ESXi, `lnx` = Linux, `win` = Windows |
| `VVV` | Organizational unit | `prd` = production, `app` = application, `dr` = disaster recovery |
| `NN` | System ID | `00`, `01`, `02` |
| `domain` | Domain name | `core` |
| `TLD` | Top-level domain | `.biz` |

**Example:** `hkesx-prd00.core.biz`
- `hk` → Hong Kong
- `esx` → ESXi server
- `prd` → production
- `00` → first instance
- `core.biz` → domain

---
## Key Concepts

| Term | Definition |
|------|-----------|
| **RPO** (Recovery Point Objective) | Maximum acceptable data loss measured in time — how old can the recovered data be? |
| **RTO** (Recovery Time Objective) | Maximum acceptable time to restore service after a disaster |
| **Planned Migration** | Controlled failover where both sites are operational; zero data loss |
| **Disaster Recovery** | Failover when the protected site is unavailable; minimal data loss |
| **Reprotect** | After failover, reverses the replication direction so the new active site is protected |
| **Protection Group** | A named SRM group of VMs that fail over together under the same recovery plan |
| **vSAN** | VMware Virtual SAN — pools local ESXi host disks into shared storage |
| **FT Logging** | Network stream carrying CPU instruction state between primary and secondary FT VMs |
| **Hot Standby** | DR site is always running and continuously synchronized with production |
| **Warm Standby** | DR site runs but synchronization is periodic, not continuous |
| **Cold Standby** | DR site is powered off; longest activation time |

---
## VM Fault Tolerance Design

vSphere Fault Tolerance (FT) provides **zero-downtime protection** for individual VMs by maintaining a live secondary VM on a separate ESXi host that mirrors every CPU instruction of the primary in real time.

```
  ESXi Host Primary (hkesx-app00)      ESXi Host Secondary (hkesx-app01)
  ┌───────────────────────────┐         ┌──────────────────────────────┐
  │  Primary VM               │         │  Secondary VM                │
  │  hklnx-app00.core.biz    │◄────────►│  (shadow — always in sync)  │
  │  (active, serving traffic)│  FT Log │  (takes over instantly       │
  └───────────────────────────┘  Stream │   if primary host fails)     │
                                        └──────────────────────────────┘
```

**Requirements:**
- Both hosts must be connected to the vSAN datastore (shared storage is mandatory for FT)
- VMkernel adapters for **FT Logging** and **vMotion** must be configured on both hosts
- FT provides zero RPO and zero RTO — no shutdown, no data loss, no manual intervention

**Difference between FT and SRM Replication:**

| Feature | Fault Tolerance (FT) | SRM + vSphere Replication |
|---------|---------------------|--------------------------|
| Scope | Single host failure | Site-level disaster |
| RPO | Zero (real-time mirror) | ≥ 5 minutes |
| RTO | Zero (automatic) | Minutes (orchestrated) |
| Failback | Automatic | Manual (planned migration) |
| Use case | Critical VMs on same cluster | Cross-datacenter DR |

---

## Software Versions

| Component | Version |
|-----------|---------|
| VMware ESXi | 8.0 |
| VMware vCenter Server Appliance (VCSA) | 8.0 |
| VMware vSphere Replication | 8.7 |
| VMware Site Recovery Manager (SRM) | 8.7 |
| Windows Server (Domain Controller) | 2022 |
| Guest OS (Application VMs) | Linux |

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

## Academic Reference

> Kurniawan Rama Wahyu (GS66385). *Clustered System with Disaster Recovery Capabilities to Minimize Downtime Using Virtual Machines*. SKR-5302 Advanced Distributed Computing, 2023.
