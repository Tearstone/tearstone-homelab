# Docker

## Purpose

Docker provides the container platform for running home lab applications.

Current workloads:

- Immich photo management
- Future self-hosted applications

## Installation

Docker was installed using the official Docker repository.

Repository:

https://download.docker.com/linux/debian

Operating System:

Debian 13 (Trixie)

Architecture:

amd64

## Components

Docker Engine - Provides container lifecycle management.
Docker CLI - Command line interface for managing containers.
containerd - Container runtime used by Docker.
Docker Compose - Used for defining and managing multi-container applications.

Versions on `lab-core01`:

- Docker Engine 29.7.1
- Docker Compose 5.3.1

## Directory Structure

Docker applications are stored under `/opt` on `lab-core01`.

Current application:

- `/opt/immich`

Each application contains its Docker Compose configuration and environment configuration. Secrets such as database passwords must not be committed to GitHub.

## Storage Design

Application data is separated from the VM operating system where practical.

Current storage:

- Local VM storage for Immich application and PostgreSQL data
- Zyxel NAS326 NFS storage for the existing photo collection

The Immich server accesses the existing photo collection through a read-only container bind mount at `/mnt/photo-library`. Immich treats this location as an External Library and does not manage the underlying NAS directory structure.

## Immich Deployment

Immich is deployed as a Docker Compose application under `/opt/immich`.

Services:

- `immich_server`
- `immich_postgres`
- `immich_redis`
- `immich_machine_learning`

Immich exposes HTTP on TCP port `2283`.

The initial deployment was validated with all four containers reporting healthy.

## Immich Initial Configuration

Initial configuration completed on 2026-08-11.

- Server privacy: configured during initial setup
- Version Check: enabled
- Map: disabled
- Storage Template Engine: disabled initially
- External Library: configured as `Zyxel Photo Library`
- External Library path inside the container: `/mnt/photo-library`
- Library Watching: disabled initially
- Periodic Scanning: disabled initially

The external library contains the existing photo collection stored on the Zyxel NAS326. Immich accesses the collection in place and does not copy or reorganize the source files.

## Initial Indexing

Initial library scan identified approximately:

- 94,201 photos
- 1,407 videos
- Approximately 95,608 total assets

The source collection is approximately 384 GB as measured from `lab-core01` over NFS.

During initial indexing, the 8 GB VM experienced substantial CPU and memory utilization. The Immich server and machine learning containers were the primary consumers. Swap usage increased to approximately 727 MiB during the scan, while the VM continued to report approximately 3.3 GiB available memory.

The Proxmox host also reached high overall memory utilization during the workload. No VM resource changes were made while the initial scan was running. Post-scan resource behavior will be evaluated before increasing the VM allocation.
