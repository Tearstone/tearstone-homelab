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
