# Changelog

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

* `lab-core01` is configured with 4 vCPU, 8 GB RAM, and 2 GB swap.
* Initial Immich indexing and machine-learning processing saturated the VM CPU during processing.
* During the initial scan, the VM reached approximately 4.4 GiB RAM used and 727 MiB swap used.
* The Proxmox host also reached high memory utilization during the workload. Resource usage will be evaluated again after the initial scan completes.

## 2026-08-07

### Infrastructure

* Completed 10" mini rack installation with PDU, NETGEAR switch, patch panel, and mounted EliteDesk.
* Connected Zyxel NAS326 to the rack network via the NETGEAR switch.
* Upgraded NAS326 firmware to V5.21(AAZF.18) Hotfix 01.
* NAS326 is EOL. Downloaded the latest available applications and configured a local package repository.

### Storage

* Installed and enabled NFS on NAS326.
* Created and exported the `homelab` NFS share.
* Restricted NFS access to `lab-core01` (`192.168.12.244`).

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
- Transferred Debian 13 VM from VMware Fusion running on Macbook to Proxmox node 1
- Transferred Qualys Scanner Appliance from VMware Fusion running on Macbook Proxmox node 1
- Provisioned Kali Linux VM
