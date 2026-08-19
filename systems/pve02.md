# pve02

## Purpose

Second Proxmox VE node in the `nexus` cluster.

The node provides additional compute capacity and is intended for controlled workload migration, performance comparison, and future cluster and high-availability experiments.

## Hardware

System:
- HP EliteDesk 800 G5 Desktop Mini

CPU:
- Intel Core i5-9500
- 6 cores
- 6 threads
- 3.0 GHz base frequency

Memory:
- 16 GB DDR4
- 2 × 8 GB

Storage:
- WDC/SanDisk PC SN730 SDBQNTY-256G-1001 NVMe
- 238.5 GiB detected capacity
- SMART health: PASSED
- Temperature observed during testing: 34 C
- Percentage used: 13%
- Data written: approximately 18.8 TB
- Media and data integrity errors: 0

Network:
- Intel Ethernet Connection I219-LM
- Intel Wi-Fi 6 AX200

Graphics:
- Intel UHD Graphics 630

Firmware:
- HP R21 Ver. 02.26.00
- Firmware date: 2026-05-05

## Proxmox

Hostname:
- `pve02`

Cluster:
- `nexus`

Proxmox VE:
- 9.2.2

Kernel:
- 7.0.2-6-pve

## Storage Configuration

The 256 GB NVMe was installed using a deliberately lean Proxmox layout:

```text
/dev/nvme0n1
└── 238.5 GiB NVMe
    ├── EFI                 1 GiB
    └── LVM                237.5 GiB
        ├── pve-swap        4 GiB
        ├── pve-root       16 GiB ext4
        └── pve-data      ~197.4 GiB LVM-thin
```

Approximately 16 GiB of LVM capacity is intentionally retained as free volume-group space for future flexibility.

The Proxmox host filesystem is intentionally kept small so that the majority of the local SSD is available for guest storage.

## Network Configuration

The physical Ethernet interface is presented to Proxmox as `nic0` and connected to `vmbr0`.

```text
nic0
  |
  +-- vmbr0
       internal address omitted from public documentation
```

The node uses wired Ethernet for Proxmox cluster communication. The installed Wi-Fi adapter is not used for cluster traffic.

## Cluster Membership

`pve02` is node 2 of the `nexus` Proxmox cluster.

Current membership:

```text
nexus
├── pve
└── pve02
```

The cluster is currently quorate with two expected votes and two total votes.

## Host Benchmark Results

The host's 256 GB SN730 NVMe was tested directly with fio using a 1 MiB sequential read workload:

```text
~2,990 MiB/s read
~3,135 MB/s reported by fio
```

The host also produced approximately 217K 4K random read IOPS and 134K 4K random write IOPS against an 8 GB test logical volume at queue depth 32. These direct-host results establish the baseline for the installed NVMe and LVM-thin storage path.

CPU testing with sysbench produced approximately:

```text
1 thread: 1,438 events/s
6 threads: 8,131 events/s
```

Memory testing with 1 MiB blocks produced approximately:

```text
read: 29,946 MiB/s
write: 25,063 MiB/s
```

## lab-core01 Workload Placement Test

`lab-core01` was used as the first controlled workload comparison between `pve01` and `pve02`. The VM has 4 vCPUs, 12 GB RAM, and an 80 GB virtual disk using `virtio-scsi-single` with I/O thread enabled.

The same benchmark suite was run on the VM while hosted on each Proxmox node. Final storage tests used `sync` and `echo 3 > /proc/sys/vm/drop_caches` before each fio run.

| Test | `pve01` | `pve02` | Improvement |
| ---- | ------: | ------: | ----------: |
| CPU 1T | 1,120.76 events/s | 1,436.78 events/s | +28.2% |
| CPU 4T | 4,427.28 events/s | 5,559.86 events/s | +25.6% |
| Memory read | 23,835.9 MiB/s | 29,181.1 MiB/s | +22.4% |
| Memory write | 20,334.1 MiB/s | 25,108.7 MiB/s | +23.5% |
| 4K random read | 232K IOPS | 300K IOPS | +29.3% |
| 4K random write | 222K IOPS | 286K IOPS | +29.0% |

The preliminary cached pve02 storage result of approximately 462K read IOPS and 455K write IOPS was discarded from the official comparison. The cache-cleared results above are the authoritative measurements.

The results showed a consistent performance advantage for `pve02` across CPU, memory bandwidth, and random storage I/O. `lab-core01` was migrated to `pve02` and is now the preferred host for this workload.

## lab-core01 Memory Observation

After migration, the Proxmox API reported:

```text
maxmem:  12.00 GiB
mem:      3.50 GiB
memhost:  9.56 GiB
```

QEMU reported:

```text
actual=12288 MiB
max_mem=12288 MiB
total_mem=11900 MiB
free_mem=9664 MiB
```

Inside the Debian guest, `free -h` showed approximately 2.0 GiB used, 9.5 GiB free, 948 MiB buff/cache, and 9.6 GiB available. Approximately 437 MiB of the guest's 2 GiB swap was in use.

The VM retains its full 12 GB allocation. The observed reduction in the Proxmox memory usage graph after migration is not caused by ballooning reclaiming RAM because QEMU reports `actual=12288` and `max_mem=12288`. The current workload has a relatively small active working set and substantial available memory.

## Power Measurement

Power consumption has not yet been measured because a suitable external power meter is not currently available. Power efficiency remains an open measurement for a future test.

## Public Documentation Policy

Actual internal IP addresses, MAC addresses, serial numbers, internal DNS names, and other environment-specific network identifiers are intentionally omitted from this public repository.
