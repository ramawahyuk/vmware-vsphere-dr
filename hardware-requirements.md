## Hardware Requirements for HK and SG datacenter

### HK Datacenter — Production Site

#### Domain Controller

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | AMD Threadripper — 2 core CPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 4 GB |
| 4 | Storage | 100 GB (Windows OS) |
| — | FQDN | `hkwinprd00.core.biz` |

#### ESXi System Server HK (`hkesx-prd00`)

Hosts: vCenter Server VCSA, vSphere Replication appliance, SRM appliance

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | AMD Threadripper — 16 core CPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 64 GB |
| 4 | Storage | 150 GB ESXi OS + 150 GB VCSA + 150 GB vSphere Replication + 100 GB SRM + 200 GB Replication + 200 GB DR |

#### ESXi App Server I HK (`hkesx-app00`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | AMD Threadripper — 16 core CPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 32 GB |
| 4 | Storage | 100 GB ESXi OS + 150 GB Linux OS |

#### ESXi App Server II HK (`hkesx-app01`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | AMD Threadripper — 16 core CPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 32 GB |
| 4 | Storage | 100 GB ESXi OS + 150 GB Linux OS |

#### vSphere Replication Appliance HK (`hkvrp-app00`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | 2 core vCPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 8 GB |
| 4 | Storage | 150 GB |

#### SRM Appliance HK (`hksrm-app00`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | 2 core vCPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 8 GB |
| 4 | Storage | 100 GB |

#### vCenter Server Appliance HK (`hkvct-prd00`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | 8 core vCPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 30 GB |
| 4 | Storage | 150 GB |

---

### SG Datacenter — Disaster Recovery Site

#### ESXi System Server SG (`sgesx-dr01`)

Hosts: vCenter Server VCSA, vSphere Replication appliance, SRM appliance

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | AMD Threadripper — 16 core CPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 100 GB |
| 4 | Storage | 100 GB ESXi OS + 200 GB VCSA + 150 GB vSphere Replication + 200 GB SRM + 200 GB Replication + 200 GB DR |

#### ESXi App Server SG (`sgesxdr-app01`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | AMD Threadripper — 16 core CPUs |
| 2 | Network | 1 × Bridged NIC |
| 3 | Memory | 32 GB |
| 4 | Storage | 100 GB ESXi OS + 150 GB Linux OS |

#### vSphere Replication Appliance SG (`sgvrp-app01`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | 2 core vCPUs |
| 3 | Memory | 8 GB |
| 4 | Storage | 150 GB |

#### SRM Appliance SG (`sgsrm-app01`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | 2 core vCPUs |
| 3 | Memory | 8 GB |
| 4 | Storage | 100 GB |

#### vCenter Server Appliance SG (`sgvct-dr01`)

| # | Component | Specification |
|---|-----------|--------------|
| 1 | Processor | 8 core vCPUs |
| 3 | Memory | 30 GB |
| 4 | Storage | 150 GB |

---
## Storage Configuration

### HK Datacenter — Production Site

<img width="2261" height="1440" alt="Untitled Diagram-HK (1)" src="https://github.com/user-attachments/assets/75c8bcab-6ebd-4caa-a05b-93a0125d8874" />

| Host | Datastore | Size (GB) | Asset |
|------|-----------|----------|-------|
| `hkesx-prd00` (192.168.1.250) | datastore1 | 150 | ESXi OS |
| | datastore2 | 150 | VCSA OS |
| | datastore3 | 150 | vSphere Replication OS |
| | datastore4 | 100 | SRM OS |
| | datastore5 | 200 | Replication |
| | datastore6 | 200 | DR/vSAN |
| `hkesx-app00` (192.168.1.50) | datastore1 | 100 | ESXi OS |
| | datastore2 | 100 | App OS |
| | datastore3 | 140 | DR/vSAN |
| `hkesx-app01` (192.168.1.51) | datastore1 | 100 | ESXi OS |
| | datastore2 | 100 | App OS |


### SG Datacenter — DR Site

<img width="1443" height="1389" alt="Untitled Diagram-SG" src="https://github.com/user-attachments/assets/3c33789f-8ecc-4859-80b3-de4caf1edb73" />

| Host | Datastore | Size (GB) | Asset |
|------|-----------|----------|-------|
| `sgesx-dr01` (192.168.1.70) | datastore1 | 150 | ESXi OS |
| | datastore2 | 150 | VCSA OS |
| | datastore3 | 150 | vSphere Replication OS |
| | datastore4 | 100 | SRM OS |
| | datastore5 | 200 | Replication |
| | datastore6 | 200 | DR/vSAN |
| `sgesxdr-app01` (192.168.1.60) | datastore1 | 150 | ESXi OS |
| | datastore2 | 100 | Linux OS |

### Domain Controller

| Host | Datastore | Size (GB) | Asset |
|------|-----------|----------|-------|
| `hkwinprd00` | datastore1 | 100 | Windows OS |

### vSAN Shared Storage

Three ESXi hosts contribute dedicated disk capacity to a single vSAN datastore, accessible by all three hosts as shared storage — enabling Fault Tolerance for VMs.

| Host | Datastore | Size (GB) | Network |
|------|-----------|----------|---------|
| `hkvct-prd00` (192.168.1.251) | datastore7 | 140 | vSAN dedicated NIC / VMkernel vSAN traffic |
| `hkesx-app00` (192.168.1.50) | datastore3 | 140 | vSAN dedicated NIC / VMkernel vSAN traffic |
| `hkesx-app01` (192.168.1.51) | datastore3 | 140 | vSAN dedicated NIC / VMkernel vSAN traffic |
| **Total vSAN capacity** | | **420 GB** | |

> **Datastore parity is critical:** Production (`hkesx-prd00`) and DR (`sgesx-dr01`) primary hosts must have identical datastore naming and sizing. SRM synchronizes datastores between sites during replication and failback — any mismatch causes replication failures.

---
