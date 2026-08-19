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
| RAM | 16 GB DDR4 |
| RAM Configuration | 2 × 8 GB SODIMM |
| Storage | WDC/SanDisk PC SN730 SDBQNTY-256G-1001 NVMe |
| Storage Capacity | 256 GB |
| Proxmox VE | 9.2.2 |
| Kernel | 7.0.2-6-pve |
| Network | 1 Gb Ethernet |
| Network Interface | `nic0` |
| Proxmox Bridge | `vmbr0` |

`pve02` is the second physical node in the two-node `nexus` Proxmox cluster.

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

The 256 GB NVMe was installed using a deliberately lean Proxmox layout:

```text
238.5 GiB NVMe
└── LVM
    ├── pve-swap        4 GiB
    ├── pve-root       16 GiB ext4
    └── pve-data      ~197.4 GiB LVM-thin
```

Approximately 16 GiB of LVM capacity is intentionally retained as free volume-group space for future flexibility.

## Host Benchmark Results

### CPU

```text
1 thread: 1,438 events/s
6 threads: 8,131 events/s
```

### Memory

```text
read: 29,946 MiB/s
write: 25,063 MiB/s
```

### Local NVMe

Raw-device sequential read:

```text
~2,990 MiB/s
~3,135 MB/s reported by fio
```

An 8 GB LVM test volume produced approximately 217K 4K random-read IOPS and 134K 4K random-write IOPS at queue depth 32. These results establish the baseline for the installed NVMe and LVM-thin storage path.

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

August 2026
