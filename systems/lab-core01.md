# lab-core01

## Purpose

Primary Linux application server running on Proxmox.

Current roles:

- Docker host
- Immich application server
- Future home lab application workloads

## Virtual Machine Details

Hypervisor:
- Proxmox VE

Current host:
- `pve02`

Operating System:
- Debian GNU/Linux 13 (Trixie)

Architecture:
- amd64

Hostname:
- lab-core01

Resources:

CPU:
- 1 socket
- 4 vCPU

Memory:
- 12 GB configured
- 2 GB swap

Disk:
- 80 GB virtual disk

Network:
- Internal network address intentionally omitted from public documentation

## Installed Software

### Monitoring

Node Exporter

Purpose:
Provides Linux system metrics to Prometheus.

Port:
9100

### Container Platform

Docker Engine

Version:
- 29.7.1

Docker Compose:
- 5.3.1

Components installed:

- docker-ce
- docker-ce-cli
- containerd.io
- docker-buildx-plugin
- docker-compose-plugin

Docker application directory:

`/opt`

## NFS Storage

The Zyxel NAS326 provides NFS storage to this VM.

Current persistent mount:

```text
NAS NFS export: homelab -> /mnt/nas
```

The NAS photo share is also mounted for Immich:

```text
NAS NFS export: photo -> /mnt/photo-library
```

The photo mount uses NFSv3 and is exposed to the Immich server container as a read-only bind mount.

## Immich

Immich is deployed with Docker Compose under `/opt/immich`.

Services:

- `immich_server`
- `immich_postgres`
- `immich_redis`
- `immich_machine_learning`

Web interface:

```text
TCP 2283 on the internal network
```

The existing NAS photo collection is configured as an Immich External Library at:

```text
/mnt/photo-library
```

Initial library discovery identified approximately:

- 94,201 photos
- 1,407 videos
- 95,608 total assets

The initial scan produced significant CPU and memory activity. At one point the VM reported approximately 4.4 GiB RAM used with 727 MiB swap used, while the Proxmox host was also under high memory utilization. The scan was left running to establish a complete workload baseline.

Initial configuration choices:

- Storage Template Engine: disabled initially
- Library Watching: disabled during initial validation
- Periodic Scanning: disabled during initial validation
- Map: disabled
- Version Check: enabled

PostgreSQL is currently deployed as part of the Immich Compose stack rather than in a separate LXC. This keeps the application self-contained and avoids unnecessary inter-VM database overhead for the current homelab scale.

## Public Documentation Policy

Actual internal IP addresses, DNS names, MAC addresses, and other environment-specific network identifiers are intentionally omitted from this public repository.
