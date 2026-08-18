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

- 32 GB RAM
- 1 TB NVMe under consideration / planned purchase

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
