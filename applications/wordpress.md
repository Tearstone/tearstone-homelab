# WordPress

## Purpose

`rsanderlin.com` and `rvtravelbug.com` are WordPress sites migrated from DreamHost shared hosting to the home lab on `prod-web01`.

The migrations use the same production architecture so additional WordPress sites can be moved without exposing the home's ISP address.

## Target Architecture

The current production path for both web applications is:

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

The home ISP public IP is not used as either application's public DNS target. Cloudflare provides the public DNS and proxy layer, while a secure outbound tunnel provides connectivity to `prod-web01`.

## Migration Status

As of 2026-08-22, the following work is complete for `rsanderlin.com`:

* Provisioned `prod-web01` as a Debian 13 VM.
* Configured PHP 8.4 and PHP-FPM.
* Configured MariaDB 11.8.
* Created a dedicated PHP-FPM pool and Linux service account for the site.
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
* Completed browser-level validation of the migrated site.

The following work is complete for `rvtravelbug.com`:

* Created a complete DreamHost filesystem archive before migration: approximately 502 MB of WordPress files compressed to approximately 407 MB.
* Created a complete WordPress database dump from the DreamHost-hosted database; the dump is approximately 7.8 MB and was verified to complete successfully.
* Transferred the filesystem archive and database dump to `prod-web01`.
* Created a dedicated MariaDB database `wordpress_rvtravelbug` and database user `wp_rvtravelbug`.
* Imported the WordPress database; 12 tables are present after import.
* Created dedicated Linux user and group `wp-rvtravelbug` with `/usr/sbin/nologin` and `/srv/www/rvtravelbug.com` as the home directory.
* Created `/srv/www/rvtravelbug.com/public` as the WordPress document root.
* Migrated the WordPress filesystem from DreamHost and assigned it to the dedicated `wp-rvtravelbug` account.
* Updated `wp-config.php` to use the local MariaDB database and generated new WordPress authentication salts.
* Created a dedicated PHP-FPM pool and socket for the site.
* Configured Nginx for HTTP and HTTPS using a dedicated virtual host.
* Generated and installed a Cloudflare Origin Certificate for `rvtravelbug.com` and `*.rvtravelbug.com`.
* Validated the local HTTPS origin and confirmed the expected WordPress title: `RV Travel Bug – Full Time RV Family Travel Blog`.
* Added `rvtravelbug.com` to Cloudflare and changed authoritative DNS from DreamHost to Cloudflare.
* Added `rvtravelbug.com` and `www.rvtravelbug.com` to the existing `prod-web01` Cloudflare Tunnel.
* Configured both tunnel routes to use `https://localhost:443` with `originServerName` set to `rvtravelbug.com`.
* Replaced the public DreamHost A records with Cloudflare Tunnel-backed DNS records.
* Verified the public apex site through Cloudflare with HTTP/2 `200`.
* Verified `www.rvtravelbug.com` redirects to the canonical `https://rvtravelbug.com/` URL.
* Changed Cloudflare SSL/TLS to **Full (strict)** and verified both public hostnames continue to function correctly.
* Completed browser-level validation including the homepage, WordPress administration, media library, and older posts with images.

## DNS and Cloudflare

Both domains are registered with Namecheap.

Cloudflare is authoritative for both domains.

### rsanderlin.com

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

### rvtravelbug.com

Current web routing:

```text
rvtravelbug.com
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
www.rvtravelbug.com
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

Cloudflare manages the Tunnel DNS records for the published web applications. The public DNS therefore does not expose the home's ISP address as either web origin.

The remaining legacy database/service records require separate retirement decisions. The web applications no longer depend on the former DreamHost web IP.

Cloudflare SSL/TLS is set to **Full (strict)** for the migrated sites. The origin connection is encrypted and Cloudflare validates the Cloudflare Origin Certificate presented by the origin.

## Cloudflare Tunnel

The existing `prod-web01` tunnel is shared by the migrated WordPress applications and is managed remotely through Cloudflare.

The connector runs as a systemd service on `prod-web01`:

```text
cloudflared.service
```

The service is enabled and maintains multiple QUIC connections to Cloudflare. The tunnel reports **Healthy** in the Cloudflare dashboard.

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

```text
Hostname: rvtravelbug.com
Service: https://localhost:443
Origin Server Name: rvtravelbug.com
```

```text
Hostname: www.rvtravelbug.com
Service: https://localhost:443
Origin Server Name: rvtravelbug.com
```

The origin server name is required because the Cloudflare Origin Certificates validate against the site hostname rather than `localhost`.

The tunnel token and other private credentials are not committed to this repository.

## Origin Configuration

Each WordPress site has a dedicated Nginx virtual host, PHP-FPM pool, Linux service account, and MariaDB database/user.

### rsanderlin.com

Document root:

`/srv/www/rsanderlin.com/public`

PHP-FPM socket:

`/run/php/rsanderlin.sock`

Linux account:

`wp-rsanderlin`

The Cloudflare Origin Certificate is stored under:

`/etc/nginx/ssl/rsanderlin.com/`

### rvtravelbug.com

Document root:

`/srv/www/rvtravelbug.com/public`

PHP-FPM socket:

`/run/php/rvtravelbug.sock`

Linux account:

`wp-rvtravelbug`

The Cloudflare Origin Certificate is stored under:

`/etc/nginx/ssl/rvtravelbug.com/`

For both PHP-FPM pools, the WordPress worker runs under the dedicated site account while the FPM socket is owned by `www-data` so Nginx can access it.

Private certificate material and application credentials are not committed to this repository.

## WordPress Databases

Each migrated site uses a dedicated local MariaDB database and database user. Credentials are stored only in the server's `wp-config.php` files and are not documented in this repository.

### rsanderlin.com

The imported WordPress database contains 40 tables.

The WordPress `home` and `siteurl` values remain `https://rsanderlin.com`.

### rvtravelbug.com

The local database is:

`wordpress_rvtravelbug`

The local database user is:

`wp_rvtravelbug`

The imported WordPress database contains 12 tables.

The WordPress `home` and `siteurl` values remain `https://rvtravelbug.com`.

## Plugin Migration Notes

### rsanderlin.com

The following legacy plugins were temporarily removed from WordPress's plugin discovery path by renaming their files/directories:

* Crayon Syntax Highlighter
* DreamHost Panel Login
* WP Super Cache

Crayon generated PHP 8.4 regular-expression warnings during initial testing. The plugin was disabled without changing the WordPress database.

### rvtravelbug.com

The source site contained four plugins:

* Advanced Database Cleaner
* Akismet
* Jetpack
* Really Simple SSL

No DreamHost-specific plugin was identified during the initial migration inventory.

## Validation

### rsanderlin.com

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

### rvtravelbug.com

The origin was validated locally with HTTPS before the public web cutover:

* PHP-FPM configuration test passed.
* The dedicated `rvtravelbug.sock` was created and made accessible to Nginx.
* Nginx HTTPS configuration loaded successfully after the Origin Certificate was installed.
* `curl -kI --resolve rvtravelbug.com:443:127.0.0.1 https://rvtravelbug.com/` returned `HTTP/1.1 200 OK`.
* The local response contained the expected title: `RV Travel Bug – Full Time RV Family Travel Blog`.

The tunnel-backed public site was subsequently validated:

* `dig +trace NS rvtravelbug.com` confirmed Cloudflare authoritative nameservers.
* Public DNS changed from the DreamHost web IP to Cloudflare anycast addresses after the Tunnel DNS records were enabled.
* The Cloudflare Tunnel configuration contains routes for both `rvtravelbug.com` and `www.rvtravelbug.com` with `originServerName=rvtravelbug.com`.
* `curl -I https://rvtravelbug.com/` returns `HTTP/2 200` with `server: cloudflare`.
* `curl -I https://www.rvtravelbug.com/` returns `HTTP/2 301` with `Location: https://rvtravelbug.com/`.
* Cloudflare SSL/TLS **Full (strict)** was enabled and validated successfully.
* Browser validation confirmed the homepage, WordPress administration, media library, and older posts with images are functional.

## Current Milestone

Both `rsanderlin.com` and `rvtravelbug.com` are now operational from the home lab through the shared `prod-web01` Cloudflare Tunnel.

The public web paths no longer depend on the home's public ISP address. Cloudflare validates the encrypted origin connections using the installed Cloudflare Origin Certificates under **Full (strict)** mode.

Both domains publish their apex and `www` hostnames through the same tunnel. WordPress treats the apex hostname as canonical and redirects `www` to the apex hostname.

This establishes a repeatable migration pattern for subsequent public web applications and domains.

## Next Steps

1. Identify and retire the remaining DreamHost web/database services associated with the migrated domains after confirming no dependencies remain.
2. Continue application validation and monitoring for both migrated WordPress sites.
3. Use the established Cloudflare Tunnel and Full (strict) pattern for the next domain migration.
4. Migrate `rvtravelbug.com` legacy services only after confirming they are no longer required by the new local WordPress installation.
