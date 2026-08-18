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
- `pve02.lab.tearstone.com`

IP address:
- `192.168.12.248/24`

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
       192.168.12.248/24
```

The node uses wired Ethernet for Proxmox cluster communication. The installed Wi-Fi adapter is not used for cluster traffic.

## Cluster Membership

`pve02` is node 2 of the `nexus` Proxmox cluster.

Current membership:

```text
nexus
├── pve       192.168.12.247
└── pve02     192.168.12.248
```

The cluster is currently quorate with two expected votes and two total votes.

## Benchmark Plan

`pve02` will be benchmarked against the existing `pve` node. The two systems use the same HP EliteDesk 800 G5 Mini platform and 6-core/6-thread Coffee Lake family but different CPU variants:

- `pve`: Intel Core i5-9500T
- `pve02`: Intel Core i5-9500

`lab-core01` is the first candidate workload for a controlled placement comparison. The VM will be evaluated on both nodes using consistent CPU, memory, storage, and application workload measurements.

Power measurements are also planned to determine whether the standard i5-9500 provides a meaningful performance advantage without compromising the low-power objective of the homelab.
