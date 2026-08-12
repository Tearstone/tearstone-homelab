# Architecture

## Current Architecture

```mermaid
graph TD
    Internet --> Router["T-Mobile Gateway"]
    Router --> Switch["NETGEAR GS108E"]

    Switch --> Proxmox["Proxmox Node 1"]
    Switch --> NAS["Zyxel NAS326  192.168.12.168"]

    subgraph "Proxmox Node 1"
        Core["lab-core01  Debian 13 / Docker"]
        Kali["lab-kali01  Kali Linux"]
        Qualys["lab-qualys01  Qualys Scanner"]
        Grafana["infra-grafana01  Grafana"]
        Prometheus["infra-prometheus01  Prometheus"]
    end

    Proxmox --> Core
    Proxmox --> Kali
    Proxmox --> Qualys
    Proxmox --> Grafana
    Proxmox --> Prometheus

    NAS -->|"NFS: homelab"| Core
    NAS -->|"NFS: photo"| Core
    Core -->|"read-only bind mount"| Immich["Immich External Library"]
```

The current homelab consists of a Proxmox virtualization host, Linux VMs and LXCs. `lab-core01` is a dedicated Docker application host, and a Zyxel NAS326 provides network storage using HDD.

The Zyxel NAS326 provides an NFS share named `homelab`, currently restricted to `lab-core01`. A second NFS export provides the existing NAS `photo` directory to `lab-core01` for Immich.

Immich runs as a Docker Compose application on `lab-core01`. Its PostgreSQL and Redis dependencies are containerized alongside the Immich server. The existing NAS photo collection is presented to Immich as a read-only External Library, preserving the NAS filesystem and existing SMB access for other users.

Prometheus collects metrics from the Linux systems using Node Exporter, while Grafana provides visualization of the collected metrics.


## Proposed Cluster

In the not-to-distant future, we'll add a second HP EliteDesk node to the Proxmox environment and establish a small two-node cluster.

```mermaid
graph LR

    NAS[(Zyxel NAS326  NFS Storage)]

    Node1["EliteDesk 1  Proxmox"]
    Node2["EliteDesk 2  Proxmox"]

    Node1 <-->|Proxmox Cluster| Node2

    Node1 -->|NFS| NAS
    Node2 -->|NFS| NAS

    Node1 -->|Docker / Immich| Immich1["Immich"]
    Immich1 -->|External Library| NAS
```

The second node will provide additional compute capacity and allow experimentation with Proxmox clustering, migration, and high availability concepts.

The NAS will provide shared storage accessible to both Proxmox nodes.

## Initial Layout

The original homelab layout

```mermaid
graph TD
    Internet --> Router["T-Mobile Gateway"]
    Router --> Proxmox["Proxmox Node 1"]

    Proxmox --> Core["lab-core01"]
    Proxmox --> Kali["lab-kali01"]
    Proxmox --> Qualys["lab-qualys01"]
    Proxmox --> Grafana["infra-grafana01"]
    Proxmox --> Prometheus["infra-prometheus01"]
```
