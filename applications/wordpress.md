# WordPress

## Purpose

`rsanderlin.com` is being migrated from DreamHost shared hosting to the home lab on `prod-web01`.

The migration is being performed in stages so the existing public site remains available until the new origin has been fully validated.

## Target Architecture

The current production path for the apex domain is:

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
* Installed a Cloudflare Origin Certificate on `prod-web01`.
* Verified the migrated WordPress site locally over HTTPS with Nginx, PHP-FPM, and MariaDB functioning together.
* Installed `cloudflared` 2026.8.2 and established a healthy Cloudflare Tunnel from `prod-web01`.
* Published `rsanderlin.com` through the tunnel using `https://localhost:443` as the origin service.
* Configured the tunnel origin server name as `rsanderlin.com` so the Cloudflare Origin Certificate validates correctly.
* Removed the public apex A record pointing to the former DreamHost web origin and replaced it with the tunnel-backed Cloudflare configuration.
* Verified the public apex site through Cloudflare with HTTP/2 `200` and the expected WordPress page title.

## DNS and Cloudflare

The domain is registered with Namecheap.

Cloudflare is now authoritative for the domain.

Current web routing for the apex domain:

```text
rsanderlin.com
    |
    v
Cloudflare proxied DNS
    |
    v
Cloudflare Tunnel
    |
    v
prod-web01:443
```

The tunnel publishes the apex hostname to the local Nginx HTTPS service. Cloudflare DNS uses the tunnel-backed configuration rather than the former DreamHost web IP.

Current Cloudflare proxy behavior:

* `rsanderlin.com` is proxied through Cloudflare and is routed through the `prod-web01` tunnel.
* `www.rsanderlin.com` has not yet been moved to the tunnel.
* FTP, MySQL, and SSH records remain DNS-only and are still associated with the legacy hosting environment.

Cloudflare SSL/TLS is currently set to **Full**. The final **Full (strict)** configuration remains a subsequent migration step after the tunnel-backed apex site has been observed and validated.

## Cloudflare Tunnel

The tunnel is named `prod-web01` and is managed remotely through Cloudflare.

The connector runs as a systemd service on `prod-web01`:

```text
cloudflared.service
```

The service is enabled and has established multiple QUIC connections to Cloudflare. The tunnel reports **Healthy** in the Cloudflare dashboard.

The published application route is:

```text
Hostname: rsanderlin.com
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
* The response contains the expected WordPress page title: `Russ Sanderlin – Onwards and Upwards`.
* Browser access to `https://rsanderlin.com` successfully loads the migrated site.
* The tunnel logs no longer report the previous origin certificate hostname mismatch after setting `originServerName` to `rsanderlin.com`.

## Next Steps

The apex web migration is now operational through Cloudflare Tunnel. The migration remains staged rather than fully complete.

Next work:

1. Observe and validate the apex site through the tunnel.
2. Move `www.rsanderlin.com` to the tunnel and validate redirects and HTTPS behavior.
3. Change Cloudflare SSL/TLS from Full to Full (strict).
4. Validate WordPress administration, uploads, background tasks, and remaining application functionality through the tunnel.
5. Determine the future role of the legacy FTP, MySQL, and SSH records.
6. Retire the DreamHost web hosting after the migrated site is confirmed stable.
