# pve01

## Purpose

Primary Proxmox VE node in the `nexus` cluster.

The node provides compute and storage capacity for the home lab and serves as the first node in the current two node Proxmox cluster.

## Hardware

System:
- HP EliteDesk 800 G5 Desktop Mini

CPU:
- Intel Core i5-9500T
- 6 cores
- 6 threads
- 2.20 GHz base frequency

Memory:
- 16 GiB DDR4 installed
- Approximately 15 GiB reported by the operating system

Storage:
- KXG50ZNV256G TOS NVMe
- 238.5 GiB detected capacity
- 1 GiB EFI partition
- 237.5 GiB LVM physical volume

Firmware:
- HP R21 Ver. 02.26.00
- Firmware date: 2026-05-05

## Proxmox

Hostname:
- `pve`

Cluster:
- `nexus`

Proxmox VE:
- 9.2.0
- pve-manager 9.2.2

Kernel:
- 7.0.2-6-pve

## Storage Configuration

The 256 GB NVMe uses the standard Proxmox LVM layout with local guest storage:

```text
/dev/nvme0n1
└── 238.5 GiB NVMe
    ├── EFI                 1 GiB
    └── LVM                237.5 GiB
        ├── pve-swap        8 GiB
        ├── pve-root       69.4 GiB ext4
        └── pve-data      141.2 GiB LVM-thin
```

The LVM-thin pool currently contains virtual disks for the lab workloads, including VM IDs 101, 102, 200, and 201.

## Network Configuration

The Proxmox host participates in the internal lab network and uses the cluster network for Proxmox communication.

Internal addressing and other environment-specific network identifiers are intentionally omitted from public documentation.

## Cluster Membership

`pve01` is node 1 of the `nexus` Proxmox cluster.

Current membership:

```text
nexus
├── pve
└── pve02
```

Current cluster status:
- 2 nodes
- 2 expected votes
- 2 total votes
- Quorate: yes
- Corosync transport: knet
- Secure authentication: enabled

## Current Host State

At the time of documentation on 2026-08-22:

```text
Memory:
15 GiB total
5.6 GiB used
5.7 GiB free
9.8 GiB available
8 GiB swap
```

Swap usage was approximately 749 MiB at the time of the readout.

## Workload Placement

`pve01` is the original Proxmox node used for the lab's initial workload placement and remains an active member of the `nexus` cluster.

A controlled workload comparison between `pve01` and `pve02` has established `pve02` as the preferred host for `lab-core01` based on CPU, memory, and random storage performance. The authoritative benchmark results and comparison are documented in `systems/pve02.md`.

## Public Documentation Policy

Actual internal IP addresses, MAC addresses, serial numbers, internal DNS names, and other environment-specific network identifiers are intentionally omitted from this public repository.
