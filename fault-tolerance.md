## Replication & Failback Scenarios

The full DR lifecycle is managed through SRM and consists of six sequential operations:

```
┌─────────┐    ┌─────────┐    ┌────────────┐    ┌─────────────┐    ┌────────────┐    ┌─────────────┐
│  Test   │───►│ Cleanup │───►│ Recovery I │───►│ Reprotect I │───►│Recovery II │───►│Reprotect II │
│         │    │         │    │ HK → SG    │    │ SG becomes  │    │ SG → HK    │    │ HK becomes  │
│(non-    │    │         │    │(planned    │    │ protected   │    │(planned    │    │ protected   │
│disrupt.)│    │         │    │ migration) │    │ site        │    │ migration) │    │ site again  │
└─────────┘    └─────────┘    └────────────┘    └─────────────┘    └────────────┘    └─────────────┘
```

### Phase 1: Test Recovery Plan (Non-Destructive)

Validates the DR plan without disrupting production. Creates a writable snapshot of replicated storage and powers on VMs in an isolated test network.

```
Plan:      Failover-Test
Operation: Test Recovery Plan
Protected: PROD-00 (HK)
Recovery:  DRS-01  (SG)
```

Steps:
1. Synchronize storage between sites
2. Restore recovery site hosts from standby
3. Suspend non-critical VMs at recovery site
4. Create writable storage snapshot for Protection Group VR
5. Configure test networks for `hklnx-app00.core.biz`
6. Power on priority 3 VMs at recovery site
7. Wait for VMware Tools to report ready

### Phase 2: Cleanup

Rolls back all test state — powers off test VMs, discards snapshot, resets storage, resumes suspended VMs.

### Phase 3: Recovery I — Planned Migration (HK → SG)

Migrates the protected VM from HK production site to SG DR site in a controlled manner. Both sites must be operational.

```
Algorithm:
  if ProtectionGroupVR available:
    preSynchronizeStorage()
    shutDownVM("hklnx-app00.core.biz")   // at protected site
    restoreRecoverySiteHostsFromStandby()
    prepareProtectedSiteVMsForMigration("hklnx-app00.core.biz")
    changeStorageToReadOnly("datastore2")
    synchronizeStorage()
    changeRecoverySiteStorageToWritable("hklnx-app00.core.biz")
    configureStorage("datastore2")
    powerOnPriorityVMs("hklnx-app00.core.biz")
```

### Phase 4: Reprotect I — Reverse Replication Direction

After Recovery I, SG becomes the new protected site and HK becomes the recovery site. Replication direction reverses (SG → HK).

```
Algorithm:
  RestoreProtectedSiteHostsFromStandby()
  ConfigureStorageToReverseDirection()
  ConfigureVRReplication("hklnx-app00.core.biz")
  ConfigureProtectionToReverseDirection()
  ConfigureVMsProtection("hklnx-app00.core.biz")
  CleanUpStorage()
  SynchronizeStorage()
```

### Phase 5: Recovery II — Planned Migration (SG → HK)

Migrates the VM back to HK. Protected site is now DRS-01 (SG), recovery site is PROD-00 (HK).

```
Algorithm:
  preSynchronizeStorage()
  restoreRecoverySiteHostFromStandby()
  restoreProtectedSiteHostFromStandby()
  if ProtectionGroupVR exists:
    prepareVM("hklnx-app00.core.biz")
    changeStorageToReadOnly()
    synchronizeStorage()
    changeRecoverySiteStorageToWritable("hklnx-app00.core.biz")
    configureStorage()
  powerOnVMs("hklnx-app00.core.biz")
```

### Phase 6: Reprotect II — Restore Original Configuration

Returns the system to the original state: HK is protected, SG is recovery, replication direction is HK → SG.

```
Algorithm:
  restoreProtectedSiteHostsFromStandby()
  configureStorageToReverseDirection()
  if ProtectionGroupVR exists:
    configureVRReplicationForVM("hklnx-app00.core.biz")
    configureProtectionForVM("hklnx-app00.core.biz")
    cleanUpStorageForProtectionGroup()
    synchronizeStorageForProtectionGroup()
```

