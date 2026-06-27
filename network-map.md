# Network & IP Address Reference

Quick reference for all IP addresses, FQDNs, and VMkernel roles in this environment.
<img width="2610" height="1917" alt="Untitled Diagram-Network" src="https://github.com/user-attachments/assets/323e01e6-24d5-46a1-82c2-c6f941cef128" />

### HK Datacenter — Production Site (`192.168.1.x`)

| Role | Type | VMkernel | FQDN | IP Address | Subnet | DNS |
|------|------|---------|------|-----------|--------|-----|
| System Server | ESXi | Management | `hkesx-prd00.core.biz` | 192.168.1.250 | 255.255.255.0 | 192.168.1.100 |
| vCenter Server | VM | Management | `hkvct-prd00.core.biz` | 192.168.1.251 | 255.255.255.0 | 192.168.1.100 |
| vSphere Replication | VM | vMotion / vSAN / Replication | `hkvrp-app00.core.biz` | 192.168.1.120 | 255.255.255.0 | 192.168.1.100 |
| Site Recovery Manager | VM | vMotion / vSAN / Replication | `hksrm-app00.core.biz` | 192.168.1.121 | 255.255.255.0 | 192.168.1.100 |
| App Server I | ESXi | Management | `hkesx-app00.core.biz` | 192.168.1.50 | 255.255.255.0 | 192.168.1.100 |
| Linux Appliance (FT Primary) | VM | Management | `hklnx-app00.core.biz` | 192.168.1.55 | 255.255.255.0 | 192.168.1.100 |
| App Server II | ESXi | Management | `hkesx-app01.core.biz` | 192.168.1.51 | 255.255.255.0 | 192.168.1.100 |
| Linux Appliance (FT Secondary) | VM | Management | `hkftx-app00.core.biz` | 192.168.1.58 | 255.255.255.0 | 192.168.1.100 |

### SG Datacenter — DR Site (`192.168.1.x`)

| Role | Type | VMkernel | FQDN | IP Address | Subnet | DNS |
|------|------|---------|------|-----------|--------|-----|
| System Server | ESXi | Management | `sgesx-dr01.core.biz` | 192.168.1.70 | 255.255.255.0 | 192.168.1.100 |
| vCenter Server | VM | Management | `sgvct-dr01.core.biz` | 192.168.1.71 | 255.255.255.0 | 192.168.1.100 |
| vSphere Replication | VM | vMotion / vSAN / Replication | `sgvrp-app01.core.biz` | 192.168.1.130 | 255.255.255.0 | 192.168.1.100 |
| Site Recovery Manager | VM | vMotion / vSAN / Replication | `sgsrm-app01.core.biz` | 192.168.1.131 | 255.255.255.0 | 192.168.1.100 |

### Domain Controller

| Role | Type | FQDN | IP Address | Subnet | DNS |
|------|------|------|-----------|--------|-----|
| Domain Controller | Standalone VM | `hkwinprd00.core.biz` | 192.168.1.100 | 255.255.255.0 | 192.168.1.100 |

> **Note:** The domain controller is intentionally installed as a standalone VM (not on the ESXi cluster) to prevent DNS failure during ESXi host failures. SRM's replication relies heavily on FQDN/DNS resolution for all inter-site communication.

---
## VMkernel Port Group Requirements

| VMkernel Role | Required On | Purpose |
|--------------|-------------|---------|
| Management | All ESXi hosts | vCenter communication, SSH |
| vMotion | App ESXi hosts | Live VM migration between hosts |
| vSAN | All 3 cluster ESXi hosts | Shared storage traffic |
| FT Logging | App ESXi hosts | Fault Tolerance real-time sync stream |
| Replication | System ESXi hosts | vSphere Replication traffic |

## DNS Records Required

All forward (A) and reverse (PTR) records must exist on `hkwinprd00.core.biz` for SRM to function correctly.

Domain: `core.biz`

```
hkwinprd00   A  192.168.1.100
hkesx-prd00  A  192.168.1.250
hkvct-prd00  A  192.168.1.251
hkvrp-app00  A  192.168.1.120
hksrm-app00  A  192.168.1.121
hkesx-app00  A  192.168.1.50
hklnx-app00  A  192.168.1.55
hkesx-app01  A  192.168.1.51
hkftx-app00  A  192.168.1.58
sgesx-dr01   A  192.168.1.70
sgvct-dr01   A  192.168.1.71
sgvrp-app01  A  192.168.1.130
sgsrm-app01  A  192.168.1.131
sgesxdr-app01 A  192.168.1.60
```
