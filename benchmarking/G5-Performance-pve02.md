# G5 Performance: pve02

Performance baseline for the second HP EliteDesk 800 G5 Mini running Proxmox VE.

This document is intentionally separate from `G5-Performance.md`, which contains the original `pve` baseline. The two systems are similar G5 Mini platforms but use different CPUs and storage devices.

## System

| Component          | Specification |
| ------------------ | ------------- |
| System             | HP EliteDesk 800 G5 Mini |
| Proxmox Node        | `pve02` |
| IP Address          | `192.168.12.248/24` |
| CPU                | Intel Core i5-9500 |
| CPU Cores          | 6 |
| CPU Threads        | 6 |
| RAM                | 16 GB DDR4 |
| RAM Configuration  | 1 × 16 GB SODIMM |
| RAM Slots          | 2 |
| RAM Maximum        | 32 GB |
| Storage            | WDC PC SN730 SDBQNTY-256G-1001 NVMe |
| Storage Capacity   | 256 GB |
| Proxmox VE         | 9.2.2 |
| Kernel             | 7.0.2-6-pve |
| Network            | 1 Gb Ethernet |
| Proxmox Bridge     | `vmbr0` |

`pve02` is the second physical node in the two-node `nexus` Proxmox cluster. It uses the standard 65 W-class i5-9500 rather than the 35 W-class i5-9500T installed in `pve`.

The system currently contains a single 16 GB memory module. A future second 16 GB module would increase capacity to 32 GB and enable dual-channel operation.

## Network Benchmark

The physical network interface is connected through the NETGEAR GS108E managed switch.

The network was tested with `iperf3` between `pve02` (`192.168.12.248`) and `pve` (`192.168.12.247`).

### Results

| Direction | Throughput | Retransmits |
| --------- | ---------: | ----------: |
| `pve02` → `pve` | **935 Mbits/sec** | 0 |
| `pve` → `pve02` | **934 Mbits/sec** | 0 |

The measured throughput is approximately 93.4% of the nominal 1 Gb Ethernet line rate and is consistent with a healthy 1 GbE connection after protocol overhead.

The network is therefore not showing an obvious performance problem between the two Proxmox nodes.

## Storage Configuration

The Proxmox installation resides on the WDC NVMe drive.

```text
/dev/nvme0n1
└── 256 GB WDC PC SN730 SDBQNTY-256G-1001
    ├── EFI
    └── LVM
        ├── pve-swap       4 GB
        ├── pve-root      16 GB
        └── pve-data     197.4 GB
```

The LVM-thin `pve-data` pool currently contains the Proxmox VM storage.

### NVMe Information

| Property | Value |
| -------- | ----- |
| Model | WDC PC SN730 SDBQNTY-256G-1001 |
| Serial | `204919801031` |
| Firmware | `11170101` |
| NVMe Version | 1.3 |
| Capacity | 256,060,514,304 bytes / 256 GB |
| Namespace | 1 |

### NVMe Health Baseline

Health information was captured using `smartctl -a /dev/nvme0n1`.

| Metric | Result |
| ------ | -----: |
| Temperature | 34°C |
| Available spare | 100% |
| Percentage used | 13% |
| Data read | 8.44 TB |
| Data written | 18.8 TB |
| Power-on hours | 44,724 |
| Power cycles | 81 |
| Unsafe shutdowns | 37 |
| Media/data integrity errors | 0 |
| Error log entries | 1 |
| Critical warnings | 0 |

The NVMe reports a healthy overall status, 100% available spare capacity, 13% endurance consumed, and no media or data integrity errors. The drive has approximately 44,724 power-on hours, so it is not new, but its reported endurance and error indicators are currently favorable.

The SMART output also showed no active self-test and no errors in the current error log contents despite the lifetime `Error Information Log Entries: 1` counter.

### Supported Power States

The controller reports the following NVMe power states:

| State | Maximum Power |
| ----- | ------------: |
| PS0 | 5.00 W |
| PS1 | 3.50 W |
| PS2 | 3.00 W |
| PS3 | 0.070 W |
| PS4 | 0.0035 W |

These are controller-supported power-state limits, not measurements of whole-system power consumption.

## CPU Benchmark

CPU performance was measured from the `lab-core01` VM after it was migrated to `pve02`.

The VM was configured with 4 vCPUs. The benchmark therefore measures the practical CPU performance available to the workload through the Proxmox/QEMU virtualization stack rather than a bare-metal CPU score.

### Method

```text
Benchmark: sysbench CPU
Prime limit: 10,000
Duration: 60 seconds
Threads: 1 and 4
```

### Results

| Threads | Events/sec | Average Latency |
| ------: | ----------: | --------------: |
| 1 | **1,436.78** | 0.70 ms |
| 4 | **5,559.86** | 0.72 ms |

These results are the current workload-placement baseline for `lab-core01` on `pve02`.

## Memory Benchmark

The host contains a single 16 GB DDR4 module. Memory performance was measured from `lab-core01` using the same benchmark parameters used during the workload-placement comparison.

### Method

```text
Benchmark: sysbench memory
Block size: 1 MiB
Total transfer: 10 GiB
Threads: 1
Scope: global
```

### Results

| Operation | Throughput |
| --------- | ---------: |
| Read | **29,181.12 MiB/sec** |
| Write | **25,108.70 MiB/sec** |

The results establish the current single-module memory baseline. A future 32 GB dual-channel configuration should be tested using the same parameters to measure the effect of the second DIMM.

## NVMe Benchmark

### Raw NVMe Sequential Read

A nondestructive raw-device sequential-read test was performed against `/dev/nvme0n1`.

```text
Device: /dev/nvme0n1
Operation: Sequential read
Block size: 1 MiB
IO engine: libaio
Queue depth: 32
Duration: 30 seconds
Direct I/O: Enabled
Read-only: Enabled
```

### Result

| Test | Result |
| ---- | -----: |
| Raw sequential read | **2,990 MiB/s** |
| Raw sequential read IOPS | **2,990 IOPS** |
| Average latency | **10.70 ms** |

The raw-device result is the preferred baseline for physical SSD sequential-read performance.

### LVM-Thin 4K Random I/O

An 8 GB LVM logical volume named `fio-test` was temporarily created in the `pve` volume group and used for controlled random I/O testing. The test volume was removed after testing.

```text
/dev/pve/fio-test
```

Test parameters:

```text
Block size: 4 KiB
Queue depth: 32
IO engine: libaio
Direct I/O: Enabled
Duration: 60 seconds
Jobs: 1
```

### Results

| Workload | IOPS | Bandwidth | Average Latency |
| -------- | ---: | --------: | --------------: |
| 4K random read | **217K IOPS** | **849 MiB/s** | **146.67 µs** |
| 4K random write | **134K IOPS** | **524 MiB/s** | **238.01 µs** |

The random-read test completed approximately 13.0 million I/O operations during the 60-second run. The random-write test completed approximately 8.05 million I/O operations.

These results measure the Proxmox LVM-thin storage path rather than the raw NAND media alone and should be interpreted accordingly.

## lab-core01 Workload Placement Benchmark

The primary reason for benchmarking `pve02` was to determine whether `lab-core01` was better suited to the second G5 system.

The same VM was tested on both nodes using the same 4 vCPU and 12 GB RAM configuration.

### Controlled Results

| Test | `lab-core01` on `pve` | `lab-core01` on `pve02` | Change |
| ---- | --------------------: | -----------------------: | -----: |
| CPU 1T | 1,120.76 events/s | **1,436.78 events/s** | **+28.2%** |
| CPU 4T | 4,427.28 events/s | **5,559.86 events/s** | **+25.6%** |
| Memory read | 23,835.9 MiB/s | **29,181.1 MiB/s** | **+22.4%** |
| Memory write | 20,334.1 MiB/s | **25,108.7 MiB/s** | **+23.5%** |
| 4K random read | 232K IOPS | **300K IOPS** | **+29.3%** |
| 4K random write | 222K IOPS | **286K IOPS** | **+29.0%** |

The storage values in this table are the controlled cache-cleared VM filesystem tests, not the raw NVMe test or the LVM-thin test-volume results.

The comparison showed a consistent performance advantage for `pve02` across CPU, memory bandwidth, and random I/O. `lab-core01` was therefore migrated to `pve02` and remains there as its preferred placement.

## Power Measurement

Whole-system power consumption has **not** been measured.

A suitable power meter was not available during testing, so no idle or load wattage should be inferred from the CPU TDP, NVMe power states, or benchmark results.

Power efficiency remains a future measurement. The goal is to compare the two G5 systems under idle and representative homelab workloads rather than rely on component TDP values.

## Future Tests

The following tests remain useful for completing the pve/pve02 comparison:

* Whole-system idle power measurement
* Whole-system load power measurement
* CPU temperature under sustained load
* CPU frequency and thermal behavior under sustained load
* 32 GB dual-channel memory comparison after the second 16 GB DIMM is installed
* 1 TB Optimus 5001 NVMe benchmark after installation

The existing benchmark parameters should be reused wherever practical so future results remain directly comparable.
