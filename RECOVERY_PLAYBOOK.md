# SRM Recovery Playbook

Step-by-step operational runbook for executing and validating the full DR lifecycle using VMware Site Recovery Manager (SRM) in this dual-datacenter environment.

---

## Pre-Flight Checklist

Before running any recovery operation, verify the following:

- [ ] vSphere Replication is active and synchronized on both sites
- [ ] SRM pairing between `hksrm-app00` and `sgsrm-app01` is established
- [ ] DNS/FQDN resolution working on domain controller (`hkwinprd00.core.biz`)
- [ ] Protection Group VR is healthy in SRM console
- [ ] Recovery plan `Failover-Test` shows **Ready** status
- [ ] Both vCenter servers (`hkvct-prd00` and `sgvct-dr01`) are reachable
- [ ] Datastore5 (Replication) and Datastore6 (DR) are available on both sites

---

## Step 1: Test Recovery Plan

**Purpose:** Validate the DR plan without impacting production. Non-destructive.

**SRM Plan Summary:**

| Field | Value |
|-------|-------|
| Name | Failover-Test |
| Operation | Test Recovery Plan |
| Protected Site | PROD-00 (HK DC) |
| Recovery Site | DRS-01 (SG DC) |

**Procedure:**

1. Log in to vSphere Client as administrator
2. Navigate to **Site Recovery** → **Recovery Plans**
3. Select `Failover-Test` → click **Test**
4. Monitor progress — SRM will:
   - Synchronize storage (Protection Group VR)
   - Restore recovery site hosts from standby
   - Suspend non-critical VMs at SG site
   - Create writable storage snapshot for `hklnx-app00.core.biz`
   - Configure isolated test networks
   - Power on `hklnx-app00.core.biz` (priority 3)
   - Wait for VMware Tools heartbeat
5. Verify the VM is running at the SG site without affecting HK production

**Expected duration:** ~1 minute 10 seconds

---

## Step 2: Cleanup

**Purpose:** Roll back all test state. Must run before any real recovery.

**SRM Plan Summary:**

| Field | Value |
|-------|-------|
| Name | Failover-Test |
| Operation | Test Recovery Plan Clean-up |
| Protected Site | PROD-00 (HK DC) |
| Recovery Site | DRS-01 (SG DC) |

**Procedure:**

1. In SRM Recovery Plans, select `Failover-Test` → click **Cleanup**
2. SRM will:
   - Restore recovery site hosts from standby
   - Power off test VMs at SG site (`hklnx-app00.core.biz`)
   - Reset storage snapshot for Protection Group VR
   - Resume any suspended non-critical VMs at SG site
   - Discard all test data

**Expected duration:** ~2–6 seconds

---

## Step 3: Recovery I — Planned Migration (HK → SG)

**Purpose:** Migrate protected VM from production HK to DR SG. Controlled, zero data loss.

**SRM Plan Summary:**

| Field | Value |
|-------|-------|
| Name | Failover-Test |
| Operation | Recovery |
| Recovery Type | Planned Migration |
| Protected Site | PROD-00 (HK DC) |
| Recovery Site | DRS-01 (SG DC) |

**Procedure:**

1. In SRM Recovery Plans → `Failover-Test` → **Run**
2. Select **Planned Migration**
3. SRM executes:
   - Pre-synchronize storage for Protection Group VR
   - Gracefully shut down `hklnx-app00.core.biz` at HK (clean shutdown, not force-off)
   - Restore both sites' hosts from standby
   - Prepare `hklnx-app00.core.biz` for migration → set `datastore2` to read-only → synchronize storage
   - Set SG recovery storage to writable → configure `datastore2` at SG
   - Power on `hklnx-app00.core.biz` at SG site
   - Wait for VMware Tools to report ready

4. Verify VM is running at SG, accessible on expected IP

**Expected duration:** ~1 minute 25 seconds

---

## Step 4: Reprotect I — Reverse Replication to SG

**Purpose:** Make SG the new protected site. Reverse replication direction from SG → HK.

**SRM Plan Summary:**

| Field | Value |
|-------|-------|
| Name | Failover-Test |
| Operation | Reprotect |
| Reprotect Type | Ignore errors during reprotect |
| Protected Site | PROD-00 (HK DC) |
| Recovery Site | DRS-01 (SG DC) |

**Procedure:**

1. After Recovery I completes → click **Reprotect**
2. SRM executes:
   - Restore protected site (HK) hosts from standby
   - Reconfigure storage in Protection Group VR to reverse direction
   - Configure vSphere Replication for `hklnx-app00.core.biz` (now replicating SG → HK)
   - Configure protection group and VM protection in reverse direction
   - Clean up and synchronize storage in Protection Group VR

3. Confirm Protection Group VR shows SG as protected site, HK as recovery

**Expected duration:** ~6–30 seconds

---

## Step 5: Recovery II — Planned Migration (SG → HK)

**Purpose:** Migrate VM back to its original HK home. Restores normal operational state.

**SRM Plan Summary:**

| Field | Value |
|-------|-------|
| Name | Failover-Test |
| Operation | Recovery |
| Recovery Type | Planned Migration |
| Protected Site | DRS-01 (SG DC) |
| Recovery Site | PROD-00 (HK DC) |

**Note:** Protected and Recovery sites are now swapped compared to Recovery I.

**Procedure:**

1. In SRM Recovery Plans → `Failover-Test` → **Run**
2. SRM executes:
   - Pre-synchronize storage → verify Protection Group VR exists
   - Shut down `hklnx-app00.core.biz` at SG
   - Resume any VMs suspended by previous recovery
   - Restore both sites from standby
   - Prepare `hklnx-app00.core.biz` for migration → change storage to read-only → synchronize
   - Change HK recovery storage to writable → configure storage
   - Power on `hklnx-app00.core.biz` at HK site
   - Wait for VMware Tools

**Expected duration:** ~43 seconds – 1 minute 28 seconds

---

## Step 6: Reprotect II — Restore Original Configuration

**Purpose:** Return the system to its initial state. HK is protected, SG is recovery, replication is HK → SG.

**SRM Plan Summary:**

| Field | Value |
|-------|-------|
| Name | Failover-Test |
| Operation | Reprotect |
| Reprotect Type | None |
| Protected Site | DRS-01 (SG DC) |
| Recovery Site | PROD-00 (HK DC) |

**Procedure:**

1. After Recovery II → click **Reprotect**
2. SRM executes:
   - Restore protected site (SG) hosts from standby
   - Configure storage in Protection Group VR back to original direction (HK → SG)
   - If Protection Group VR exists:
     - Configure vSphere Replication for `hklnx-app00.core.biz` (HK → SG)
     - Configure VM protection at HK
     - Clean up Protection Group VR storage
     - Synchronize Protection Group VR storage
3. Confirm HK is back as protected site, SG as recovery

**Expected duration:** ~24–27 seconds

---

## Post-Recovery Verification

After completing all 6 steps, verify:

- [ ] `hklnx-app00.core.biz` is running at HK site (192.168.1.55)
- [ ] vSphere Replication shows HK → SG direction in SRM console
- [ ] Protection Group VR status is **OK** (not Warning or Error)
- [ ] DNS resolution for all FQDNs works correctly
- [ ] vSAN datastore is accessible from all 3 ESXi hosts
- [ ] Recovery plan `Failover-Test` shows **Ready** for next execution

---

## Common Issues

| Issue | Likely Cause | Resolution |
|-------|-------------|-----------|
| Replication lag / Warning | Network congestion between sites | Check VMkernel replication traffic; increase bandwidth or reduce RPO interval |
| SRM cannot find Protection Group | FQDN resolution failure | Verify DNS on `hkwinprd00.core.biz`; check A and PTR records for all VMs |
| Storage synchronization fails | Datastore name mismatch between sites | Ensure `datastore5` and `datastore6` names match exactly on both `hkesx-prd00` and `sgesx-dr01` |
| VM fails to power on at recovery site | IP conflict or network misconfiguration | Check recovery site network mapping in SRM; verify `192.168.1.x` range is available |
| Reprotect fails with timeout | VMware Tools not running in VM | Install/restart VMware Tools inside `hklnx-app00.core.biz` |
| FT VM not starting secondary | No vSAN connectivity | Verify vSAN VMkernel adapters on both app servers; check vSAN cluster health |
