# WordPress

## Purpose

`rsanderlin.com` is being migrated from DreamHost shared hosting to the home lab on `prod-web01`.

The migration is being performed in stages so the existing public site remains available until the new origin has been fully validated.

## Target Architecture

The planned production path is:

```text
Internet
  |
  v
Cloudflare
  |
  | Secure outbound tunnel
  v
prod-web01
  |
  +-- Nginx
  +-- PHP-FPM 8.4
  +-- MariaDB 11.8
  +-- WordPress
```

The home ISP public IP will not be used as the application's permanent public DNS target. Cloudflare will provide the public DNS and proxy layer, while a secure outbound tunnel will provide connectivity to `prod-web01`.

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

## DNS and Cloudflare

The domain is registered with Namecheap.

Cloudflare is now authoritative for the domain. The existing web records remain configured to the former DreamHost origin while the new home lab origin is prepared.

Current Cloudflare proxy behavior:

* Web records are proxied through Cloudflare.
* FTP, MySQL, and SSH records remain DNS-only.

Cloudflare SSL/TLS is currently set to **Full**. The planned final mode is **Full (strict)** after the secure tunnel and new origin are fully validated.

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

The origin was validated locally with HTTPS before any public web cutover:

* Nginx configuration test passed.
* Nginx was listening on TCP 80 and 443.
* `curl` returned `HTTP/1.1 200 OK` when connecting to the local HTTPS virtual host with `rsanderlin.com` as the hostname.
* The response contained the expected WordPress site HTML and title.

## Next Steps

The migration is intentionally paused before the final web cutover.

Next work:

1. Build and validate the secure Cloudflare tunnel to `prod-web01`.
2. Verify Cloudflare-to-origin connectivity through the tunnel.
3. Change Cloudflare SSL/TLS from Full to Full (strict).
4. Change the proxied web DNS records from the DreamHost origin to the new tunnel-backed origin.
5. Validate the public site.
6. Retire the DreamHost web hosting after the new site is confirmed stable.
