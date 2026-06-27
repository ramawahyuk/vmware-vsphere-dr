## Results

### Recovery Test Run 1

| Operation | Result | Duration |
|-----------|--------|---------|
| Test | ✅ Success | 1 m 13 s |
| Cleanup | ✅ Success | 2 s |
| Recovery I (HK → SG) | ✅ Success | 1 m 26 s |
| Reprotect I | ✅ Success | 30 s |
| Recovery II (SG → HK) | ✅ Success | 1 m 28 s |
| Reprotect II | ✅ Success | 27 s |
| **Total** | | **5 m 15 s** |

### Recovery Test Run 2

| Operation | Result | Duration |
|-----------|--------|---------|
| Test | ✅ Success | 1 m 6 s |
| Cleanup | ✅ Success | 6 s |
| Recovery I (HK → SG) | ✅ Success | 1 m 23 s |
| Reprotect I | ✅ Success | 6 s |
| Recovery II (SG → HK) | ✅ Success | 43 s |
| Reprotect II | ✅ Success | 24 s |
| **Total** | | **3 m 8 s** |

**vSphere Replication active period:** June 12, 2023 – June 19, 2023

### Key Findings

- Full 6-step DR cycle (migration + failback) completed in **under 4 minutes 15 seconds** for two Linux VMs
- SRM dramatically simplifies the DR process — administrators define the plan once; execution is automated
- SRM with planned migration achieves **zero data loss** (pre-synchronization before shutdown)
- Fault Tolerance provides **zero-downtime** protection against individual host failures within the same cluster
- RPO as low as **5 minutes** is achievable with vSphere Replication
- DNS/FQDN resolution is a hard dependency — domain controller must remain available throughout DR operations

