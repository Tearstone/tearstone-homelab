# Homepage

## Purpose

Homepage provides a centralized dashboard for navigating and viewing the status of lab services. It is hosted on the dedicated `infra-homepage01` LXC container.

![Nexus Lab Homepage](/images/homepage.jpg)

## Deployment

Homepage is installed directly on Debian GNU/Linux 13 rather than running inside Docker.

Application:
- Homepage 2.1.2

Runtime:
- Node.js 22.23.2 LTS
- npm 10.9.8
- pnpm 10.34.5

Installation path:

```text
/opt/homepage/homepage
```

Configuration path:

```text
/opt/homepage/homepage/config
```

## Dashboard Services

The dashboard currently provides links and selected live widgets for:

| Group | Services |
| --- | --- |
| Infrastructure | Proxmox, Zyxel NAS326, NETGEAR GS108Ev4 |
| Monitoring | Grafana, Prometheus |
| Management | Portainer, AdGuard |
| Applications | Immich |

## Proxmox Widget

Homepage uses a dedicated Proxmox API token to retrieve read only cluster information.

The widget displays:

- Running and total QEMU VM counts
- Running and total LXC counts
- Cluster CPU utilization
- Cluster memory utilization

The CPU and memory values are cluster wide because no individual Proxmox node is specified in the widget configuration.

The API identity is assigned the `PVEAuditor` role through the dedicated `api-readonly` group. Privilege separation is enabled for the API token.

## Resource Allocation

The LXC is allocated 512 MB RAM and 512 MB swap. The RAM allocation was temporarily increased to 1 GB during the initial Next.js production build because the build process exceeded the smaller Node.js heap limit. The container was returned to 512 MB after the build completed.

## Configuration Management

Homepage automatically reloads YAML configuration changes, allowing dashboard updates without a manual application restart.

The live configuration contains private network addresses and API credentials. Those values are intentionally excluded from this public repository.

## Public Documentation Policy

API token secrets, passwords, private IP addresses, MAC addresses, and other internal operational details are not published in this repository.
