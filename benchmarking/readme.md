# Homelab Benchmarking

Performance baselines and benchmarks for the homelab infrastructure.

## Systems

| System | Role | Documentation |
| ------ | ---- | ------------- |
| `pve01` | HP EliteDesk 800 G5 Mini / Proxmox VE | [G5 Performance: pve01](G5-Performance-pve01.md) |
| `pve02` | HP EliteDesk 800 G5 Mini / Proxmox VE | [G5 Performance: pve02](G5-Performance-pve02.md) |
| Zyxel NAS326 | Shared NFS storage | Documented within the node network benchmarks |
| NETGEAR GS108E | Managed network switch | Documented within the node network benchmarks |

## Benchmark Status

| System / Workload | CPU | Memory | Storage | Network |
| ----------------- | :--: | :----: | :-----: | :-----: |
| `pve01` host | ✅ | ✅ | ✅ | ✅ |
| `pve02` host | ✅ | ✅ | ✅ | ✅ |
| `lab-core01` on `pve01` | ✅ | ✅ | ✅ | ✅ |
| `lab-core01` on `pve02` | ✅ | ✅ | ✅ | ✅ |
| Zyxel NAS / NFS | N/A | N/A | ✅ | ✅ |

## Key Results

The two G5 systems were benchmarked both as Proxmox hosts and by running the same `lab-core01` VM on each node.

### Host Baselines

`pve01` was upgraded in August 2026 from 16 GB to 32 GB DDR4 using two 16 GB SODIMMs. The current host memory baseline below reflects the 32 GB configuration. The original 16 GB results remain documented in the pve01 benchmark record for comparison.

| Metric | `pve01` | `pve02` |
| ------ | ------: | ------: |
| CPU, 1 thread | 1,221.43 events/sec | **1,438.38 events/sec** |
| CPU, 4 threads | — | **5,543.02 events/sec** |
| CPU, 6 threads | **6,734.57 events/sec** | — |
| Memory read, 1 thread | **24,180.55 MiB/sec** | **29,606.82 MiB/sec** |
| Memory write, 1 thread | **21,487.18 MiB/sec** | **25,190.33 MiB/sec** |
| Memory read, 6 threads | **106,762.51 MiB/sec** | — |
| Memory write, 6 threads | **78,147.37 MiB/sec** | — |
| Raw NVMe sequential read | 2,064 MiB/s | **2,990 MiB/s** |

The host CPU results are not directly interchangeable because `pve01` uses the 35 W i5-9500T and `pve02` uses the 65 W i5-9500. The node-specific documents retain the appropriate native test configuration.

### pve01 Memory Upgrade

The documented memory comparison is:

| Operation | 16 GB Baseline | 32 GB | Change |
| --- | ---: | ---: | ---: |
| Read, 1 thread | 25,645.48 MiB/sec | **24,180.55 MiB/sec** | **−5.71%** |
| Write, 1 thread | 21,665.73 MiB/sec | **21,487.18 MiB/sec** | **−0.82%** |

The upgrade's primary benefit is doubling available memory from 16 GB to 32 GB and enabling a populated dual-channel configuration. The single-thread memory bandwidth results did not materially improve.

### lab-core01 A/B Comparison

The same VM was tested on both nodes using 4 vCPUs and 12 GB RAM.

| Test | `pve01` | `pve02` | Improvement |
| ---- | ------: | ------: | ----------: |
| CPU 1T | 1,120.76 events/s | **1,436.78 events/s** | **+28.2%** |
| CPU 4T | 4,427.28 events/s | **5,559.86 events/s** | **+25.6%** |
| Memory read | 23,835.9 MiB/s | **29,181.1 MiB/s** | **+22.4%** |
| Memory write | 20,334.1 MiB/s | **25,108.7 MiB/s** | **+23.5%** |
| 4K random read | 232K IOPS | **300K IOPS** | **+29.3%** |
| 4K random write | 222K IOPS | **286K IOPS** | **+29.0%** |

The A/B testing showed a consistent performance advantage for `pve02`. `lab-core01` has therefore been migrated to `pve02` as its preferred Proxmox placement.

## Network Baseline

The two Proxmox nodes are connected through the NETGEAR GS108E using 1 Gb Ethernet.

`iperf3` measured:

* `pve01` → `pve02`: **934 Mbits/sec**, 0 retransmits
* `pve02` → `pve01`: **935 Mbits/sec**, 0 retransmits

This is approximately 93.4% of nominal 1 Gb Ethernet line rate and indicates a healthy node-to-node network path.

## NAS / NFS Baseline

The Zyxel NAS326 provides shared NFS storage to `lab-core01` at `/mnt/nas`.

Current measured results include:

| Workload | Bandwidth | Average Latency |
| -------- | --------: | --------------: |
| Sequential read, 1 MiB | **97.7 MiB/s** | 163.6 ms |
| Sequential write, 1 MiB | **18.0 MiB/s** | 890.8 ms |
| 4K random read | **475 KiB/s** | 268.4 ms |
| 4K random write | **39.9 MiB/s** | 3.13 ms |

The NAS is therefore best treated as shared capacity, backup, and archive storage rather than high-performance local storage. Local NVMe storage is preferred for latency-sensitive VM and container workloads.

## Power Measurement

Whole-system power consumption has **not** yet been measured. A suitable power meter was not available during the G5 comparison.

CPU TDP and NVMe power-state values are not being used as substitutes for whole-system measurements.

## Planned Comparisons

* Whole-system idle and load power on `pve01` and `pve02`
* CPU temperature and sustained-load behavior
* 1 TB Optimus 5001 NVMe versus the current 256 GB drives
* Local NVMe versus Zyxel NFS for representative workloads
* Future network upgrades beyond 1 Gb Ethernet

## Legacy Documentation

`G5-Performance.md` is the original G5 benchmark document. The authoritative node-specific baselines are now:

* [G5 Performance: pve01](G5-Performance-pve01.md)
* [G5 Performance: pve02](G5-Performance-pve02.md)

Future benchmark updates should be made to the node-specific documents rather than the legacy file.

## Baseline Date

**August 2026**

**Last updated:** August 30, 2026
