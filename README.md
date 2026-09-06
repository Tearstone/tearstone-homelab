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
  * 32 GB RAM
  * Proxmox VE
* MacBook Pro 2020

### Virtualization

* Proxmox VE 9.x
* Two-node Proxmox cluster: `nexus`
* `lab-core01` Docker application VM, currently hosted on `pve02`
* Debian Linux VM
* Kali Linux VM
* Qualys Scanner Appliance
* `infra-homepage01` Debian 13 LXC providing the Homepage dashboard
* `infra-uptime01` Debian 13 LXC providing Uptime Kuma monitoring

### Storage

* Zyxel NAS326
* 256 GB system NVMe on each G5 node
* 1 TB SanDisk Optimus 5001 NVMe on `pve02`
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
* Uptime Kuma

### Security

* Qualys Scanner Appliance
* Kali Linux

### Applications

* Docker
* Immich
* Homepage
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

The addition of the 1 TB SanDisk Optimus 5001 NVMe to `pve02` established a new dedicated high-performance storage tier. Raw-device fio testing using `io_uring`, queue depth 32, measured approximately 3.4 GiB/s sequential read, 3.2 GiB/s sequential write, 355K 4K random-read IOPS, 102K 4K random-write IOPS, and 63K read / 27K write IOPS in a 70/30 random mixed workload.

Both G5 nodes now have 32 GB of RAM. `pve01` and `pve02` each use two 16 GB SODIMMs. The pve02 upgrade from 16 GB to 32 GB established a new memory baseline of 29,995.98 MiB/sec read and 25,693.97 MiB/sec write with one thread, with additional four-thread results of 112,454.39 MiB/sec read and 85,633.22 MiB/sec write. The primary benefit of the upgrade is increased memory capacity and workload headroom rather than a material change in single-thread memory bandwidth.

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

The immediate infrastructure direction is to continue optimizing the two-node G5 Proxmox cluster rather than moving to larger enterprise servers. The entire lab setup consumes very little power and produces no fan noise. 

## Public Documentation Policy

This repository is public. Documentation intentionally describes architecture, configuration concepts, and benchmark results without publishing unnecessary internal network identifiers such as private IP addresses, MAC addresses, or internal DNS details.

Actual addressing and other environment-specific operational details are maintained separately from the public repository.

## Project Status

**Actively building, testing, measuring, and documenting.**

The lab is now a two-node Proxmox environment with shared NAS storage, dedicated local NVMe storage on `pve02`, containerized applications, centralized metrics monitoring, service availability monitoring, and an established performance baseline. Dedicated Debian LXCs provide a native Homepage dashboard and Uptime Kuma monitoring for navigation, service status, and alerting. The project will continue to evolve as additional compute, storage, networking, automation, and security capabilities are added.
