# G5 Performance: pve02

Performance baseline for the HP EliteDesk 800 G5 Mini running Proxmox VE on `pve02`.

## System

| Component | Specification |
| --- | --- |
| System | HP EliteDesk 800 G5 Mini |
| Proxmox Node | `pve02` |
| CPU | Intel Core i5-9500 |
| CPU Cores | 6 |
| CPU Threads | 6 |
| CPU Base Frequency | 3.0 GHz |
| RAM | 32 GB DDR4 |
| RAM Configuration | 2 × 16 GB SODIMM |
| RAM Rated Speed | 3200 MT/s |
| RAM Configured Speed | 2667 MT/s |
| System Storage | WDC/SanDisk PC SN730 256 GB NVMe |
| Additional Storage | SanDisk Optimus 5001 1 TB NVMe |
| Proxmox VE | 9.2.2 |
| Kernel | 7.0.2-6-pve |
| Network | 1 Gb Ethernet |
| Network Interface | `nic0` |
| Proxmox Bridge | `vmbr0` |

`pve02` is the second physical node in the two-node `nexus` Proxmox cluster.

The node was upgraded in September 2026 from 16 GB to 32 GB using two 16 GB DDR4-3200 SODIMMs. The EliteDesk platform currently configures the memory at 2667 MT/s.

## Network Configuration

The node uses wired Ethernet for Proxmox cluster communication. The installed Wi-Fi adapter is not used for cluster traffic.

```text
pve02
 │
 │ 1 Gb Ethernet
 ▼
NETGEAR GS108E
```

### Network Benchmark

`iperf3` measured approximately 934–935 Mbits/sec between the two Proxmox nodes with zero retransmits in both directions.

| Direction | Throughput | Retransmits |
| --- | ---: | ---: |
| `pve01` → `pve02` | **934 Mbits/sec** | 0 |
| `pve02` → `pve01` | **935 Mbits/sec** | 0 |

## Storage Configuration

### System NVMe

The original 256 GB NVMe remains the Proxmox system disk and contains the existing `pve` volume group:

```text
238.5 GiB NVMe
└── LVM
    ├── pve-swap        4 GiB
    ├── pve-root       16 GiB ext4
    └── pve-data      ~197.4 GiB LVM-thin
```

The `pve` volume group retains approximately 16 GiB of free capacity.

### 1 TB NVMe Expansion

A SanDisk Optimus 5001 1 TB NVMe was added as a second physical drive. The drive is approximately 931.5 GiB and was configured as a dedicated LVM-thin storage tier:

```text
931.5 GiB SanDisk Optimus 5001
└── GPT
    └── nvme0n1p1     931.5 GiB Linux LVM
        └── pve-fast VG
            └── data  ~931.3 GiB LVM-thin
```

Proxmox exposes this volume group and thin pool as:

```text
Storage ID: nvme-lvm
Type:       lvmthin
VG:         pve-fast
Thin pool:  data
Content:    images, rootdir
```

The thin pool was created across the available capacity of the 1 TB NVMe. LVM selected a 512 KiB thin-pool chunk size and allocated approximately 120 MiB for metadata.

## 1 TB NVMe Raw Performance

The Optimus 5001 was tested as a raw device before partitioning and deployment. Tests used `fio` 3.39 with the `io_uring` I/O engine, direct I/O, one worker, queue depth 32, and 30-second time-based runs.

### Sequential Read

```text
Block size: 1 MiB
IO depth:   32
Result:     3,425 MiB/s
            3,592 MB/s
```

Average latency was approximately 9.34 ms at queue depth 32. The device sustained approximately 3.38–3.44 GiB/s throughout the run.

### Sequential Write

```text
Block size: 1 MiB
IO depth:   32
Result:     3,216 MiB/s
            3,372 MB/s
```

Average latency was approximately 9.95 ms at queue depth 32. The device sustained approximately 3.18–3.23 GiB/s throughout the run.

### 4K Random Read

```text
Block size: 4 KiB
IO depth:   32
Result:     355.6K IOPS
            1,389 MiB/s
```

Average latency was approximately 89.7 microseconds. The 99th percentile latency was approximately 124 microseconds.

### 4K Random Write

```text
Block size: 4 KiB
IO depth:   32
Result:     102.4K IOPS
            400 MiB/s
```

Average latency was approximately 312 microseconds. The 99th percentile latency was approximately 938 microseconds.

### 70/30 Random Mixed Read/Write

```text
Read/write mix: 70/30
Block size:     4 KiB
IO depth:       32
Read:           63.0K IOPS / 246 MiB/s
Write:          27.0K IOPS / 105 MiB/s
Total:          89.9K IOPS / 351 MiB/s
```

Average read latency was approximately 464 microseconds. Average write latency was approximately 102 microseconds.

### Raw NVMe Summary

| Test | Result |
| --- | ---: |
| Sequential read, 1 MiB QD32 | **3,425 MiB/s** |
| Sequential write, 1 MiB QD32 | **3,216 MiB/s** |
| 4K random read QD32 | **355.6K IOPS** |
| 4K random write QD32 | **102.4K IOPS** |
| 4K random 70/30 read | **63.0K IOPS** |
| 4K random 70/30 write | **27.0K IOPS** |

These results are raw-device measurements and should not be treated as direct measurements of guest VM performance through Proxmox LVM-thin. A future benchmark can establish the performance of the complete `nvme-lvm` storage path under an actual VM workload.

## Host Benchmark Results

### CPU

The existing CPU baseline was established before the September 2026 memory upgrade. The memory change does not invalidate the CPU result.

```text
1 thread: 1,438.38 events/s
4 threads: 5,543.02 events/s
```

### Memory

The original 16 GB baseline was measured using 1 MiB blocks, 10 GiB total transfer, one thread, and global scope. The system was subsequently upgraded to 32 GB using two 16 GB SODIMMs.

#### Original 16 GB Baseline

| Operation | 16 GB Baseline |
| --- | ---: |
| Read | **29,946 MiB/s** |
| Write | **25,063 MiB/s** |

#### 32 GB Upgrade Results

The same 1 thread test was repeated after the RAM upgrade.

| Operation | 16 GB Baseline | 32 GB | Change |
| --- | ---: | ---: | ---: |
| Read | 29,946 MiB/s | **29,995.98 MiB/s** | **+0.17%** |
| Write | 25,063 MiB/s | **25,693.97 MiB/s** | **+2.51%** |

The single-thread results show essentially unchanged memory bandwidth after doubling capacity. The primary benefit of the upgrade is increasing available memory from 16 GB to 32 GB.

Additional four-thread testing was performed after the upgrade using 1 MiB blocks and a 20 GiB total transfer:

| Operation | Threads | Total Transfer | Result |
| --- | ---: | ---: | ---: |
| Read | 4 | 20 GiB | **112,454.39 MiB/s** |
| Write | 4 | 20 GiB | **85,633.22 MiB/s** |

The existing 16 GB benchmark documentation does not contain corresponding four-thread, 20 GiB memory results, so a percentage improvement cannot be calculated for those tests without inventing a baseline. The four-thread results are retained as the new 32 GB multi-thread memory baseline.

The installed modules are rated for 3200 MT/s but the platform currently configures them at 2667 MT/s. The benchmark results therefore represent the system's actual configured memory speed rather than the modules' rated maximum.

## lab-core01 Workload Placement Test

`lab-core01` was used as the first controlled workload comparison between `pve01` and `pve02`. The VM has 4 vCPUs, 12 GB RAM, and an 80 GB virtual disk using `virtio-scsi-single` with I/O thread enabled.

Final storage tests used `sync` and `echo 3 > /proc/sys/vm/drop_caches` before each fio run.

| Test | `pve01` | `pve02` | Improvement |
| --- | ---: | ---: | ---: |
| CPU 1T | 1,120.76 events/s | 1,436.78 events/s | +28.2% |
| CPU 4T | 4,427.28 events/s | 5,559.86 events/s | +25.6% |
| Memory read | 23,835.9 MiB/s | 29,181.1 MiB/s | +22.4% |
| Memory write | 20,334.1 MiB/s | 25,108.7 MiB/s | +23.5% |
| 4K random read | 232K IOPS | 300K IOPS | +29.3% |
| 4K random write | 222K IOPS | 286K IOPS | +29.0% |

The preliminary cached pve02 storage result of approximately 462K read IOPS and 455K write IOPS was discarded from the official comparison. The cache-cleared results above are the authoritative measurements.

The results showed a consistent performance advantage for `pve02` across CPU, memory bandwidth, and random storage I/O. `lab-core01` was migrated to `pve02` and is now the preferred host for this workload.

The VM's 80 GB system disk now resides on `nvme-lvm`, the dedicated storage tier provided by the 1 TB Optimus 5001. The online migration was performed with `qm move_disk` and the original `local-lvm` volume was removed only after the mirror completed successfully.

## lab-core01 Post-Migration Validation

After migration, the Proxmox VM configuration showed:

```text
VMID:       100
Name:       lab-core01
Status:     running
Disk:       nvme-lvm:vm-100-disk-0
Disk size:  80 GB
Controller: virtio-scsi-single
I/O thread: enabled
Memory:     12 GB
vCPU:       4
```

The guest continued to see an 80 GB system disk after migration. Docker was operational and all four Immich containers were reported healthy during validation.

## lab-core01 Memory Observation

After migration, Proxmox reported approximately 3.5 GiB of active VM memory and 9.56 GiB available to the guest. Inside Debian, `free -h` showed approximately 2.0 GiB used, 9.5 GiB free, and 9.6 GiB available, with approximately 437 MiB of swap in use.

QEMU reported:

```text
actual=12288 MiB
max_mem=12288 MiB
total_mem=11900 MiB
free_mem=9664 MiB
```

The VM retains its full 12 GB allocation. The observed reduction in the Proxmox memory graph is not caused by ballooning reclaiming RAM.

## Power Measurement

Power consumption has not yet been measured because a suitable external power meter is not currently available.

## Public Documentation Policy

This public benchmark intentionally omits private IP addresses, MAC addresses, serial numbers, and internal DNS names. Benchmark methodology and results remain documented because they do not require exposing internal network identifiers.

## Baseline Date

September 2026
