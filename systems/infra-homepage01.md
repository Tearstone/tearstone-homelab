# infra-homepage01

## Purpose

`infra-homepage01` is a dedicated LXC container providing the lab's Homepage dashboard. It serves as the primary navigation point for lab infrastructure, monitoring, management, and applications.

The container intentionally runs Homepage directly on Debian rather than adding Docker as another containerization layer.

## Platform

Operating system:
- Debian GNU/Linux 13 (trixie)

Virtualization:
- Proxmox VE LXC

Hostname:
- `infra-homepage01`

Architecture:
- x86-64

Memory:
- 512 MB RAM
- 512 MB swap

The container was temporarily increased to 1 GB RAM during the initial Next.js production build. The memory allocation was reduced to 512 MB after the build completed successfully.

## Homepage

Homepage:
- Version 2.1.2
- Installed from source
- Runs directly on the LXC
- No Docker or Docker Compose layer

The installation uses the Homepage source repository and a native Node.js runtime.

Node.js:
- 22.23.2 LTS

npm:
- 10.9.8

pnpm:
- 10.34.5

The production build required additional memory because Next.js performs a memory intensive build process. The completed application runs comfortably within the 512 MB container allocation.

## Dashboard Organization

Homepage is organized into the following groups:

### Infrastructure

- Proxmox VE cluster
- Zyxel NAS326
- NETGEAR GS108Ev4 managed switch

### Monitoring

- Grafana
- Prometheus

### Management

- Portainer
- AdGuard

### Applications

- Immich

The dashboard uses service specific icons and descriptions to make the primary lab services identifiable at a glance.

## Proxmox Integration

Homepage is integrated with the `nexus` Proxmox cluster using a dedicated read only API identity.

Proxmox authentication configuration:

- User: `homepage@pam`
- Group: `api-readonly`
- Role: `PVEAuditor`
- API token: `dashboard`
- Privilege separation: enabled
- Permission path: `/`
- Propagation: enabled

The API token is intentionally scoped to read only Proxmox information. The token secret is not stored in public documentation.

The Homepage Proxmox widget reports cluster wide VM and LXC counts and cluster CPU and memory utilization. No individual node is specified, so CPU and memory are calculated for the complete cluster.

## Configuration

Homepage configuration is maintained under:

```text
/opt/homepage/homepage/config/
```

Primary configuration files include:

```text
services.yaml
proxmox.yaml
```

`services.yaml` defines the dashboard service links and service widgets. The Proxmox service widget uses the dedicated API token to retrieve cluster information.

The live configuration contains internal service addresses and API credentials and is therefore not committed to the public repository.

## Resource Considerations

The container is intentionally small because Homepage is primarily a dashboard and navigation service rather than a monitoring engine.

Prometheus and Grafana remain responsible for monitoring and historical metrics. Homepage provides convenient access to those services and selected live status information.

## Public Documentation Policy

Actual internal IP addresses, API token secrets, authentication credentials, MAC addresses, and other environment specific operational details are intentionally omitted from this public repository.
