# G5 Performance: pve01

Performance baseline for the HP EliteDesk 800 G5 Mini running Proxmox VE on `pve01`.

## System

| Component          | Specification             |
| ------------------ | ------------------------- |
| System             | HP EliteDesk 800 G5 Mini  |
| Proxmox Node       | `pve01`                   |
| IP Address         | `192.168.12.247/24`       |
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

`pve01` is the first physical node in the two-node `nexus` Proxmox cluster. It uses the 35 W-class Intel Core i5-9500T processor.

## Network Configuration

The G5 is connected to the homelab network using its physical Ethernet interface.

| Property         | Value               |
| ---------------- | ------------------- |
| Interface        | `nic0`              |
| MAC Address      | `04:0e:3c:a8:ff:5b` |
| Proxmox Bridge   | `vmbr0`              |
| IP Address       | `192.168.12.247/24` |
| Link Speed       | 1000 Mb/s           |
| Duplex           | Full                |
| Auto Negotiation | Enabled             |
| Link Status      | Up                  |

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

Network throughput was measured with `iperf3` between `pve01` (`192.168.12.247`) and `pve02` (`192.168.12.248`).

| Direction | Throughput | Retransmits |
| --------- | ---------: | ----------: |
| `pve01` → `pve02` | **934 Mbits/sec** | 0 |
| `pve02` → `pve01` | **935 Mbits/sec** | 0 |

The measured throughput is approximately 93.4% of the nominal 1 Gb Ethernet line rate and is consistent with a healthy 1 GbE connection after protocol overhead.

The network path between the two Proxmox nodes is therefore not showing an obvious performance limitation.

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

Storage benchmarks are performed using controlled test files or test volumes rather than destructive whole-device testing because this is the active Proxmox system disk.

## CPU Benchmark

### Host CPU Baseline

CPU performance was measured using `sysbench` directly on the Proxmox host.

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

The official host CPU baseline uses the results obtained with all VMs and LXCs stopped:

* **Single-thread:** 1,221.43 events/sec
* **Six-thread:** 6,734.57 events/sec

The initial single-thread test was performed while Proxmox guests were running and therefore is retained for comparison but is not used as the primary host baseline.

### Observations

Stopping the Proxmox guests increased single-thread performance from 1,170.16 to 1,221.43 events/sec, an improvement of approximately 4.4%.

The six-thread result reached approximately 5.5× the single-thread performance. Perfect linear scaling is not expected because the benchmark is affected by operating-system scheduling, shared CPU resources, cache behavior, and memory subsystem performance.

## Memory Benchmark

The system currently contains a single 16 GB DDR4-2667 SODIMM.

### Host Memory Baseline

Memory performance was measured using `sysbench` directly on the Proxmox host.

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

The current host memory baseline is:

* **Read:** 25,645.48 MiB/sec
* **Write:** 21,665.73 MiB/sec

The same benchmark parameters should be used for future comparisons.

A future upgrade to 32 GB using two 16 GB modules will provide an opportunity to compare both capacity and memory-channel performance.

## lab-core01 Workload Placement Benchmark

The workload-placement benchmark measures the same `lab-core01` VM on `pve01` and `pve02`. This is the appropriate comparison for deciding which physical node should host the VM.

The VM was configured with 4 vCPUs and 12 GB RAM on both nodes.

### Controlled Results

| Test | `lab-core01` on `pve01` | `lab-core01` on `pve02` | Change |
| ---- | ----------------------: | -----------------------: | -----: |
| CPU 1T | 1,120.76 events/s | **1,436.78 events/s** | **+28.2%** |
| CPU 4T | 4,427.28 events/s | **5,559.86 events/s** | **+25.6%** |
| Memory read | 23,835.9 MiB/s | **29,181.1 MiB/s** | **+22.4%** |
| Memory write | 20,334.1 MiB/s | **25,108.7 MiB/s** | **+23.5%** |
| 4K random read | 232K IOPS | **300K IOPS** | **+29.3%** |
| 4K random write | 222K IOPS | **286K IOPS** | **+29.0%** |

The storage values in this table are the controlled cache-cleared VM filesystem tests. They are separate from the host raw-NVMe and LVM-thin test-volume results documented below.

The comparison showed a consistent performance advantage for `pve02` across CPU, memory bandwidth, and random I/O. `lab-core01` was therefore migrated to `pve02` and remains there as its preferred placement.

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

## NAS / NFS Benchmark

The Zyxel NAS326 is being used as shared network storage for the homelab. The NFS share is mounted on the Debian VM `lab-core01` at:

```text
/mnt/nas
```

Benchmark files are stored under:

```text
/mnt/nas/benchmark
```

Testing was performed with `fio` from `lab-core01` against the NFS-mounted storage.

## Test Configuration

| Test             | Block Size | I/O Depth | Jobs | Test Size |   Runtime |
| ---------------- | ---------: | --------: | ---: | --------: | --------: |
| Sequential Write |      1 MiB |        16 |    1 |     4 GiB | Full test |
| Sequential Read  |      1 MiB |        16 |    1 |     4 GiB | Full test |
| Random Read      |      4 KiB |        32 |    1 |     4 GiB |    60 sec |
| Random Write     |      4 KiB |        32 |    1 |     4 GiB |    60 sec |

All sequential tests used `--direct=1`. The random I/O tests used the default buffered I/O configuration.

---

## NFS Performance Results

| Workload                    |      IOPS |      Bandwidth | Average Latency | P99 Latency |
| --------------------------- | --------: | -------------: | --------------: | ----------: |
| **Sequential Read, 1 MiB**  |    **97** | **97.7 MiB/s** |        163.6 ms |      359 ms |
| **Sequential Write, 1 MiB** |    **17** | **18.0 MiB/s** |        890.8 ms |    1.20 sec |
| **Random Read, 4 KiB**      |   **118** |  **475 KiB/s** |        268.4 ms |      443 ms |
| **Random Write, 4 KiB**     | **10.2K** | **39.9 MiB/s** |         3.13 ms |     19.8 ms |

### Sequential Read

The NFS share achieved approximately **97.7 MiB/s** of sequential read throughput using 1 MiB blocks and a queue depth of 16.

This is close to the practical throughput expected from a 1 GbE network connection after accounting for protocol overhead and storage performance.

Read latency averaged approximately **164 ms**, with a P99 latency of approximately **359 ms**.

### Sequential Write

Sequential write performance was significantly lower than sequential read performance.

The test achieved approximately:

* **18.0 MiB/s**
* **17 IOPS**
* **891 ms average latency**
* **1.20 sec P99 latency**

The write result strongly suggests that the NAS storage subsystem is a significant bottleneck for sustained sequential writes. At 18 MiB/s, the workload is well below the theoretical capacity of the 1 GbE link. The independent iperf3 benchmark confirms that the network itself is capable of approximately 934 to 935 Mbits/sec between the Proxmox nodes.

### 4K Random Read

The 4K random-read test produced only:

* **118 IOPS**
* **475 KiB/s**
* **268 ms average latency**
* **443 ms P99 latency**

This is a significant reduction in performance compared with sequential reads.

The result demonstrates that the NAS is poorly suited to workloads requiring large numbers of small random reads.

### 4K Random Write

The 4K random-write test produced:

* **10.2K IOPS**
* **39.9 MiB/s**
* **3.13 ms average latency**
* **176 µs median latency**
* **19.8 ms P99 latency**
* **204 ms P99.9 latency**
* **2.67 sec maximum observed latency**

The random-write result is unusual compared with the random-read result. The high IOPS and relatively low median latency suggest that caching and write behavior in the NAS/NFS stack are substantially affecting the result.

The large latency tail is also significant. Although the median operation completed in approximately 176 µs, the P99 latency increased to approximately 20 ms and the P99.9 latency exceeded 200 ms.

---

## NFS Performance Summary

The benchmark demonstrates a substantial difference between sequential and random workloads.

**Sequential read performance is the strongest characteristic of the NAS**, reaching approximately 98 MiB/s and approaching the practical limit of a 1 GbE network connection.

**Sequential write performance is considerably weaker**, reaching 18 MiB/s with nearly 900 ms average I/O latency.

**4K random read performance is particularly poor**, at approximately 118 IOPS and 475 KiB/s.

The 4K random-write result is considerably better at approximately 10.2K IOPS and 39.9 MiB/s, although the large latency tail indicates inconsistent completion times under load.

### Suitability for Homelab Workloads

Based on these results, the NAS is well suited for:

* General file storage
* Backups
* ISO images
* Installation media
* Large sequential reads
* Archive storage
* Infrequently accessed data

The NAS is less suitable for:

* VM boot disks
* Databases
* High-I/O virtual machines
* Write-intensive Docker containers
* Applications requiring consistently low storage latency
* Workloads dominated by small random reads

For performance-sensitive VM and container workloads, **local NVMe storage on the EliteDesk G5 should be preferred**.

The NFS share remains valuable as shared capacity and backup storage, but the benchmark results indicate that it should not be considered a replacement for local SSD storage for latency-sensitive workloads.

---

## Benchmark Conclusion

The NFS benchmark confirms that the Zyxel NAS326 is primarily a **capacity and shared-storage resource rather than a high-performance storage subsystem**.

The approximately **97.7 MiB/s sequential read result** demonstrates that the network path can provide useful throughput for large-file operations. However, the **18 MiB/s sequential write**, **118 IOPS 4K random read**, and high random-read latency demonstrate significant limitations for storage-intensive workloads.

For the homelab architecture, the current results support the following storage strategy:

| Storage             | Recommended Workloads                                          |
| ------------------- | -------------------------------------------------------------- |
| **Local NVMe**      | VM disks, containers, databases, active application data       |
| **Zyxel NAS / NFS** | Backups, shared files, ISO images, archives, secondary storage |

Results will be compared against the G5's local NVMe performance to determine whether storage performance is being constrained by:

* NVMe performance
* Network throughput
* NFS
* NAS storage subsystem
* NAS CPU or other hardware limitations

## Power Measurement

Whole-system power consumption has **not** been measured.

A suitable power meter was not available during testing, so no idle or load wattage should be inferred from the CPU TDP, NVMe power states, or benchmark results.

Power efficiency remains a future measurement. The goal is to compare the two G5 systems under idle and representative homelab workloads rather than rely on component TDP values.

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
* Whole-system power consumption on `pve01` vs. `pve02`

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

**Last updated:** August 19, 2026

Additional benchmark results will be added as testing progresses.
