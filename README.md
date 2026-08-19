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
  * 16 GB RAM
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
* Local NVMe storage on both G5 nodes
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

Planned work includes:

* Install the planned 1 TB Optimus 5001 NVMe in the appropriate G5 node
* Evaluate a second 16 GB DIMM and 32 GB dual-channel configuration
* Measure whole-system idle and load power
* Continue storage and network performance testing
* Implement automated Proxmox backups and backup verification
* Expand monitoring, DNS, TLS, and network services
* Explore VLANs, segmentation, automation, and additional security infrastructure

## Public Documentation Policy

This repository is public. Documentation intentionally describes architecture, configuration concepts, and benchmark results without publishing unnecessary internal network identifiers such as private IP addresses, MAC addresses, or internal DNS details.

Actual addressing and other environment-specific operational details are maintained separately from the public repository.

## Project Status

**Actively building, testing, measuring, and documenting.**

The lab is now a two-node Proxmox environment with shared NAS storage, containerized applications, centralized monitoring, and an established performance baseline. The project will continue to evolve as additional compute, storage, networking, automation, and security capabilities are added.
