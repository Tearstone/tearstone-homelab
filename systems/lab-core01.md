# lab-core01

## Purpose

Primary Linux application server running on Proxmox.

Current roles:

- Docker host
- Future Immich application server
- Future home lab application workloads

## Virtual Machine Details

Hypervisor:
- Proxmox VE

Operating System:
- Debian GNU/Linux 13 (Trixie)

Architecture:
- amd64

Hostname:
- lab-core01

Resources:

CPU:
- 1 socket
- 4 core

Disk:
- 80GB virtual disk

Memory:
- [document current value]

Network:
- DHCP address assigned from T-Mobile router

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

/opt/docker
