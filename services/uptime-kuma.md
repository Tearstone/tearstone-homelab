# Uptime Kuma

## Purpose

Uptime Kuma provides service availability monitoring and email alerting for the home lab. It complements Prometheus and Grafana by monitoring whether infrastructure, applications, and public endpoints are available rather than collecting performance metrics.

## Deployment

Uptime Kuma is hosted on the dedicated `infra-uptime01` Debian 13 LXC.

| Component | Value |
| --- | --- |
| Uptime Kuma | 2.0.0 |
| Node.js | 22.23.2 LTS |
| Database | SQLite |
| Web port | 3001 |
| Installation | Native Linux installation |
| Service account | `uptime-kuma` |
| Service manager | systemd |

Detailed host and deployment information is documented in [infra-uptime01](../systems/infra-uptime01.md).

## Monitoring Model

The initial configuration uses a 10 minute heartbeat interval and two retries. This is intentionally sized for a non-critical home lab.

### Infrastructure

ICMP Ping monitors are used for infrastructure such as the Proxmox nodes. The LXC remains unprivileged and the `ping` executable has the required `cap_net_raw` file capability.

### Internal Services

HTTP(s) monitors are used for application services. Dedicated health endpoints are preferred when available.

Prometheus is monitored through `/-/healthy`. Portainer is monitored over HTTPS with TLS validation disabled for the internal monitor because its certificate is self-signed.

### Public Websites

Uptime Kuma monitors the public HTTPS endpoints for:

* `tearstone.com`
* `rvtravelbug.com`
* `rsanderlin.com`

Public certificate validation remains enabled for these monitors.

## Notifications

Email notifications are sent through Google Workspace SMTP using TLS/STARTTLS on port 587. The SMTP account uses a Google application-specific password. Credentials are not stored in the public repository.

## Homepage Integration

A `Lab Status` Uptime Kuma status page provides the data source for the native Uptime Kuma widget on the Homepage dashboard. Uptime Kuma is displayed in the Homepage Monitoring group using the `uptime-kuma` Dashboard Icons asset.

## Security

* Uptime Kuma runs as a dedicated non-login system account.
* The LXC is unprivileged.
* No Docker layer is used for the Uptime Kuma deployment.
* SMTP credentials and other secrets remain outside public documentation.
* Private network addresses are excluded from the public repository.

## Lessons Learned

* Uptime Kuma provides a useful availability layer alongside Prometheus and Grafana.
* Health endpoints are preferable to simple port checks when an application exposes them.
* Internal self-signed certificates can be monitored by selectively disabling TLS validation for the affected monitor.
* SQLite is sufficient for the current home lab workload.
