# infra-uptime01

## Purpose

`infra-uptime01` is a dedicated Debian 13 LXC providing Uptime Kuma for service availability monitoring and alerting across the home lab.

Uptime Kuma complements Prometheus and Grafana by focusing on service availability, endpoint health, and notifications rather than time series metrics and performance visualization.

## Deployment

| Setting | Value |
| --- | --- |
| Hostname | `infra-uptime01` |
| Platform | Proxmox LXC |
| Operating System | Debian GNU/Linux 13 (Trixie) |
| Uptime Kuma | 2.0.0 |
| Node.js | 22.23.2 LTS |
| npm | 10.9.8 |
| Database | SQLite |
| Application path | `/opt/uptime-kuma` |
| Default web port | 3001 |
| CPU | 1 vCPU |
| Memory | 512 MB |
| Swap | 512 MB |
| Root disk | 8 GB |
| Container security | Unprivileged |

Uptime Kuma is installed directly on the LXC rather than running inside Docker. The application runs as the dedicated `uptime-kuma` system user and is managed by systemd.

## Installation

Node.js 22.23.2 LTS and npm 10.9.8 were installed using the NodeSource Node.js 22 repository.

The application was cloned from the upstream Uptime Kuma repository and pinned to release `2.0.0`.

Installation was completed with:

```text
npm run setup
```

The setup process installed the production dependency tree and downloaded the Uptime Kuma distribution assets.

## systemd Service

Uptime Kuma is configured as a systemd service using `/etc/systemd/system/uptime-kuma.service`.

The service:

* Runs as the `uptime-kuma` system account.
* Uses `/opt/uptime-kuma` as its working directory.
* Starts `server/server.js` with the system Node.js runtime.
* Restarts automatically after an unexpected failure.
* Starts automatically when the LXC boots.

## Database

SQLite was selected for the initial deployment because the home lab monitoring workload does not require a separate database server. Keeping the database local also avoids introducing another service to maintain and back up.

## ICMP Monitoring

Uptime Kuma Ping monitors require the ability to create raw ICMP sockets. The LXC remains unprivileged.

Debian's `iputils-ping` package did not provide the required file capability inside this LXC, so `cap_net_raw` was explicitly assigned to `/usr/bin/ping`:

```text
/usr/bin/ping cap_net_raw=ep
```

The capability was verified by successfully running an ICMP ping as the non-root `uptime-kuma` service account.

This approach grants the capability to the ping executable rather than adding `CAP_NET_RAW` to the entire LXC configuration.

## Monitoring Configuration

The initial monitoring strategy uses a 10 minute heartbeat interval with two retries. This is appropriate for a non-critical home lab and avoids unnecessary monitoring traffic while still providing reasonable outage detection.

### Infrastructure

ICMP Ping monitors are used for infrastructure systems such as the Proxmox nodes.

### Internal Services

HTTP(s) monitors are used for application services. Where an application provides a dedicated health endpoint, the health endpoint is preferred over checking only the application's root page.

Prometheus is monitored using:

```text
/-/healthy
```

rather than only checking TCP port 9090.

Portainer is monitored over HTTPS. Because the internal Portainer endpoint uses a self-signed certificate, the Uptime Kuma monitor is configured to ignore TLS/SSL validation errors for that internal monitor.

### Public Services

The following public websites are monitored through their HTTPS endpoints:

* `tearstone.com`
* `rvtravelbug.com`
* `rsanderlin.com`

Public HTTPS certificate validation remains enabled for these monitors so certificate problems can be detected.

## Notifications

Uptime Kuma is configured to send email notifications through Google Workspace SMTP.

The SMTP configuration uses Google's authenticated SMTP service with TLS/STARTTLS on port 587. Authentication uses a Google application-specific password rather than the normal account password.

SMTP credentials are not stored in the public repository.

## Homepage Integration

The Uptime Kuma status page is integrated into the Homepage dashboard on `infra-homepage01`.

Homepage uses the Uptime Kuma service widget to display aggregate monitoring status. The dashboard includes a dedicated Uptime Kuma entry in the Monitoring group and uses the Dashboard Icons `uptime-kuma` icon.

The status page provides a summary of monitored sites and their current availability without exposing Uptime Kuma's internal configuration or credentials.

## Security and Documentation Notes

* The LXC is unprivileged.
* Uptime Kuma runs as a dedicated non-login system account.
* The application is installed directly on the LXC without an additional Docker layer.
* SMTP credentials and other secrets are kept out of public documentation.
* Private IP addresses and other internal network identifiers are intentionally omitted from this public repository.

## Lessons Learned

* An unprivileged Debian LXC can run Uptime Kuma successfully without making the container privileged.
* Uptime Kuma Ping monitoring requires `CAP_NET_RAW` access, which can be provided through the `ping` executable's file capability.
* Dedicated application health endpoints provide more meaningful monitoring than checking whether a TCP port responds.
* Internal services with self-signed certificates can be monitored over HTTPS by selectively disabling TLS validation for that monitor.
* SQLite is sufficient for the current home lab monitoring workload.
