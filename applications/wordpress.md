# WordPress

## Purpose

`rsanderlin.com` is being migrated from DreamHost shared hosting to the home lab on `prod-web01`.

The migration is being performed in stages so the existing public site remains available until the new origin has been fully validated.

## Target Architecture

The current production path for the web application is:

```text
Internet
  |
  v
Cloudflare DNS / Proxy
  |
  | Cloudflare Tunnel (outbound from prod-web01)
  v
prod-web01
  |
  +-- cloudflared
  +-- Nginx
  +-- PHP-FPM 8.4
  +-- MariaDB 11.8
  +-- WordPress
```

The home ISP public IP is not used as the application's public DNS target. Cloudflare provides the public DNS and proxy layer, while a secure outbound tunnel provides connectivity to `prod-web01`.

## Migration Status

As of 2026-08-22, the following work is complete:

* Provisioned `prod-web01` as a Debian 13 VM.
* Configured PHP 8.4 and PHP-FPM.
* Configured MariaDB 11.8.
* Created a dedicated PHP-FPM pool for the site.
* Created `/srv/www/rsanderlin.com/public` as the WordPress document root.
* Migrated the WordPress filesystem from DreamHost.
* Removed an obsolete Qualys OVA and legacy Drupal directory from the source copy before migration.
* Preserved the WordPress database dump and site archive as migration artifacts during the transfer.
* Imported the WordPress database into local MariaDB; 40 tables were present after import.
* Updated `wp-config.php` for the local MariaDB database and generated new WordPress authentication salts.
* Removed the old DreamHost WP-Cache path from `wp-config.php`.
* Temporarily disabled legacy DreamHost-specific and PHP-incompatible plugins by renaming their plugin paths rather than modifying the WordPress database.
* Configured Nginx for HTTP and HTTPS.
* Generated and installed a Cloudflare Origin Certificate on `prod-web01`.
* Verified the migrated WordPress site locally over HTTPS with Nginx, PHP-FPM, and MariaDB functioning together.
* Installed `cloudflared` 2026.8.2 and established a healthy Cloudflare Tunnel from `prod-web01`.
* Published `rsanderlin.com` through the tunnel using `https://localhost:443` as the origin service.
* Published `www.rsanderlin.com` through the same tunnel using `https://localhost:443` as the origin service.
* Configured the tunnel origin server name as `rsanderlin.com` for both published application routes so the Cloudflare Origin Certificate validates correctly.
* Removed the public apex A record pointing to the former DreamHost web origin and replaced it with the tunnel-backed Cloudflare configuration.
* Replaced the former `www` web record with the Cloudflare-managed Tunnel record associated with `prod-web01`.
* Verified the public apex site through Cloudflare with HTTP/2 `200` and the expected WordPress page title.
* Verified `www.rsanderlin.com` through Cloudflare and confirmed WordPress redirects it to the canonical `https://rsanderlin.com/` URL.
* Changed Cloudflare SSL/TLS encryption mode from **Full** to **Full (strict)**.
* Verified the public site continues to return HTTP/2 `200` after enabling Full (strict).

## DNS and Cloudflare

The domain is registered with Namecheap.

Cloudflare is now authoritative for the domain.

Current web routing:

```text
rsanderlin.com
    |
    v
Cloudflare Tunnel record
    |
    v
prod-web01
    |
    v
https://localhost:443
```

```text
www.rsanderlin.com
    |
    v
Cloudflare Tunnel record
    |
    v
prod-web01
    |
    v
https://localhost:443
```

Cloudflare manages the Tunnel DNS records for the published applications. The public DNS therefore does not expose the home's ISP address as the web origin.

Current DNS records documented during the migration include:

* `rsanderlin.com` — Cloudflare Tunnel, proxied, associated with `prod-web01`.
* `www.rsanderlin.com` — Cloudflare Tunnel, proxied, associated with `prod-web01`.
* `mysql.rsanderlin.com` — A record, DNS-only, still associated with the legacy hosting environment.

The remaining legacy MySQL service requires a separate retirement decision. No inbound web service depends on the former DreamHost web IP.

Cloudflare SSL/TLS is now set to **Full (strict)**. The origin connection is encrypted and Cloudflare validates the Cloudflare Origin Certificate presented by the origin.

## Cloudflare Tunnel

The tunnel is named `prod-web01` and is managed remotely through Cloudflare.

The connector runs as a systemd service on `prod-web01`:

```text
cloudflared.service
```

The service is enabled and has established multiple QUIC connections to Cloudflare. The tunnel reports **Healthy** in the Cloudflare dashboard.

The published application routes are:

```text
Hostname: rsanderlin.com
Service: https://localhost:443
Origin Server Name: rsanderlin.com
```

```text
Hostname: www.rsanderlin.com
Service: https://localhost:443
Origin Server Name: rsanderlin.com
```

The origin server name is required because the Cloudflare Origin Certificate is valid for `rsanderlin.com` and `*.rsanderlin.com`, not `localhost`.

The tunnel token and other private credentials are not committed to this repository.

## Origin Configuration

Nginx uses a dedicated server configuration for `rsanderlin.com` and `www.rsanderlin.com`.

Document root:

`/srv/www/rsanderlin.com/public`

PHP-FPM socket:

`/run/php/rsanderlin.sock`

The Cloudflare Origin Certificate is stored under:

`/etc/nginx/ssl/rsanderlin.com/`

Private certificate material is not committed to this repository.

## WordPress Database

The migrated site uses a dedicated local MariaDB database and database user. Credentials are stored only in the server's `wp-config.php` and are not documented in this public repository.

The imported WordPress database contains 40 tables.

The WordPress `home` and `siteurl` values remain `https://rsanderlin.com`.

## Plugin Migration Notes

The following legacy plugins were temporarily removed from WordPress's plugin discovery path by renaming their files/directories:

* Crayon Syntax Highlighter
* DreamHost Panel Login
* WP Super Cache

Crayon generated PHP 8.4 regular-expression warnings during initial testing. The plugin was disabled without changing the WordPress database.

The remaining active plugins were left untouched during the initial migration validation.

## Validation

The origin was validated locally with HTTPS before the public web cutover:

* Nginx configuration test passed.
* Nginx was listening on TCP 80 and 443.
* `curl` returned `HTTP/1.1 200 OK` when connecting to the local HTTPS virtual host with `rsanderlin.com` as the hostname.
* The response contained the expected WordPress site HTML and title.

The tunnel-backed public site was subsequently validated:

* Cloudflare DNS resolves `rsanderlin.com` to Cloudflare anycast addresses rather than the former DreamHost web IP.
* The Cloudflare Tunnel reports **Healthy**.
* `curl -I https://rsanderlin.com/` returns `HTTP/2 200` with `server: cloudflare`.
* `curl -s https://rsanderlin.com/ | grep -i '\<title>' | head` returns the expected WordPress page title: `Russ Sanderlin – Onwards and Upwards`.
* Browser access to `https://rsanderlin.com` successfully loads the migrated site.
* `curl -I https://www.rsanderlin.com/` returns `HTTP/2 301` with `Location: https://rsanderlin.com/`.
* The tunnel logs no longer report the previous origin certificate hostname mismatch after setting `originServerName` to `rsanderlin.com`.
* After changing Cloudflare SSL/TLS to **Full (strict)**, `curl -I https://rsanderlin.com/` continued to return `HTTP/2 200` with `server: cloudflare`.

## Current Milestone

The `rsanderlin.com` web application migration to the home lab is operational through Cloudflare Tunnel.

The public web path no longer depends on the home's public ISP address, and Cloudflare now validates the encrypted origin connection using the installed Cloudflare Origin Certificate.

Both the apex and `www` hostnames are published through the same `prod-web01` tunnel. WordPress treats the apex hostname as canonical and redirects `www` to `rsanderlin.com`.

This milestone establishes the Cloudflare Tunnel and Full (strict) pattern that can be reused for subsequent public applications and domains.

## Next Steps

The `rsanderlin.com` web routing milestone is complete. Remaining work is application validation and legacy service retirement rather than DNS migration of the web application.

Next work:

1. Validate WordPress administration, uploads, background tasks, and remaining application functionality through the tunnel.
2. Determine the future role of the legacy `mysql.rsanderlin.com` record and the remaining DreamHost services.
3. Retire the DreamHost web hosting after the migrated site and required legacy services are confirmed stable or replaced.
4. Use the established Cloudflare Tunnel pattern when beginning migration of `rvtravelbug.com`.
