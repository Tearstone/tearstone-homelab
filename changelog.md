# Changelog

## 2026-08-22

### Applications

* Began migration of `rsanderlin.com` WordPress from DreamHost shared hosting to the home lab `prod-web01` VM.
* Provisioned the WordPress application stack on Debian 13 with Nginx, PHP 8.4, PHP-FPM, and MariaDB 11.8.
* Migrated the WordPress filesystem and imported the existing database; the imported database contains 40 tables.
* Updated the WordPress configuration for the new local MariaDB backend and generated new authentication salts.
* Removed the obsolete DreamHost WP-Cache path from the WordPress configuration.
* Temporarily disabled Crayon Syntax Highlighter, DreamHost Panel Login, and WP Super Cache by renaming their plugin paths rather than modifying the WordPress database.
* Documented Crayon Syntax Highlighter as incompatible with the PHP 8.4 environment after it generated regular-expression warnings during testing.

### Networking and TLS

* Moved authoritative DNS for `rsanderlin.com` from DreamHost to Cloudflare.
* Configured the web records through the Cloudflare proxy while leaving FTP, MySQL, and SSH records DNS-only.
* Created and installed a Cloudflare Origin Certificate on `prod-web01`.
* Configured Nginx for HTTPS on TCP 443 and verified the migrated WordPress site locally over HTTPS.
* Confirmed the local origin returns `HTTP/1.1 200 OK` and serves the expected WordPress HTML.
* Installed `cloudflared` 2026.8.2 on `prod-web01` and established a healthy outbound Cloudflare Tunnel.
* Published `rsanderlin.com` through the tunnel using `https://localhost:443` as the origin service.
* Configured the tunnel origin server name as `rsanderlin.com` so the Cloudflare Origin Certificate validates correctly.
* Removed the apex web A record pointing to the former DreamHost origin and moved `rsanderlin.com` to the tunnel-backed Cloudflare configuration.
* Verified the public apex site through Cloudflare with HTTP/2 `200` and the expected WordPress page title.
* Confirmed the home ISP public IP is not used as the permanent public web origin; the public apex site is now served through the outbound Cloudflare Tunnel.
* Cloudflare SSL/TLS remains in `Full` mode pending final validation and the later move to `Full (strict)`.

### Migration Status

* The apex `rsanderlin.com` website is now operational through Cloudflare Tunnel and `prod-web01`.
* `www.rsanderlin.com` has not yet been moved to the tunnel.
* FTP, MySQL, and SSH remain DNS-only and still point toward the legacy hosting environment.
* The migration is intentionally paused at this stable milestone before moving additional hostnames or retiring the DreamHost services.
* Next steps are to observe the apex site, migrate `www`, validate WordPress functionality, move Cloudflare to `Full (strict)`, and then determine the retirement plan for the remaining DreamHost services.

## 2026-08-19

### Benchmarking

* Reconciled the G5 Proxmox benchmark documentation for `pve01` and `pve02` so both node documents use consistent structure, terminology, and benchmark scope.
* Standardized the node-specific documentation around `pve01` and `pve02` rather than the legacy `G5-Performance.md` document.
* Added the `pve01` ↔ `pve02` `iperf3` network baseline: approximately 934 to 935 Mbits/sec with zero retransmits in both directions.
* Distinguished Proxmox host CPU and memory benchmarks from the controlled `lab-core01` VM workload benchmarks.
* Documented the controlled `lab-core01` A/B comparison showing `pve02` ahead of `pve01` by approximately 22% to 29% across CPU, memory bandwidth, and 4K random I/O.
* Confirmed `lab-core01` as the preferred workload placement on `pve02`.
* Added current whole-system power measurement status and future power testing to the G5 benchmark documentation.
* Updated `benchmarking/readme.md` to serve as the index for the node-specific G5 baselines, workload comparison, network baseline, and NAS/NFS results.
* Identified `benchmarking/G5-Performance.md` as the legacy G5 benchmark document; future benchmark updates should use the node-specific `pve01` and `pve02` documents.
* Removed private IP addresses, MAC addresses, serial numbers, and internal DNS details from public infrastructure documentation while retaining useful architecture and benchmark information.

## 2026-08-16

### Applications

* Completed Immich storage migration on `lab-core01`.
* Moved Immich managed media storage from the local VM disk to the NAS while retaining thumbnails and PostgreSQL on local NVMe.
* Configured Immich managed storage at `/mnt/immich-photo/Immich` backed by the NAS `photo` NFS export.
* Retained the existing NAS photo collection as a separate read-only Immich External Library.
* Enabled the default Immich Storage Template: `{{y}}/{{y}}-{{MM}}-{{dd}}/{{filename}}`.
* Successfully migrated and verified a test HEIC asset into the date-based storage hierarchy.
* Validated iOS mobile photo upload from the phone through Immich to the NAS.
* Began the full iOS photo backup containing more than 30,000 files.

### Storage

* Added NAS-backed Immich directories for `library`, `upload`, `encoded-video`, `backups`, and `profile`.
* Kept Immich thumbnails on local NVMe to preserve low-latency browsing performance.
* Moved approximately 14 GB of encoded video and 1.8 GB of Immich backups from the local VM disk to NAS storage.
* Reclaimed approximately 15.8 GB from the `lab-core01` VM disk.
* Reduced `lab-core01` root filesystem utilization from approximately 83% to 62%.
* Verified NAS copies of 607 encoded video files and 6 backup files before removing the local copies.

### Reliability

* Created pre-migration backups of the Immich Docker Compose and `.env` configuration files.
* Resolved Immich system-integrity checks by preserving the required `.immich` marker files during the storage migration.
* Verified all Immich containers healthy after the storage migration.

## 2026-08-11

### Applications

* Deployed Immich on `lab-core01` using Docker Compose.
* Deployed Immich Server, PostgreSQL, Redis/Valkey, and Machine Learning services.
* Confirmed all Immich containers report healthy.
* Configured Immich for HTTP access on TCP port 2283.
* Completed initial Immich privacy and application configuration.
* Left Storage Template Engine disabled for the initial deployment.
* Left External Library watching and periodic scanning disabled during initial validation.
* Enabled Immich Version Check and disabled the optional Map integration.

### Storage

* Created an NFS export for the existing NAS `photo` directory restricted to `lab-core01`.
* Mounted the photo export at `/mnt/photo-library` on `lab-core01` using NFSv3.
* Exposed `/mnt/photo-library` to the Immich server container as a read-only bind mount.
* Configured the existing photo collection as an Immich External Library.
* Initial discovery identified approximately 94,201 photos and 1,407 videos.

### Performance

* `lab-core01` was initially configured with 4 vCPU, 8 GB RAM, and 2 GB swap.
* Initial Immich indexing and machine-learning processing saturated the VM CPU during processing.
* During the initial scan, the VM reached approximately 4.4 GiB RAM used and 727 MiB swap used.
* The Proxmox host also reached high memory utilization during the workload. Resource usage was evaluated again after migration to `pve02`.

## 2026-08-07

### Infrastructure

* Completed 10" mini rack installation with PDU, NETGEAR switch, patch panel, and mounted EliteDesk.
* Connected Zyxel NAS326 to the rack network via the NETGEAR switch.
* Upgraded NAS326 firmware to V5.21(AAZF.18) Hotfix 01.
* NAS326 is EOL. Downloaded the latest available applications and configured a local package repository.

### Storage

* Installed and enabled NFS on NAS326.
* Created and exported the `homelab` NFS share.
* Restricted NFS access to `lab-core01`.

## 2026-07-21

- lab-core01 configured as Docker application host
- Installed Docker Engine on lab-core01
- Prepared Docker directory structure
- Began Immich deployment planning

## 2026-07-20

- Added Node Exporter to Prometheus LXC, Grafana LXC, Debian 13 VM, Kali Linux VM
- Confirmed metrics collection
- Created initial GitHub repository structure
- Installed Grafana
- Installed Node Exporter - Full dashboard

## 2026-07-19

- Installed Prometheus
- Transferred Debian 13 VM from VMware Fusion running on MacBook to Proxmox node 1
- Transferred Qualys Scanner Appliance from VMware Fusion on MacBook to Proxmox node 1
- Provisioned Kali Linux VM
