# prod-web01

## Purpose

Dedicated Linux web application server for public-facing home lab services migrated from DreamHost.

Current roles:

- WordPress application server
- Nginx web server
- PHP-FPM application runtime
- MariaDB database server
- Cloudflare Tunnel connector
- Node Exporter monitoring endpoint

Current applications:

- `rsanderlin.com`
- `rvtravelbug.com`

Both public sites use the same Cloudflare Tunnel and separate Linux users, PHP-FPM pools, databases, and Nginx virtual hosts.

## Virtual Machine Details

Hypervisor:
- Proxmox VE

Operating System:
- Debian GNU/Linux 13 (Trixie)

Architecture:
- amd64

Hostname:
- `prod-web01`

Resources and storage details are intentionally omitted from this public system document where they have not been explicitly established as part of the current inventory.

Network:
- Internal network address intentionally omitted from public documentation

## Installed Software

### Web Server

Nginx

Version:
- 1.26.3

The web server terminates HTTPS locally using Cloudflare Origin CA certificates and serves the migrated WordPress applications.

Site document roots:

```text
/srv/www/rsanderlin.com/public
/srv/www/rvtravelbug.com/public
```

Nginx virtual hosts:

```text
/etc/nginx/sites-available/rsanderlin.com
/etc/nginx/sites-available/rvtravelbug.com
```

### PHP

PHP:
- 8.4.24

PHP-FPM:
- 8.4

Each WordPress site uses a dedicated PHP-FPM pool and Unix socket.

```text
rsanderlin.com
  pool: rsanderlin
  socket: /run/php/rsanderlin.sock
  user: wp-rsanderlin
  group: wp-rsanderlin

rvtravelbug.com
  pool: rvtravelbug
  socket: /run/php/rvtravelbug.sock
  user: wp-rvtravelbug
  group: wp-rvtravelbug
```

The PHP-FPM sockets are owned by `www-data` with mode `0660` so Nginx can access them while the PHP worker processes remain isolated under their site-specific Linux accounts.

### Database

MariaDB:
- 11.8.6

Each migrated WordPress site uses a separate database and database account.

`rsanderlin.com`:

```text
database: wordpress_rsanderlin
user: wp_rsanderlin
```

`rvtravelbug.com`:

```text
database: wordpress_rvtravelbug
user: wp_rvtravelbug
```

Database credentials are stored only in the respective WordPress `wp-config.php` files and are not documented in this public repository.

### Cloudflare Tunnel

`cloudflared`:
- 2026.8.2

Systemd service:

```text
cloudflared.service
```

The connector is remotely managed through Cloudflare and establishes outbound QUIC connections to the Cloudflare edge. No inbound web port forwarding is required from the home network.

The current Tunnel is named:

```text
prod-web01
```

Published application routes:

```text
rsanderlin.com       -> https://localhost:443
www.rsanderlin.com   -> https://localhost:443
rvtravelbug.com      -> https://localhost:443
www.rvtravelbug.com  -> https://localhost:443
```

The `rsanderlin.com` routes use `originServerName=rsanderlin.com`.

The `rvtravelbug.com` routes use `originServerName=rvtravelbug.com`.

The tunnel token and other private credentials are not committed to this repository.

## TLS

Cloudflare Origin CA certificates are installed for the migrated domains under:

```text
/etc/nginx/ssl/rsanderlin.com/
/etc/nginx/ssl/rvtravelbug.com/
```

Cloudflare SSL/TLS mode for the migrated web applications is **Full (strict)**.

Cloudflare validates the origin certificates presented by Nginx over the tunnel.

## WordPress Applications

### rsanderlin.com

Migrated from DreamHost shared hosting to `prod-web01`.

Application path:

```text
Cloudflare
  -> Cloudflare Tunnel
  -> Nginx :443
  -> PHP-FPM pool `rsanderlin`
  -> MariaDB `wordpress_rsanderlin`
  -> WordPress
```

The imported WordPress database contains 40 tables.

Legacy DreamHost-specific or PHP-incompatible plugins were disabled by renaming their plugin paths rather than altering the WordPress database.

### rvtravelbug.com

Migrated from DreamHost shared hosting to `prod-web01`.

Application path:

```text
Cloudflare
  -> Cloudflare Tunnel
  -> Nginx :443
  -> PHP-FPM pool `rvtravelbug`
  -> MariaDB `wordpress_rvtravelbug`
  -> WordPress
```

The imported WordPress database contains 12 tables.

The migrated filesystem was approximately 502 MB, with approximately 447 MB in `wp-content` and approximately 407 MB in the compressed migration archive.

The active WordPress plugin inventory at migration time included:

- Advanced Database Cleaner
- Akismet
- Jetpack
- Really Simple SSL

## Monitoring

Node Exporter is installed for Prometheus system monitoring.

Port:
- 9100

The node is monitored as part of the home lab Prometheus/Grafana environment.

## Validation

`rsanderlin.com`:

- Local Nginx HTTPS origin validated successfully.
- Cloudflare Tunnel reports healthy.
- Public apex returns HTTP/2 `200` through Cloudflare.
- `www.rsanderlin.com` returns the expected WordPress redirect to the apex.
- Browser access validated successfully.
- Cloudflare Full (strict) validated successfully.

`rvtravelbug.com`:

- Local Nginx HTTPS origin validated successfully.
- WordPress returned the expected site title: `RV Travel Bug – Full Time RV Family Travel Blog`.
- Cloudflare Tunnel configuration includes both apex and `www` routes.
- Public apex returns HTTP/2 `200` through Cloudflare.
- `www.rvtravelbug.com` returns HTTP `301` to `https://rvtravelbug.com/`.
- Browser validation completed successfully, including the WordPress administration area, media library, and older posts with images.
- Cloudflare Full (strict) validated successfully.

## Architecture Notes

The server intentionally uses separate Linux service accounts, PHP-FPM pools, databases, and Nginx virtual hosts for each WordPress application. This provides application isolation while allowing both domains to share the same physical VM and Cloudflare Tunnel connector.

The public web applications do not depend on the home's ISP public IP being stable. Cloudflare DNS and proxy services provide the public edge, while `cloudflared` maintains outbound connectivity from `prod-web01`.

## Public Documentation Policy

Actual internal IP addresses, MAC addresses, serial numbers, private credentials, Cloudflare tunnel tokens, certificate private keys, and other environment-specific secrets are intentionally omitted from this public repository.
