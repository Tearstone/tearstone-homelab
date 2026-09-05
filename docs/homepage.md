# Homepage Dashboard

## Overview

`infra-homepage01` is a dedicated Debian 13 LXC that provides the Homepage dashboard for the homelab. Homepage is installed natively rather than through Docker or another nested container runtime.

Current software stack:

```text
Debian GNU/Linux 13
Node.js 22.23.2
npm 10.9.8
pnpm 10.34.5
Homepage 2.1.2
```

The LXC uses 512 MB RAM and 512 MB swap for normal operation. A temporary 1 GB RAM allocation was used during the Next.js production build, with `NODE_OPTIONS="--max-old-space-size=768"` required for the build. Memory was returned to 512 MB after the build completed.

## Dashboard Organization

The dashboard is organized into four service groups:

| Group | Services |
| --- | --- |
| Infrastructure | Proxmox, ZyXEL NAS326, NETGEAR GS108Ev4 |
| Monitoring | Prometheus, Grafana |
| Management | AdGuard, Portainer |
| Applications | Immich |

The service order is intentional. Infrastructure provides the underlying platform, Monitoring provides operational visibility, Management provides administration tools, and Applications contains user facing services.

## Layout Strategy

Homepage's default responsive service layout is used. No `layout.yaml` or custom layout override is required for the current dashboard.

This is intentional. Homepage automatically arranges the service groups into responsive columns based on available screen width. The resulting layout keeps the dashboard compact on large displays while remaining usable at smaller widths.

The dashboard is not being customized simply because Homepage supports additional layout controls. A layout override will be introduced only if the default behavior creates a specific usability problem.

## Widgets

Widgets are intentionally limited to operational information that is useful from the dashboard itself. The goal is not to reproduce the functionality of the applications or the detailed dashboards already provided by Grafana and the individual services.

Current widgets:

### Proxmox

The Proxmox widget uses a dedicated read only `homepage@pam` account and privilege separated API token. It displays cluster VM and LXC counts along with CPU and memory utilization.

### Prometheus

The Prometheus widget queries the Prometheus targets API and displays:

* Targets up
* Targets down
* Total targets

This provides an immediate monitoring health check without duplicating Grafana dashboards.

### AdGuard Home

The AdGuard widget uses the `/control/stats` API with HTTP Basic Authentication and displays:

* DNS queries
* Blocked queries
* Filtered requests
* DNS processing latency

### Immich

The Immich widget uses a dedicated API key and displays application statistics including users, photos, videos, and storage usage.

## Configuration

The active service definitions are maintained in `config/services.yaml` on `infra-homepage01`. Secrets are supplied through Homepage environment variables and are not committed to the repository.

The public repository contains a sanitized example at `docs/homepage-services.yaml.example`. Private host addresses and credentials are replaced with placeholders.

The current configuration uses environment variables for the Proxmox API token, AdGuard password, and Immich API key. The AdGuard username is non secret configuration and may be specified directly in the service definition.

## Service Management

Homepage runs as a native systemd service:

```ini
[Service]
Type=simple
User=root
WorkingDirectory=/opt/homepage/homepage
Environment=NODE_ENV=production
Environment=HOMEPAGE_ALLOWED_HOSTS=<HOMEPAGE_HOST>:3000
ExecStart=/usr/bin/pnpm start
Restart=on-failure
RestartSec=5
```

The application listens on TCP port 3000.

## Design Principle

The dashboard is intended to be an operational landing page rather than a catalog of every Homepage feature. Links provide access to the underlying applications, while widgets expose only the small set of metrics that are useful at a glance.

As the lab grows, new widgets should be added only when they provide meaningful operational value and do not unnecessarily duplicate an existing monitoring or application interface.
