# Hardware

## Compute

### HP EliteDesk 800 G5 Mini — pve

CPU
Intel Core i5-9500T
6 cores / 6 threads
2.2 GHz base / 3.7 GHz turbo

Memory
16 GB DDR4

Storage
256 GB NVMe

Network
Intel I219-LM 1 Gb Ethernet

Hostname
`pve.lab.tearstone.com`

IP Address
`192.168.12.247/24`

Future Upgrades

- 32 GB RAM using a second 16 GB DIMM
- 1 TB Optimus 5001 NVMe as an additional drive

### HP EliteDesk 800 G5 Mini — pve02

CPU
Intel Core i5-9500
6 cores / 6 threads
3.0 GHz base frequency

Memory
16 GB DDR4
2 × 8 GB

Storage
256 GB WDC/SanDisk PC SN730 NVMe

Network
Intel I219-LM 1 Gb Ethernet

Wireless
Intel Wi-Fi 6 AX200

Graphics
Intel UHD Graphics 630

Hostname
`pve02.lab.tearstone.com`

IP Address
`192.168.12.248/24`

Proxmox Storage Layout

```text
238.5 GiB NVMe
└── LVM
    ├── pve-swap        4 GB
    ├── pve-root       16 GB ext4
    └── pve-data      ~197.4 GB LVM-thin
```

The installation intentionally reserves approximately 16 GB of free LVM space for future flexibility.

## Proxmox Cluster

Cluster name
`nexus`

Nodes

- `pve` — 192.168.12.247
- `pve02` — 192.168.12.248

The two nodes are HP EliteDesk 800 G5 Desktop Mini systems with the same 6-core/6-thread CPU family but different power/performance variants. `pve` uses the lower-power i5-9500T, while `pve02` uses the standard i5-9500.

## Workload Placement and A/B Testing

`lab-core01` was used as the first controlled workload placement comparison between the two Proxmox nodes. The VM has 4 vCPUs, 12 GB RAM, and an 80 GB virtual disk using `virtio-scsi-single` with I/O thread enabled.

The same sysbench and fio workloads were run on both hosts. The final storage comparison used `sync` and `drop_caches` before each test to avoid using the preliminary cached result.

| Test | lab-core01 on pve | lab-core01 on pve02 | Improvement |
| ---- | -----------------: | ------------------: | ----------: |
| CPU, 1 thread | 1,120.76 events/s | 1,436.78 events/s | +28.2% |
| CPU, 4 threads | 4,427.28 events/s | 5,559.86 events/s | +25.6% |
| Memory read | 23,835.9 MiB/s | 29,181.1 MiB/s | +22.4% |
| Memory write | 20,334.1 MiB/s | 25,108.7 MiB/s | +23.5% |
| 4K random read | 232K IOPS | 300K IOPS | +29.3% |
| 4K random write | 222K IOPS | 286K IOPS | +29.0% |

The preliminary pve02 storage run produced approximately 462K read IOPS and 455K write IOPS, but those results were not retained as the official comparison because a subsequent cache-cleared test produced approximately 300K/286K IOPS. The cache-cleared results are the authoritative A/B measurements.

The results show that `pve02` is consistently faster for the tested `lab-core01` workload across CPU, memory bandwidth, and random storage I/O. `lab-core01` was therefore migrated to `pve02` and will remain there as the preferred host.

Power consumption has not yet been measured because a suitable power meter is not currently available. The low-power objective remains a design consideration for future measurement.

## Why this hardware?

After comparing with Raspberry Pi and HP EliteDesk Minis, I selected the EliteDesk because it provides:

- Proxmox compatibility
- Low power consumption suitable for continuous operation
- Whisper quiet operation
- Small physical footprint
- Affordable refurbished pricing
- Case and storage included
- Enough CPU and memory capacity for a practical virtualization lab

The compact EliteDesk Mini platform was selected instead of larger enterprise servers because the homelab prioritizes low power consumption, low noise, and minimal rack/desk space while still providing x86 virtualization capabilities.

The two-node `nexus` cluster provides a platform for testing Proxmox clustering, migration, workload placement, performance comparison, and future high-availability concepts.
