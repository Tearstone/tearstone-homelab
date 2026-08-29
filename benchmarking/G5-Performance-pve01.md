# G5 Performance: pve01

Performance baseline for the HP EliteDesk 800 G5 Mini running Proxmox VE on `pve01`.

## System

| Component | Specification |
| --- | --- |
| System | HP EliteDesk 800 G5 Mini |
| Proxmox Node | `pve01` |
| CPU | Intel Core i5-9500T |
| CPU Cores | 6 |
| CPU Threads | 6 |
| CPU Base Frequency | 2.2 GHz |
| CPU Max Turbo | 3.7 GHz |
| RAM | 32 GB DDR4 |
| RAM Configuration | 2 × 16 GB SODIMM |
| RAM Speed | 2667 MT/s |
| RAM Maximum | 32 GB |
| Storage | Toshiba KXG50ZNV256G NVMe |
| Storage Capacity | 256 GB |
| Proxmox VE | 9.2.2 |
| Kernel | 7.0.2-6-pve |
| Network | 1 Gb Ethernet |
| Network Interface | `nic0` |
| Proxmox Bridge | `vmbr0` |

`pve01` is the first physical node in the two-node `nexus` Proxmox cluster. It uses the 35 W-class Intel Core i5-9500T processor.

## Network Configuration

The G5 is connected to the homelab network using its physical Ethernet interface.

| Property | Value |
| --- | --- |
| Interface | `nic0` |
| Proxmox Bridge | `vmbr0` |
| Link Speed | 1000 Mb/s |
| Duplex | Full |
| Auto Negotiation | Enabled |
| Link Status | Up |

The physical network path is:

```text
pve01
 │
 │ 1 Gb Ethernet
 ▼
NETGEAR GS108E
 │
 ├── pve02
 ├── Zyxel NAS
 ├── Proxmox guests
 └── Other network devices
```

A 1 Gb Ethernet connection provides a theoretical maximum of 125 MB/s before protocol and filesystem overhead.

### Network Benchmark

Network throughput was measured with `iperf3` between `pve01` and `pve02`.

| Direction | Throughput | Retransmits |
| --- | ---: | ---: |
| `pve01` → `pve02` | **934 Mbits/sec** | 0 |
| `pve02` → `pve01` | **935 Mbits/sec** | 0 |

The measured throughput is approximately 93.4% of the nominal 1 Gb Ethernet line rate and is consistent with a healthy 1 GbE connection after protocol overhead.

## Storage Configuration

The Proxmox installation resides on the Toshiba NVMe drive.

```text
256 GB Toshiba KXG50ZNV256G
└── LVM
    ├── pve-swap
    ├── pve-root
    └── pve-data
```

Storage benchmarks are performed using controlled test files or test volumes rather than destructive whole-device testing because this is the active Proxmox system disk.

## CPU Benchmark

CPU performance was measured using `sysbench` directly on the Proxmox host.

Parameters:

```text
Benchmark: CPU
Prime limit: 10,000
Duration: 60 seconds
```

Official host baseline with guests stopped:

| Threads | Events/sec | Avg Latency |
| ---: | ---: | ---: |
| 1 | **1,221.43** | **0.82 ms** |
| 6 | **6,734.57** | **0.89 ms** |

The initial single-thread test performed with guests running produced 1,170.16 events/sec and is retained only as historical context.

## Memory Benchmark

The system was upgraded in August 2026 from a single 16 GB DDR4-2667 SODIMM to two 16 GB DDR4-2667 SODIMMs for a total of 32 GB. The modules are installed in Channel A and Channel B.

### Original 16 GB Baseline

The original baseline used:

```text
Benchmark: memory
Block size: 1 MiB
Total transfer: 10 GiB
Threads: 1
Scope: global
```

| Operation | 16 GB Baseline |
| --- | ---: |
| Read | **25,645.48 MiB/sec** |
| Write | **21,665.73 MiB/sec** |

### 32 GB Upgrade Results

The same 1 thread test was repeated after the RAM upgrade using the same parameters.

| Operation | 16 GB Baseline | 32 GB | Change |
| --- | ---: | ---: | ---: |
| Read | 25,645.48 MiB/sec | **24,180.55 MiB/sec** | **−5.71%** |
| Write | 21,665.73 MiB/sec | **21,487.18 MiB/sec** | **−0.82%** |

The single-thread results show no meaningful memory bandwidth improvement from the RAM upgrade. The primary benefit of the upgrade is the increase from 16 GB to 32 GB of available memory and the ability to operate with two populated memory channels.

A 6 thread memory test was also performed after the upgrade using the same 1 MiB block size and 10 GiB total transfer:

| Operation | 32 GB Result |
| --- | ---: |
| Read | **106,762.51 MiB/sec** |
| Write | **78,147.37 MiB/sec** |

The existing 16 GB benchmark documentation does not contain a corresponding 6 thread, 10 GiB memory result, so a percentage improvement for the 6 thread test cannot be calculated from the documented baseline without inventing a value. The 6 thread results above are retained as the new 32 GB multi-thread memory baseline.

## lab-core01 Workload Placement Benchmark

The same `lab-core01` VM was tested on `pve01` and `pve02` using 4 vCPUs and 12 GB RAM.

| Test | `lab-core01` on `pve01` | `lab-core01` on `pve02` | Change |
| --- | ---: | ---: | ---: |
| CPU 1T | 1,120.76 events/s | **1,436.78 events/s** | **+28.2%** |
| CPU 4T | 4,427.28 events/s | **5,559.86 events/s** | **+25.6%** |
| Memory read | 23,835.9 MiB/s | **29,181.1 MiB/s** | **+22.4%** |
| Memory write | 20,334.1 MiB/s | **25,108.7 MiB/s** | **+23.5%** |
| 4K random read | 232K IOPS | **300K IOPS** | **+29.3%** |
| 4K random write | 222K IOPS | **286K IOPS** | **+29.0%** |

The comparison showed a consistent performance advantage for `pve02` across CPU, memory bandwidth, and random I/O. `lab-core01` was therefore migrated to `pve02` and remains there as its preferred placement.

## NVMe Health Baseline

The Toshiba XG5 is an older 256 GB OEM NVMe SSD with significant prior operating time. Current health data indicates that it remains usable for the present workload.

| Metric | Result |
| --- | ---: |
| Temperature | 32°C |
| Available spare | 100% |
| Percentage used | 47% |
| Data read | 37.98 TB |
| Data written | 56.40 TB |
| Power-on hours | 50,332 |
| Power cycles | 336 |
| Unsafe shutdowns | 130 |
| Media errors | 0 |
| Error log entries | 0 |
| Critical warnings | 0 |

The drive should be considered aging but currently healthy. Future checks should monitor percentage used, available spare, media errors, error-log entries, unsafe shutdowns, temperature, and power-on hours.

## NVMe Benchmark

The raw-device sequential-read test used `fio` with 1 MiB blocks, queue depth 32, direct I/O, and a 30-second read-only runtime.

Primary physical baseline:

**2,064 MiB/s raw sequential read**

Additional testing against an 8 GB LVM-thin test volume produced approximately:

* 155K 4K random-read IOPS
* 62.2K 4K random-write IOPS
* 328 MiB/s sequential write

Logical-device read results that exceeded the physical device's expected capability were treated as storage-layer/cache behavior rather than SSD throughput.

## NAS / NFS Benchmark

The Zyxel NAS326 is used as shared network storage for the homelab. Testing was performed from `lab-core01` against the NFS-mounted storage.

| Workload | Bandwidth | Average Latency |
| --- | ---: | ---: |
| Sequential read, 1 MiB | **97.7 MiB/s** | 163.6 ms |
| Sequential write, 1 MiB | **18.0 MiB/s** | 890.8 ms |
| 4K random read | **475 KiB/s** | 268.4 ms |
| 4K random write | **39.9 MiB/s** | 3.13 ms |

The NAS is best treated as shared capacity, backup, and archive storage rather than high-performance local storage. Local NVMe is preferred for latency-sensitive VM and container workloads.

## Power Measurement

Whole-system power consumption has not yet been measured because a suitable power meter was not available during the G5 comparison.

## Public Documentation Policy

This public benchmark intentionally omits private IP addresses, MAC addresses, serial numbers, and internal DNS names. The benchmark results remain because they are useful for evaluating the hardware without exposing internal network identifiers.

## Baseline Date

August 2026
