# G5 Performance

Performance baseline for the HP EliteDesk 800 G5 Mini running Proxmox VE.

## System

| Component          | Specification             |
| ------------------ | ------------------------- |
| System             | HP EliteDesk 800 G5 Mini  |
| CPU                | Intel Core i5-9500T       |
| CPU Cores          | 6                         |
| CPU Threads        | 6                         |
| CPU Base Frequency | 2.2 GHz                   |
| CPU Max Turbo      | 3.7 GHz                   |
| L3 Cache           | 9 MB                      |
| RAM                | 16 GB DDR4                |
| RAM Configuration  | 1 × 16 GB SODIMM          |
| RAM Speed          | 2667 MT/s                 |
| RAM Slots          | 2                         |
| RAM Maximum        | 32 GB                     |
| Storage            | Toshiba KXG50ZNV256G NVMe |
| Storage Capacity   | 256 GB                    |
| Proxmox VE         | 9.2.2                     |
| Kernel             | 7.0.2-6-pve               |
| Network            | 1 Gb Ethernet             |
| Network Interface  | `nic0`                    |
| Proxmox Bridge     | `vmbr0`                   |

## Network Configuration

The G5 is connected to the homelab network using its physical Ethernet interface.

| Property         | Value               |
| ---------------- | ------------------- |
| Interface        | `nic0`              |
| MAC Address      | `04:0e:3c:a8:ff:5b` |
| Proxmox Bridge   | `vmbr0`             |
| IP Address       | `192.168.12.247/24` |
| Link Speed       | 1000 Mb/s           |
| Duplex           | Full                |
| Auto Negotiation | Enabled             |
| Link Status      | Up                  |

The physical network path is:

```text
G5
 │
 │ 1 Gb Ethernet
 ▼
NETGEAR GS108E
 │
 ├── Zyxel NAS
 ├── Proxmox guests
 └── Other network devices
```

A 1 Gb Ethernet connection provides a theoretical maximum of 125 MB/s before protocol and filesystem overhead.

## Storage Configuration

The Proxmox installation resides on the Toshiba NVMe drive.

```text
/dev/nvme0n1
└── 256 GB Toshiba KXG50ZNV256G
    ├── EFI
    └── LVM
        ├── pve-swap       8 GB
        ├── pve-root      69.4 GB
        └── pve-data     141.2 GB
```

The `pve-data` LVM-thin pool currently contains virtual disks for the Proxmox workloads.

### NVMe Information

| Property  | Value                |
| --------- | -------------------- |
| Model     | KXG50ZNV256G TOSHIBA |
| Serial    | `48KF71X0F6FS`       |
| Firmware  | `AAHA4102`           |
| Namespace | 1                    |
| Capacity  | 256.06 GB            |

Storage benchmarks will be performed using controlled test files rather than destructive whole-device testing because this is the active Proxmox system disk.

## CPU Benchmark

### Method

CPU performance was measured using `sysbench`.

Parameters:

```text
Benchmark: CPU
Prime limit: 10,000
Duration: 60 seconds
```

### Results

| Threads | Environment      |   Events/sec | Avg Latency |
| ------: | ---------------- | -----------: | ----------: |
|       1 | VMs/LXCs running |     1,170.16 |     0.85 ms |
|       1 | VMs/LXCs stopped | **1,221.43** | **0.82 ms** |
|       6 | VMs/LXCs stopped | **6,734.57** | **0.89 ms** |

### Baseline

The official CPU baseline uses the results obtained with all VMs and LXCs stopped:

* **Single-thread:** 1,221.43 events/sec
* **Six-thread:** 6,734.57 events/sec

The initial single-thread test was performed while Proxmox guests were running and therefore is retained for comparison but is not used as the primary baseline.

### Observations

Stopping the Proxmox guests increased single-thread performance from 1,170.16 to 1,221.43 events/sec, an improvement of approximately 4.4%.

The six-thread result reached approximately 5.5× the single-thread performance. Perfect linear scaling is not expected because the benchmark is affected by operating-system scheduling, shared CPU resources, cache behavior, and memory subsystem performance.

## Memory Benchmark

The system currently contains a single 16 GB DDR4-2667 SODIMM.

### Method

Memory performance was measured using `sysbench`.

Parameters:

```text
Benchmark: memory
Block size: 1 MiB
Total transfer: 10 GiB
Threads: 1
Scope: global
```

### Results

| Operation |            Throughput | Average Latency |
| --------- | --------------------: | --------------: |
| Read      | **25,645.48 MiB/sec** |         0.04 ms |
| Write     | **21,665.73 MiB/sec** |         0.05 ms |

### Baseline

The system is currently configured with a single 16 GB DDR4-2667 SODIMM.

The current memory baseline is:

* **Read:** 25,645.48 MiB/sec
* **Write:** 21,665.73 MiB/sec

### Observations

The G5 is currently operating with one memory module. The system has two physical memory slots, with one slot currently unpopulated.

A future upgrade to 32 GB using two 16 GB modules will provide an opportunity to measure the effect of dual-channel memory on memory throughput.

The same benchmark parameters should be used for future comparisons.

A future upgrade to 32 GB using two 16 GB modules will provide an opportunity to compare both capacity and memory-channel performance.

## NVMe Benchmark

The G5 uses a 256 GB Toshiba KXG50ZNV256G XG5 OEM NVMe SSD. The drive is an older component with significant prior operating time, but testing indicates that it remains healthy and performs adequately for the current Proxmox workload.

### Drive

| Property   |                   Result |
| ---------- | -----------------------: |
| Model      | Toshiba KXG50ZNV256G XG5 |
| Capacity   |                   256 GB |
| Firmware   |                 AAHA4102 |
| Interface  |              PCIe 3.0 x4 |
| PCIe Link  |                8 GT/s x4 |
| Controller |         Toshiba XG5 NVMe |
| Driver     |                   `nvme` |

### NVMe Health Baseline

Health information was captured using `nvme smart-log`.

| Metric            |      Result |
| ----------------- | ----------: |
| Temperature       | 89°F / 32°C |
| Available spare   |        100% |
| Percentage used   |         47% |
| Data read         |    37.98 TB |
| Data written      |    56.40 TB |
| Power-on hours    |      50,332 |
| Power cycles      |         336 |
| Unsafe shutdowns  |         130 |
| Media errors      |           0 |
| Error log entries |           0 |
| Critical warnings |           0 |

The drive has approximately 50,332 power-on hours, equivalent to roughly 5.7 years of continuous powered-on operation. The 47% percentage-used value also indicates substantial prior use.

Despite its age and usage, the current health indicators are good. The drive reports no media errors, no error-log entries, no critical warnings, and 100% available spare capacity. Temperature during testing was only 32°C.

The drive should therefore be considered **aging but currently healthy**, rather than failing or unreliable.

### PCIe Configuration

The NVMe controller is operating at:

* PCIe 3.0 / 8 GT/s
* PCIe x4
* Current link: 8 GT/s x4
* ASPM L1 enabled

The negotiated PCIe link confirms that the SSD is operating at the expected PCIe generation and lane width. There is no evidence that the NVMe performance is being limited by a reduced PCIe link.

### Benchmark Methodology

Storage testing was performed with `fio`.

The raw-device sequential read test was nondestructive and used:

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

Additional testing was performed against an 8 GB LVM-thin test volume:

```text
/dev/pve/fio-test
```

The test volume was used to evaluate the behavior of the Proxmox LVM-thin storage layer without testing directly against active VM disks.

### Performance Results

| Test                       |                     Result |
| -------------------------- | -------------------------: |
| Raw sequential read        |            **2,064 MiB/s** |
| LVM-thin sequential read   |           **3,964 MiB/s*** |
| Sequential write, LVM-thin |              **328 MiB/s** |
| 4K random read             |  **155K IOPS / 604 MiB/s** |
| 4K random write            | **62.2K IOPS / 243 MiB/s** |

* The LVM-thin sequential-read result is not representative of the physical SSD's sustained sequential-read capability. The result was influenced by the logical storage layer and caching behavior. It is retained as an illustration of the difference between testing a logical storage device and testing the underlying physical device.

The raw-device sequential-read result of approximately **2.06 GiB/s** is therefore the primary physical NVMe baseline.

### Storage Stack Comparison

Testing demonstrated why storage benchmarks must identify the layer being measured.

| Test                                   |     Throughput | What it measures                                            |
| -------------------------------------- | -------------: | ----------------------------------------------------------- |
| LVM-thin read, unpopulated test volume |     12.6 GiB/s | Logical/storage-layer behavior; not physical SSD throughput |
| LVM-thin read, populated test volume   |     3.96 GiB/s | Logical storage path with caching effects                   |
| Raw NVMe read                          | **2.06 GiB/s** | **Physical NVMe baseline**                                  |
| LVM-thin sequential write              |  **328 MiB/s** | Write behavior through the Proxmox LVM-thin layer           |

The very high logical-device read results should not be interpreted as evidence that the Toshiba SSD can sustain those speeds. The raw-device test provides the cleanest measurement of the physical drive.

### VM-Relevant Random I/O

The 4K random tests are particularly relevant to a virtualization host because VM workloads frequently consist of many small, non-sequential I/O operations.

The G5 achieved:

* **155K 4K random-read IOPS**
* **62.2K 4K random-write IOPS**

These results indicate that the aging Toshiba SSD still provides strong small-block random I/O performance for the current homelab workload.

The random-write test also showed considerably higher latency variability than the random-read test. This is expected to some degree from SSD flash management, garbage collection, and the characteristics of the aging drive.

### Assessment

The Toshiba XG5 is an older 256 GB OEM NVMe SSD with approximately 50,000 power-on hours and 47% reported endurance consumed. It should therefore be regarded as an aging component.

However, the performance and health data do not currently indicate a failing drive.

The drive is operating correctly on a PCIe 3.0 x4 link and delivers approximately **2.06 GiB/s of raw sequential-read throughput**, along with approximately **155K 4K random-read IOPS and 62K random-write IOPS**.

For the current Proxmox homelab, storage performance is adequate. The more immediate limitation is **capacity**, rather than raw performance.

A future NVMe upgrade remains worthwhile because a newer, larger SSD would provide additional capacity, newer NAND/controller technology, and a fresh endurance rating. The current drive can continue serving as the baseline for comparison.

Future NVMe health checks should monitor:

* Percentage used
* Available spare
* Media errors
* Error-log entries
* Unsafe shutdowns
* Temperature
* Power-on hours


## Network Benchmark

**Pending**

Network throughput will be measured using `iperf3`.

The goal is to establish actual network throughput before testing NAS/NFS performance.

## NAS / NFS Benchmark

**Pending**

The Zyxel NAS will be benchmarked separately over NFS.

Results will be compared against the G5's local NVMe performance to determine whether storage performance is being constrained by:

* NVMe performance
* Network throughput
* NFS
* NAS storage subsystem
* NAS CPU or other hardware limitations

## Future Comparisons

This baseline can be repeated after hardware or configuration changes.

Potential future comparisons:

* 16 GB vs. 32 GB RAM
* Single-channel vs. dual-channel memory
* Current NVMe vs. upgraded NVMe
* Local storage vs. Zyxel NFS
* Current Zyxel NAS vs. future NAS
* 1 Gb Ethernet vs. future network upgrade
* Idle system vs. system under virtualization workload

## Benchmarking Tools

The following tools are installed on the Proxmox host:

```text
sysbench
fio
iperf3
nvme-cli
smartmontools
lm-sensors
stress-ng
```

## Baseline Date

**August 8, 2026**

Additional benchmark results will be added as testing progresses.
