---
tags:
  - aks
---
## Choosing the right VM Image for high-traffic AKS clusters

Azure offers various VM types optimized for different workloads. For running small and medium-sized Azure Kubernetes Service (AKS) clusters with high traffic, the following VM types are most suitable

| VM Series | vCPU | Memory (GB) | OS Disk Size (Max) | VM IOPS (Max) | Ephemeral Disk IOPS (Max) |
| --------- | ---- | ----------- | ------------------ | ------------- | ------------------------- |
| **Ddsv5** | 2-96 | 8-384       | 24,576 GB          | 153,600       | Up to 450,000             |
| **Edsv5** | 2-96 | 16-672      | 2048 GB            | 32,000        | Up to 450,000             |
| **Fsv2**  | 2-72 | 4-144       | 2048 GB            | 20,000        | Up to 144,000             |
### 1. **Fsv2-series** – _Compute Optimized (Ephemeral OS Disk Supported)_

- **Best for**: CPU-intensive, bursty workloads (e.g., APIs, microservices).
- **vCPU**: 2–72
- **Memory**: 4–144 GiB
- **Ephemeral OS Disk**: ✅
- **Ephemeral Disk IOPS**: Up to 144,000 ([link](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/compute-optimized/fsv2-series?tabs=sizebasic))
- **Notes**: High-frequency CPUs and local SSDs make this series ideal for performance-critical AKS nodes.

|Size Name|vCPUs (Qty.)|Memory (GB)|
|---|---|---|
|Standard_F2s_v2|2|4|
|Standard_F4s_v2|4|8|
|Standard_F8s_v2|8|16|
|Standard_F16s_v2|16|32|
|Standard_F32s_v2|32|64|
|Standard_F48s_v2|48|96|
|Standard_F64s_v2|64|128|
|Standard_F72s_v2|72|144

---

### 2. **Ddsv5-series** – _General Purpose with Local SSD (Ephemeral OS Disk Supported)_

- **Best for**: General-purpose AKS workloads with consistent traffic and some disk I/O sensitivity.
- **vCPU**: 2–96
- **Memory**: 8–384 GiB
- **Ephemeral OS Disk**: ✅
- **Ephemeral Disk IOPS**: Up to 450,000 ([link](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/general-purpose/ddsv5-series?tabs=sizestoragelocal))
- **Notes**: Balances compute and memory with the performance of ephemeral OS disks; suitable for most production AKS workloads.

| Size Name         | vCPUs (Qty.) | Memory (GB) |
| ----------------- | ------------ | ----------- |
| Standard_D2ds_v5  | 2            | 8           |
| Standard_D4ds_v5  | 4            | 16          |
| Standard_D8ds_v5  | 8            | 32          |
| Standard_D16ds_v5 | 16           | 64          |
| Standard_D32ds_v5 | 32           | 128         |
| Standard_D48ds_v5 | 48           | 192         |
| Standard_D64ds_v5 | 64           | 256         |
| Standard_D96ds_v5 | 96           | 384         |

---

### 3. **Edsv5-series** – _Memory Optimized with Local SSD (Ephemeral OS Disk Supported)_

- **Best for**: Memory-intensive workloads like caching layers, Redis, or JVM-based microservices.
- **vCPU**: 2–96
- **Memory**: 16–672 GiB
- **Ephemeral OS Disk**: ✅
- **Ephemeral Disk IOPS**: Up to 450,000 ([link](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/memory-optimized/edsv5-series?tabs=sizestoragelocal))
- **Notes**: Excellent for applications requiring large memory and fast local storage.  

|Size Name|vCPUs (Qty.)|Memory (GB)|
|---|---|---|
|Standard_E2ds_v5|2|16|
|Standard_E4ds_v5|4|32|
|Standard_E8ds_v5|8|64|
|Standard_E16ds_v5|16|128|
|Standard_E20ds_v5|20|160|
|Standard_E32ds_v5|32|256|
|Standard_E48ds_v5|48|384|
|Standard_E64ds_v5|64|512|
|Standard_E96ds_v5|96|672|
|Standard_E104ids_v5|104|672

### Summary

|VM Series|Ideal For|Ephemeral OS Disk|Highlights|
|---|---|---|---|
|**Fsv2**|High-traffic, CPU-bound apps|✅|Low cost, high burst, fast I/O|
|**Ddsv5**|General AKS workloads|✅|Balanced compute and disk speed|
|**Edsv5**|Memory-heavy workloads|✅|High memory + high ephemeral IOPS|

## The importance of IOPS

![[azure-disk-io-capping.png]]

> **⚠️ See Also:** 
> [Disk IO Capping](https://learn.microsoft.com/en-us/azure/virtual-machines/disks-performance#disk-io-capping)

When a disk attached to an Azure VM is **IOPS-saturated**, one of the side effects you'll observe is an increase in **CPU wait time** (sometimes referred to as **%iowait** in Linux, or high "Processor Queue Length" in Windows). Here's what happens in that context

###  CPU Wait Increases

- When a process issues a disk I/O request, the **CPU yields (waits)** until that I/O is completed.
- If the disk is saturated and can't serve I/O fast enough, the CPU spends more time **idle but not free** — it's waiting for the I/O to complete.
- This shows up in monitoring tools as high **I/O wait** or **CPU Wait Time**, even though overall CPU utilization may be low.

### Apparent Low CPU Utilization

- It might seem like your VM isn't "busy" (low CPU%), but in reality the CPU is bottlenecked **waiting** for I/O — a hidden performance bottleneck.
- On Linux: `iowait` in `top`, `vmstat`, or `sar` increases.
- On Windows: `Processor Queue Length` and `Avg. Disk sec/Read` or `/Write` increase, and the CPU may be underutilized.

### Performance Degradation

- Application performance degrades because:
    - The CPU can't proceed with work until disk I/O finishes.
    - Threads/processes block on I/O.
- In multi-threaded workloads, thread contention and context switching may rise.