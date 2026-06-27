# Network & IP Address Reference

Quick reference for all IP addresses, FQDNs, and VMkernel roles in this environment.

<img width="677" height="264" alt="image" src="https://github.com/user-attachments/assets/091e8807-82fa-43c8-9736-6ae8c71bd3aa" />
<img width="677" height="163" alt="image" src="https://github.com/user-attachments/assets/e6730b76-74cc-4783-a3fd-2d3743311513" />
<img width="679" height="85" alt="image" src="https://github.com/user-attachments/assets/495e1227-9e23-4adf-95d8-acbd3273807d" />


## IP Summary Table

| IP Address | FQDN | Role | Site |
|-----------|------|------|------|
| 192.168.1.100 | hkwinprd00.core.biz | Domain Controller (DNS + AD) | Shared |
| 192.168.1.250 | hkesx-prd00.core.biz | ESXi System Server | HK (Prod) |
| 192.168.1.251 | hkvct-prd00.core.biz | vCenter Server Appliance | HK (Prod) |
| 192.168.1.120 | hkvrp-app00.core.biz | vSphere Replication Appliance | HK (Prod) |
| 192.168.1.121 | hksrm-app00.core.biz | Site Recovery Manager Appliance | HK (Prod) |
| 192.168.1.50  | hkesx-app00.core.biz | ESXi App Server I | HK (Prod) |
| 192.168.1.55  | hklnx-app00.core.biz | Linux VM (FT Primary) | HK (Prod) |
| 192.168.1.51  | hkesx-app01.core.biz | ESXi App Server II | HK (Prod) |
| 192.168.1.58  | hkftx-app00.core.biz | Linux VM (FT Secondary) | HK (Prod) |
| 192.168.1.70  | sgesx-dr01.core.biz  | ESXi System Server | SG (DR) |
| 192.168.1.71  | sgvct-dr01.core.biz  | vCenter Server Appliance | SG (DR) |
| 192.168.1.130 | sgvrp-app01.core.biz | vSphere Replication Appliance | SG (DR) |
| 192.168.1.131 | sgsrm-app01.core.biz | Site Recovery Manager Appliance | SG (DR) |
| 192.168.1.60  | sgesxdr-app01.core.biz | ESXi App Server | SG (DR) |

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
