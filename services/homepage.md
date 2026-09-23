# Homepage

## Purpose

Homepage provides a centralized dashboard for navigating and viewing the status of lab services. It is hosted on the dedicated `infra-homepage01` Debian 13 LXC.

![Nexus Lab Homepage](/images/homepage.jpg)

## Installation

Homepage 2.1.2 is installed directly from source rather than through Docker. The runtime uses Node.js 22.23.2 LTS, npm 10.9.8, and pnpm 10.34.5.

Install the runtime and build prerequisites:

```bash
apt update
apt install -y curl ca-certificates git
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
npm install --global pnpm@10.34.5
```

Clone, install, and build Homepage:

```bash
mkdir -p /opt/homepage
git clone --branch v2.1.2 --depth 1 https://github.com/gethomepage/homepage.git /opt/homepage/homepage
cd /opt/homepage/homepage
pnpm install --frozen-lockfile
cp -a src/skeleton/. config/
NODE_OPTIONS="--max-old-space-size=768" pnpm build
```

The LXC was temporarily increased from 512 MB to 1 GB RAM for the production build and returned to 512 MB afterward.

Create `/etc/systemd/system/homepage.service`:

```ini
[Unit]
Description=Homepage dashboard
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
WorkingDirectory=/opt/homepage/homepage
Environment=NODE_ENV=production
Environment=HOMEPAGE_ALLOWED_HOSTS=<HOMEPAGE_HOST>:3000
ExecStart=/usr/bin/pnpm start
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable the service:

```bash
systemctl daemon-reload
systemctl enable --now homepage
```

## Configuration

Homepage configuration is stored in `/opt/homepage/homepage/config`. The active `services.yaml` defines four service groups:

| Group | Services |
| --- | --- |
| Infrastructure | Proxmox, Zyxel NAS326, NETGEAR GS108Ev4 |
| Monitoring | Uptime Kuma, Grafana, Prometheus |
| Management | Portainer, AdGuard, Tailscale |
| Applications | Immich |

The live configuration contains private service addresses and credentials and is not committed. A sanitized example is maintained at [`docs/homepage-services.yaml.example`](../docs/homepage-services.yaml.example).

Homepage automatically reloads YAML configuration changes, so routine dashboard edits do not require a service restart.

## Validation

Verify the service, listener, and local HTTP response:

```bash
systemctl status homepage --no-pager
ss -lntp | grep ':3000'
curl --head http://localhost:3000
journalctl -u homepage -n 100 --no-pager
```

Open `http://<HOMEPAGE_HOST>:3000` and confirm each group, service link, and configured widget loads successfully.

## Monitoring and Integrations

The Proxmox widget uses a dedicated read-only `homepage@pam` API identity with a privilege-separated token and the `PVEAuditor` role. It displays cluster VM and LXC counts plus CPU and memory utilization.

The Uptime Kuma widget reads aggregate availability from the `Lab Status` status page. The Prometheus widget displays target counts. AdGuard and Immich use their application APIs for summary statistics.

The Management group includes a Tailscale tile linking to the Tailscale administration interface so enrolled devices and the sanitized private-LAN route can be reviewed.

## Security

* Proxmox access is read-only and uses a privilege-separated API token.
* API secrets are supplied through Homepage environment variables rather than committed YAML.
* Private host addresses, API tokens, passwords, and other internal identifiers are excluded from the public repository.
* The public example uses placeholders for environment-specific values.

## Lessons Learned

* Homepage runs comfortably with 512 MB RAM after the production build is complete.
* The Next.js build required a temporary memory increase and an explicit Node.js heap limit.
* Widgets are most useful when they provide concise operational status rather than duplicate full application dashboards.
* A sanitized example preserves configuration methodology without exposing the private network.
