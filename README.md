# Tearstone Homelab

A documented home lab focused on Linux, virtualization, networking, monitoring, automation, storage, and cybersecurity.

The lab is built as a practical learning environment where new technologies are deployed, tested, measured, documented, and improved over time.

## Goals

* Learn by building
* Document everything
* Automate repetitive tasks
* Simulate enterprise environments
* Apply cybersecurity best practices
* Measure system performance
* Share working configurations and lessons learned

## Current Environment

### Compute

* HP EliteDesk 800 G5 Mini `pve01`
  * Intel Core i5-9500T
  * 32 GB RAM
  * Proxmox VE
* HP EliteDesk 800 G5 Mini `pve02`
  * Intel Core i5-9500
  * 16 GB RAM
  * Proxmox VE
* MacBook Pro 2020

### Virtualization

* Proxmox VE 9.x
* Two-node Proxmox cluster: `nexus`
* `lab-core01` Docker application VM, currently hosted on `pve02`
* Debian Linux VM
* Kali Linux VM
* Qualys Scanner Appliance

### Storage

* Zyxel NAS326
* 256 GB system NVMe on each G5 node
* 1 TB SanDisk Optimus 5100 NVMe on `pve02`
* Dedicated Proxmox `nvme-lvm` LVM-thin storage on `pve02`
* NFS shared storage
* NAS-backed Immich media storage

### Networking

* NETGEAR GS108E managed switch
* 1 Gb Ethernet
* Proxmox `vmbr0`
* Node-to-node `iperf3` baseline of approximately 934–935 Mbit/sec

### Monitoring

* Prometheus
* Node Exporter
* Grafana

### Security

* Qualys Scanner Appliance
* Kali Linux

### Applications

* Docker
* Immich
* WordPress applications on `prod-web01`

## Documentation

| Area | Documentation |
| ---- | ------------- |
| Architecture | [Architecture](docs/architecture.md) |
| Hardware | [Hardware](docs/hardware.md) |
| Monitoring | [Monitoring](docs/monitoring.md) |
| Systems | [Systems](systems/) |
| Services | [Services](services/) |
| Storage | [Storage](storage/) |
| Applications | [Applications](applications/) |
| Benchmarking | [Benchmarking](benchmarking/) |
| Roadmap | [Roadmap](roadmap.md) |
| Changelog | [Changelog](changelog.md) |

## Performance Baseline

Both HP EliteDesk 800 G5 Mini systems have been benchmarked as Proxmox nodes, and `lab-core01` has been tested on both nodes using controlled A/B workloads.

The current results show `pve02` providing a consistent performance advantage for `lab-core01`, with approximately 22–29% improvement across the tested CPU, memory, and 4K random I/O workloads. `lab-core01` has therefore been migrated to `pve02` as its preferred placement.

The addition of the 1 TB SanDisk Optimus 5100 NVMe to `pve02` established a new dedicated high-performance storage tier. Raw-device fio testing using `io_uring`, queue depth 32, measured approximately 3.4 GiB/s sequential read, 3.2 GiB/s sequential write, 355K 4K random-read IOPS, 102K 4K random-write IOPS, and 63K read / 27K write IOPS in a 70/30 random mixed workload.

`pve01` was upgraded from 16 GB to 32 GB DDR4 using a second 16 GB SODIMM. The post-upgrade memory benchmark is now the current host baseline, while the original 16 GB results are retained for comparison in the node-specific benchmark record.

Benchmarks cover:

* CPU
* Memory
* NVMe storage
* LVM-thin storage
* 4K random I/O
* 1 Gb Ethernet
* NAS/NFS storage
* `lab-core01` workload performance across both G5 nodes

See the [Benchmarking](benchmarking/) documentation for node-specific results and methodology.

Whole-system power consumption has not yet been measured.

## Current Direction

The immediate infrastructure direction is to continue optimizing the two-node G5 Proxmox cluster rather than moving to larger enterprise servers.

Current work includes:

* `pve01` now has 32 GB DDR4 in a 2 × 16 GB dual-channel configuration
* `pve02` has a dedicated 1 TB SanDisk Optimus 5100 NVMe storage tier exposed to Proxmox as `nvme-lvm`
* `lab-core01` VM 100's 80 GB system disk has been migrated from `local-lvm` to `nvme-lvm` while the VM remained running
* Maintain NAS-backed Proxmox backups and verify backup availability
* Measure whole-system idle and load power
* Continue storage and network performance testing
* Expand monitoring, DNS, TLS, and network services
* Explore VLANs, segmentation, automation, and additional security infrastructure

## Public Documentation Policy

This repository is public. Documentation intentionally describes architecture, configuration concepts, and benchmark results without publishing unnecessary internal network identifiers such as private IP addresses, MAC addresses, or internal DNS details.

Actual addressing and other environment-specific operational details are maintained separately from the public repository.

## Project Status

**Actively building, testing, measuring, and documenting.**

The lab is now a two-node Proxmox environment with shared NAS storage, dedicated local NVMe storage on `pve02`, containerized applications, centralized monitoring, and an established performance baseline. The project will continue to evolve as additional compute, storage, networking, automation, and security capabilities are added.
