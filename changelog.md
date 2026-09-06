# Changelog

## 2026-09-06

### Uptime Kuma Monitoring

* Provisioned `infra-uptime01` as a dedicated Debian 13 unprivileged LXC for Uptime Kuma.
* Installed Uptime Kuma 2.0.0 natively rather than adding a Docker layer.
* Installed Node.js 22.23.2 LTS and npm 10.9.8.
* Configured Uptime Kuma to run as the dedicated `uptime-kuma` system account under systemd.
* Selected SQLite for the initial Uptime Kuma database.
* Configured 10 minute monitoring intervals with two retries before declaring a monitor unavailable.
* Enabled ICMP Ping monitoring in the unprivileged LXC by assigning `cap_net_raw` to `/usr/bin/ping` rather than granting raw socket capability to the entire container.
* Added HTTP(s) monitoring for internal services and public websites.
* Configured Prometheus monitoring through its `/-/healthy` health endpoint.
* Configured Portainer HTTPS monitoring with TLS validation disabled for the internal self-signed certificate.
* Added public HTTPS monitoring for `tearstone.com`, `rvtravelbug.com`, and `rsanderlin.com` with certificate validation enabled.
* Configured Google Workspace SMTP using TLS/STARTTLS on port 587 and an application-specific password for email notifications.
* Created the `Lab Status` Uptime Kuma status page for dashboard integration.

### Homepage Integration

* Added the Uptime Kuma service and aggregate status widget to the Homepage dashboard.
* Added Uptime Kuma to the Monitoring group and configured the `uptime-kuma` Dashboard Icons asset.
* Verified the Homepage Uptime Kuma widget displays monitor availability from the `Lab Status` status page.

### Documentation

* Added `systems/infra-uptime01.md` documenting the host, installation, systemd service, ICMP capability, monitoring strategy, notifications, security, and lessons learned.
* Added `services/uptime-kuma.md` documenting the Uptime Kuma service and monitoring model.
* Updated `services/homepage.md` with the Uptime Kuma integration.
* Updated `docs/monitoring.md` to include Uptime Kuma as the availability and alerting layer alongside Prometheus and Grafana.
* Updated `docs/architecture.md` with the new `infra-uptime01` LXC, Homepage integration, monitoring flows, and SMTP alert path.
* Updated `README.md` with Uptime Kuma and `infra-uptime01` in the current environment and monitoring stack.

### Roadmap

* Updated `roadmap.md` to mark Uptime Kuma deployment, infrastructure and application availability monitoring, and availability email alerting as complete.
* Added dedicated Homepage and Uptime Kuma infrastructure milestones.
* Marked the established Cloudflare Tunnel pattern for public web applications as complete.
* Corrected the 1 TB Optimus 5001 NVMe milestone to identify `pve02` as the host.
* Marked the completed `pve02` 32 GB memory upgrade and benchmarking work.
* Updated the roadmap status date to September 6, 2026.

### Backup and Recovery Project

* Started the Proxmox Backup and Disaster Recovery project using the existing NAS/NFS infrastructure as the initial backup target.
* Added `docs/backup.md` defining the backup architecture, workload priorities, project phases, recovery objectives, security considerations, and success criteria.
* Updated `roadmap.md` to identify backup and recovery as the immediate infrastructure priority and to defer Prometheus Alertmanager and Blackbox Exporter until metric based alerting or probing requirements emerge.
* Updated `README.md` with the backup and recovery documentation and current project direction.

## 2026-08-30

### Homepage Dashboard

* Provisioned `infra-homepage01` as a dedicated Debian 13 LXC for the lab Homepage dashboard.
* Installed Homepage 2.1.2 directly on the LXC from source rather than adding a Docker layer.
* Installed Node.js 22.23.2 LTS, npm 10.9.8, and pnpm 10.34.5 for the native Homepage build and runtime.
* Completed the Next.js production build after temporarily increasing the LXC memory allocation from 512 MB to 1 GB.
* Returned `infra-homepage01` to 512 MB RAM with 512 MB swap after the build completed successfully.
* Organized the dashboard into Infrastructure, Monitoring, Management, and Applications groups.
* Added navigation for Proxmox, Zyxel NAS326, NETGEAR GS108Ev4, Grafana, Prometheus, Portainer, AdGuard, and Immich.
* Added service specific icons and descriptions for the primary lab services.

### Homepage Proxmox Integration

* Created a dedicated `homepage@pam` Proxmox API identity.
* Created the `api-readonly` group and assigned the `PVEAuditor` role at the cluster root with propagation enabled.
* Created the `dashboard` API token with privilege separation enabled.
* Assigned `PVEAuditor` permissions to the API token at the cluster root.
* Configured the Homepage Proxmox widget to retrieve read only cluster information.
* Verified the widget reports the complete `nexus` cluster rather than an individual node.
* Verified the dashboard reports 4/4 VMs, 4/4 LXCs, and cluster CPU and memory utilization.
* Kept the API token secret and private service addresses out of the public repository documentation.

### pve01 Memory Upgrade

* Added a second 16 GB DDR4-2667 SODIMM to `pve01`.
* Increased installed memory from 16 GB to 32 GB.
* `pve01` now operates with 2 × 16 GB SODIMMs in a dual-channel configuration.
* Updated the README, hardware documentation, and pve01 system documentation to reflect the 32 GB configuration.

### pve01 Benchmarking

* Promoted the existing post-upgrade 32 GB memory results to the current pve01 host baseline.
* Retained the original 16 GB results as the historical comparison baseline.
* Documented 1 thread memory results of 24,180.55 MiB/sec read and 21,487.18 MiB/sec write after the upgrade.
* Documented 6 thread memory results of 106,762.51 MiB/sec read and 78,147.37 MiB/sec write after the upgrade.
* Updated the benchmarking index to reflect the 32 GB configuration and removed the completed 32 GB upgrade from planned comparisons.

## 2026-08-29

### pve02 Storage Expansion

* Installed a 1 TB SanDisk Optimus 5100 NVMe in `pve02` as a second physical storage device.
* Verified the drive at approximately 931.5 GiB capacity.
* Created a GPT partition using the full device and initialized `/dev/nvme0n1p1` as an LVM physical volume.
* Created the `pve-fast` volume group on the new NVMe.
* Created a full-capacity `data` LVM-thin pool of approximately 931.3 GiB.
* Added the thin pool to Proxmox as storage ID `nvme-lvm` with `images,rootdir` content enabled.
* Verified storage allocation and removal through `pvesm` using a temporary 10 GB test volume.
* Documented the 512 KiB thin-pool chunk size selected by LVM and the associated provisioning warning for future consideration.

### pve02 NVMe Benchmarking

* Benchmarked the Optimus 5100 before partitioning and deployment using fio 3.39, `io_uring`, direct I/O, queue depth 32, and 30-second time-based tests.
* Measured sequential read performance of approximately 3,425 MiB/sec with 1 MiB blocks.
* Measured sequential write performance of approximately 3,216 MiB/sec with 1 MiB blocks.
* Measured 4K random-read performance of approximately 355.6K IOPS at queue depth 32.
* Measured 4K random-write performance of approximately 102.4K IOPS at queue depth 32.
* Measured a 70/30 4K random mixed workload at approximately 63.0K read IOPS and 27.0K write IOPS.
* Recorded the results as raw-device performance rather than guest VM performance.

### lab-core01 Storage Migration

* Migrated VM 100 `lab-core01`'s 80 GB system disk from `local-lvm` to the new `nvme-lvm` storage tier.
* Performed the migration online while `lab-core01` remained running.
* Used Proxmox `qm move_disk` with source-volume deletion enabled only after the mirror completed successfully.
* Verified the resulting VM configuration points `scsi0` to `nvme-lvm:vm-100-disk-0`.
* Verified `lab-core01` remained running after migration.
* Verified the Debian guest continued to see an 80 GB system disk.
* Verified Docker was operational and all four Immich containers were healthy after migration.
* Confirmed the original `local-lvm:vm-100-disk-1` volume was removed after successful migration.

## 2026-08-22

### Applications

* Began migration of `rsanderlin.com` WordPress from DreamHost shared hosting to the home lab `prod-web01` VM.
* Provisioned the WordPress application stack on Debian 13 with Nginx, PHP 8.4, PHP-FPM, and MariaDB 11.8.
* Migrated the WordPress filesystem and imported the existing database; the imported database contains 40 tables.
* Updated the WordPress configuration for the new local MariaDB backend and generated new authentication salts.
* Removed the obsolete DreamHost WP-Cache path from the WordPress configuration.
* Temporarily disabled Crayon Syntax Highlighter, DreamHost Panel Login, and WP Super Cache by renaming their plugin paths rather than modifying the WordPress database.
* Documented Crayon Syntax Highlighter as incompatible with the PHP 8.4 environment after it generated regular-expression warnings during testing.

### `rsanderlin.com` Networking and TLS

* Moved authoritative DNS for `rsanderlin.com` from DreamHost to Cloudflare.
* Configured the web records through the Cloudflare proxy while leaving the remaining legacy service record DNS-only.
* Created and installed a Cloudflare Origin Certificate on `prod-web01`.
* Configured Nginx for HTTPS on TCP 443 and verified the migrated WordPress site locally over HTTPS.
* Confirmed the local origin returns `HTTP/1.1 200 OK` and serves the expected WordPress HTML.
* Installed `cloudflared` 2026.8.2 on `prod-web01` and established a healthy outbound Cloudflare Tunnel.
* Published `rsanderlin.com` through the tunnel using `https://localhost:443` as the origin service.
* Published `www.rsanderlin.com` through the same tunnel using `https://localhost:443` as the origin service.
* Configured the tunnel origin server name as `rsanderlin.com` so the Cloudflare Origin Certificate validates correctly for both published application routes.
* Removed the apex web A record pointing to the former DreamHost origin and moved `rsanderlin.com` to the tunnel-backed Cloudflare configuration.
* Replaced the former `www` web record with the Cloudflare-managed Tunnel record associated with `prod-web01`.
* Verified the public apex site through Cloudflare with HTTP/2 `200` and the expected WordPress page title.
* Verified `www.rsanderlin.com` through Cloudflare and confirmed WordPress redirects it to the canonical `https://rsanderlin.com/` URL.
* Confirmed the home ISP public IP is not used as the permanent public web origin; the public web application is served through the outbound Cloudflare Tunnel.
* Changed Cloudflare SSL/TLS from `Full` to **Full (strict)** and verified the public site continued to return HTTP/2 `200`.
* Completed browser-level validation of the migrated site.

### `rsanderlin.com` Migration Status

* The `rsanderlin.com` website is operational through Cloudflare Tunnel and `prod-web01`.
* Both `rsanderlin.com` and `www.rsanderlin.com` are published through the same tunnel.
* WordPress redirects `www.rsanderlin.com` to the canonical `rsanderlin.com` hostname.
* Cloudflare validates the encrypted origin connection using the installed Cloudflare Origin Certificate under **Full (strict)** mode.
* `mysql.rsanderlin.com` remains DNS-only and associated with the legacy hosting environment pending a separate retirement decision.
* The `rsanderlin.com` web routing migration is complete. Remaining work is application validation and retirement of legacy services.

### `rvtravelbug.com` Migration

* Created a complete DreamHost filesystem archive before migration: approximately 502 MB of WordPress files compressed to approximately 407 MB.
* Created and verified a complete WordPress database dump from the DreamHost-hosted database; the dump is approximately 7.8 MB.
* Transferred the filesystem archive and database dump to `prod-web01`.
* Created dedicated MariaDB database `wordpress_rvtravelbug` and database user `wp_rvtravelbug`.
* Imported the WordPress database; 12 tables are present after import.
* Created dedicated Linux user and group `wp-rvtravelbug` with `/usr/sbin/nologin` and `/srv/www/rvtravelbug.com` as the home directory.
* Created `/srv/www/rvtravelbug.com/public` as the WordPress document root and assigned it to the dedicated site account.
* Updated `wp-config.php` for the local MariaDB database and generated new WordPress authentication salts.
* Created a dedicated PHP-FPM pool and socket for `rvtravelbug.com`.
* Configured Nginx for HTTP and HTTPS using a dedicated virtual host.
* Generated and installed a Cloudflare Origin Certificate covering `rvtravelbug.com` and `*.rvtravelbug.com`.
* Validated the local HTTPS origin and confirmed the expected WordPress title: `RV Travel Bug – Full Time RV Family Travel Blog`.
* Added `rvtravelbug.com` to Cloudflare and changed authoritative DNS from DreamHost to Cloudflare.
* Added `rvtravelbug.com` and `www.rvtravelbug.com` to the existing `prod-web01` Cloudflare Tunnel.
* Configured both tunnel routes to use `https://localhost:443` with `originServerName` set to `rvtravelbug.com`.
* Replaced the public DreamHost A records with Cloudflare Tunnel-backed DNS records.
* Verified the public apex site through Cloudflare with HTTP/2 `200`.
* Verified `www.rvtravelbug.com` redirects to the canonical `https://rvtravelbug.com/` URL.
* Changed Cloudflare SSL/TLS to **Full (strict)** and verified both public hostnames continue to function correctly.
* Completed browser-level validation of the homepage, WordPress administration, media library, and older posts with images.

### `rvtravelbug.com` Migration Status

* The `rvtravelbug.com` website is now operational through Cloudflare Tunnel and `prod-web01`.
* Both `rvtravelbug.com` and `www.rvtravelbug.com` are published through the shared tunnel.
* WordPress redirects `www.rvtravelbug.com` to the canonical `rvtravelbug.com` hostname.
* Cloudflare validates the encrypted origin connection using the installed Cloudflare Origin Certificate under **Full (strict)** mode.
* The web application no longer depends on the former DreamHost web IP.
* Remaining DreamHost database/service records require a separate retirement decision.

### Next Migration

* The Cloudflare Tunnel and Full (strict) migration pattern is now established for subsequent public web applications.
* `rvtravelbug.com` and `rsanderlin.com` are the reference implementations for the next domain migration.

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
* Enabled the default Immich Storage Template: `{{y}}/{{y}}-{{MM}}/{{dd}}/{{filename}}`.
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

### Applications

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
