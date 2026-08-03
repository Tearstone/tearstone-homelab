# Docker

## Purpose

Docker provides the container platform for running home lab applications.

Current planned workloads:

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

## Directory Structure

Docker applications are stored under: /opt/docker

Example: /opt/docker/immich

Each application will contain:

- docker-compose.yml
- environment files
- configuration data

## Storage Design

Application data will be separated from the VM operating system.

Planned storage:

- Zyxel NAS326 NFS storage
- Local VM storage for application configuration
