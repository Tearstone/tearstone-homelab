# Uptime Kuma

## Purpose

Uptime Kuma provides service availability monitoring and email alerting for the home lab. It complements Prometheus and Grafana by checking whether infrastructure, applications, and public endpoints are available rather than collecting performance metrics.

The service runs natively on the dedicated `infra-uptime01` Debian 13 unprivileged LXC. The deployment uses Uptime Kuma 2.0.0, Node.js 22.23.2 LTS, SQLite, and TCP port 3001.

## Installation

Install Node.js, Git, ICMP tooling, and capability management:

```bash
apt update
apt install -y curl ca-certificates git iputils-ping libcap2-bin
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
```

Create the service account and install Uptime Kuma 2.0.0:

```bash
useradd --system --home-dir /opt/uptime-kuma --shell /usr/sbin/nologin uptime-kuma
git clone --branch 2.0.0 --depth 1 https://github.com/louislam/uptime-kuma.git /opt/uptime-kuma
cd /opt/uptime-kuma
npm run setup
chown -R uptime-kuma:uptime-kuma /opt/uptime-kuma
```

Allow the non-root service account to perform ICMP checks without making the LXC privileged:

```bash
setcap cap_net_raw=ep /usr/bin/ping
getcap /usr/bin/ping
```

Create `/etc/systemd/system/uptime-kuma.service`:

```ini
[Unit]
Description=Uptime Kuma
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=uptime-kuma
Group=uptime-kuma
WorkingDirectory=/opt/uptime-kuma
ExecStart=/usr/bin/npm run start-server
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable the service:

```bash
systemctl daemon-reload
systemctl enable --now uptime-kuma
```

## Configuration

The initial monitoring policy uses a 10-minute heartbeat interval and two retries before declaring a monitor unavailable.

* ICMP Ping monitors cover infrastructure, including the Proxmox nodes and `infra-vpn01`.
* HTTP(S) monitors cover internal application services.
* Prometheus uses its `/-/healthy` endpoint.
* The internal Portainer monitor ignores certificate validation because its endpoint uses a self-signed certificate.
* Public HTTPS monitors cover `tearstone.com`, `rvtravelbug.com`, and `rsanderlin.com` with certificate validation enabled.
* Email notifications use Google Workspace SMTP with TLS/STARTTLS on port 587 and an application-specific password.

SQLite is used for the current home-lab workload. Credentials and private monitor addresses remain outside this repository.

## Validation

Verify the service, listener, local web response, non-root ICMP capability, and logs:

```bash
systemctl status uptime-kuma --no-pager
ss -lntp | grep ':3001'
curl --head http://localhost:3001
sudo -u uptime-kuma ping -c 1 <INTERNAL_HOST>
journalctl -u uptime-kuma -n 100 --no-pager
```

Use the Uptime Kuma interface to send a notification test and confirm every configured monitor reports the expected state.

## Monitoring and Integrations

The `Lab Status` status page supplies aggregate availability data to the native Uptime Kuma widget on Homepage. Uptime Kuma appears in the Homepage Monitoring group.

`infra-vpn01` is monitored with ICMP Ping, providing an availability signal for the Tailscale subnet-router LXC independently of the Node Exporter metrics collected by Prometheus.

## Security

* Uptime Kuma runs as a dedicated non-login system account.
* The LXC remains unprivileged.
* Raw-socket capability is assigned only to `/usr/bin/ping`, not to the entire container.
* SMTP credentials, application passwords, and private monitor addresses are excluded from the public repository.
* Public TLS validation remains enabled; exceptions are limited to specific trusted internal endpoints using self-signed certificates.

## Lessons Learned

* Uptime Kuma provides a useful availability layer alongside Prometheus and Grafana.
* Dedicated health endpoints are more meaningful than simple port checks when an application exposes them.
* An unprivileged LXC can perform ICMP monitoring when the `ping` executable has `cap_net_raw`.
* SQLite is sufficient for the current home-lab monitoring workload.
