## Replication & Failback Scenarios

The full DR lifecycle is managed through SRM and consists of six sequential operations:

<img width="1956" height="804" alt="SRM" src="https://github.com/user-attachments/assets/8af8ba8d-7e60-462f-a405-208982f21bfa" />

---
### Phase 1: Test Recovery Plan (Non-Destructive)

<img width="1839" height="545" alt="image" src="https://github.com/user-attachments/assets/fdecc5b5-7488-4a22-88a2-bde6805800c4" />
<img width="1837" height="480" alt="image" src="https://github.com/user-attachments/assets/35bc11b8-c4e2-40b9-815f-7acfdd4f5535" />
Validates the DR plan without disrupting production. Creates a writable snapshot of replicated storage and powers on VMs in an isolated test network.


Plan:      Failover-Test
Operation: Test Recovery Plan
Protected: PROD-00 (HK)
Recovery:  DRS-01  (SG)

```
Steps:

1.	Synchronized storage
    Ensure that the storage at the protected and recovery sites is synchronized.
2.	Restore recovery site's host from standby.
    Bring the recovery site's host out of standby mode.
3.	Suspend non-critical VMs at the recovery site.
      a.	Identify non-critical VMs at the recovery site.
      b.	Suspend these VMs to free up resources for the recovery process.
4.	Create writable storage snapshot.
      a.	Create a writable storage snapshot for the protection group VR.
      b.	Create a writable storage snapshot for the VM hklnx-app00.core.biz
      c.	Configure storage settings for the VM hklnx-app00.core.biz
5.	Configure test networks.
    Configure test networks for the VM hklnx-app00.core.biz.
6.	Power on priority 3 VMs.
      a.	 Power on the VM hklnx-app00.core.biz
      b.	Power on the VM hklnx-app00.core.biz.
      c.	Wait for VMware Tools to be ready.
7.	Finish.
    The recovery plan test process is complete.

```

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/586b6f78-38c5-4efe-a1b4-8471e9125dc7" />

---

### Phase 2: Cleanup

<img width="1834" height="522" alt="image" src="https://github.com/user-attachments/assets/c8a28ab5-5e2d-4fe9-84b4-3d1db42cb933" />

Rolls back all test state — powers off test VMs, discards snapshot, resets storage, resumes suspended VMs.

<img width="1840" height="197" alt="image" src="https://github.com/user-attachments/assets/3d1efc4b-9a80-407a-8e63-23bab8f1d520" />

```
Below is the procedure of clean up after testing the recovery plan:
1.	Restore recovery site hosts from standby.
2.	Power off test VMs at recovery site.
    a.	hklnx-app00.core.biz
    b.	Power off
    c.	Reset storage
3.	Resume non-critical VMs at recovery site
4.	Discard test data and reset storageProtection group VR

```

---

### Phase 3: Recovery I — Planned Migration (HK → SG)

<img width="433" height="573" alt="image" src="https://github.com/user-attachments/assets/3f4c2db5-b4ea-4e0f-b53d-14bd6dd47e1c" />

Migrates the protected VM from HK production site to SG DR site in a controlled manner. Both sites must be operational.

<img width="1837" height="523" alt="image" src="https://github.com/user-attachments/assets/21d2c7b2-8a04-43ce-a574-ba9640c52029" />
<img width="1529" height="786" alt="image" src="https://github.com/user-attachments/assets/5fa35d5c-735f-460c-9ec0-4b7ba6882ee2" />

Based on the figure above after conducting test recovery plan and clean up process, the recovery plan can be tested using planned migration recovery type. The purpose of this phase is to migrate the VM from protected site or HK DC to the Recovery site or SG DC.

```
The procedure of recovery phase I:
1.	Pre-synchronize storage in the Protection Group VR
2.	Shut down VM hklnx-app00.core.biz at protected site power off 
3.	Restore the recovery site hosts from standby
4.	Restore the protected site hosts from standby
5.	Prepare protected site VMs hklnx-app00.core.biz for migration and change storage to read-only and then synchronize storage in the Protection Group VR
6.	Change recovery site storage to writable in protection Group VR for hklnx-app00.core.biz and then configure storage
7.	Power on priority VMs hklnx-app00.core.biz and wait for VMware tools
```


---

### Phase 4: Reprotect I — Reverse Replication Direction

<img width="416" height="542" alt="image" src="https://github.com/user-attachments/assets/11573c47-5ca5-458a-8f47-049a5a3e571d" />

After Recovery I, SG becomes the new protected site and HK becomes the recovery site. Replication direction reverses (SG → HK).

<img width="1838" height="523" alt="image" src="https://github.com/user-attachments/assets/40a11fed-fe5a-4b03-bcc7-78782e68ee6e" />
<img width="1836" height="325" alt="image" src="https://github.com/user-attachments/assets/6ceaa86d-2d34-4e4e-8f4f-3606610992cc" />

After recovery phase I succeeded, the next thing to do is to re-protect the VM that has been moved to the recovery site in order to make the current recovery site becomes a protected site. By this process also make the direction of the replication is reversed.
```
Below is the procedure of reprotect phase I:
1.	Restore protected site hosts from standby
2.	Configure storage to reverse direction 
3.	Configure VR replication to replicate VM hklnx-app00.core.biz
4.	Configure protection to reverse direction in protection Group VR and then configure protection for VM hklnx-app00.core.biz
5.	Clean up storage in the protection group VR and then synchronize storage in protection Group VR
```


---

### Phase 5: Recovery II — Planned Migration (SG → HK)

<img width="422" height="475" alt="image" src="https://github.com/user-attachments/assets/804b253d-90c1-417b-88c4-a0278705a190" />

Migrates the VM back to HK. Protected site is now DRS-01 (SG), recovery site is PROD-00 (HK).

<img width="1836" height="521" alt="image" src="https://github.com/user-attachments/assets/ab1a5a5f-1af6-400b-800f-8968e09a345e" />
<img width="1531" height="785" alt="image" src="https://github.com/user-attachments/assets/ebb68e99-f02d-4493-8485-cfbf7a472bb2" />

After finishing re-protect phase I, in order to bring back the replicated and the direction of replication process to the HK datacenter, recovery phase II must be conducted using planned migration recovery type. In the Plan Summary below it can be noticed that Protected Site is DRS-01 which is located in the SG datacenter, whereas the recovery site is in the HK datacenter.

```
Below is the procedure of recovery phase II:
1.	Pre-synchronize storage and Check if the Protection Group VR is existed or not
2.	Shut down VMs at protected site
3.	Shut down VM hklnx-app00.core.biz
4.	Resume VMs suspended by previous recovery
5.	Restore recovery site hosts from standby
6.	Restore protected site hosts from standby
7.	Prepare protected site VMs for migration
8.	Check again if the Protection Group VR is existed or not
9.	Prepare VM hklnx-app00.core.biz for migration, and change the storage situation to read only
10.	 Synchronize storage and check again the Protection Group VR
11.	Change recovery site storage to writable in the Protection Group VR
12.	Configure the storage for VM hklnx-app00.core.biz
13.	Power on priority VM hklnx-app00.core.biz and wait for VMware Tools work

```


---

### Phase 6: Reprotect II — Restore Original Configuration

<img width="409" height="486" alt="image" src="https://github.com/user-attachments/assets/32f11df6-f34e-44e3-8487-1dff8391f9c8" />

Returns the system to the original state: HK is protected, SG is recovery, replication direction is HK → SG.
<img width="1839" height="523" alt="image" src="https://github.com/user-attachments/assets/11d9cd33-f72f-4684-901a-d44e0d761be4" />
<img width="1839" height="325" alt="image" src="https://github.com/user-attachments/assets/afe77cec-9f2c-4282-a22e-83460b8a188d" />
After finishing the recovery phase II, the next thing to do is re-protect again the current VM configuration in order to restored the direction of replication and configuration of protection and recovery sites to pre-migration condition

```
Below is the procedure of reprotect phase II:
1.	Restore protected site hosts from standby.
2.	Configure storage to reverse direction, also configure VR replication to enable replication for VM hklnx-app00.core.biz.
3.	Configure protection to reverse direction and check in the Protection Group and then configure protection for VM hklnx-app00.core.biz.
4.	Check again if the protection group is available or not, if available clean up the storage for the VM and then synchronize the storage between VM and protection group.

```



---
