# Immich

## Overview

Immich is deployed on `lab-core01` as a Docker Compose application. It provides photo management, mobile backup, search, machine-learning processing, and access to the existing NAS photo collection.

Current Immich version: `v3.1.0`.

## Deployment

Host:

* `lab-core01`
* Debian 13
* 4 vCPU
* 8 GB RAM
* Local NVMe storage

Containers:

* `immich_server`
* `immich_postgres`
* `immich_redis`
* `immich_machine_learning`

Web interface:

* TCP 2283

Configuration directory:

```text
/opt/immich
```

The PostgreSQL data directory remains local to the VM:

```text
/opt/immich/postgres
```

## Storage Architecture

Immich uses a split storage model. Latency-sensitive metadata and thumbnails remain on local NVMe, while the large media payload is stored on the Zyxel NAS.

| Function | Host path | Storage |
| --- | --- | --- |
| PostgreSQL | `/opt/immich/postgres` | Local NVMe |
| Thumbnails | `/opt/immich/library/thumbs` | Local NVMe |
| Immich managed media | `/mnt/immich-photo/Immich` | NAS NFSv3 |
| Encoded video | `/mnt/immich-photo/Immich/encoded-video` | NAS NFSv3 |
| Backups | `/mnt/immich-photo/Immich/backups` | NAS NFSv3 |
| Profile | `/mnt/immich-photo/Immich/profile` | NAS NFSv3 |
| Existing photo collection | `/mnt/photo-library` | NAS NFSv3, read-only in container |

The NAS export is:

```text
192.168.12.172:/i-data/cfb9d897/photo
```

mounted at:

```text
/mnt/immich-photo
```

This means phone uploads are physically stored within the NAS `photo` filesystem and remain accessible through the NAS SMB Photo share.

## External Library

The existing photo collection is maintained independently of Immich's managed storage.

Host mount:

```text
/mnt/photo-library
```

Container mount:

```text
/mnt/photo-library:ro
```

The external library is read-only from the Immich container. Immich does not reorganize or manage the existing photo filesystem.

## Storage Template

Immich's Storage Template is enabled using the default template:

```text
{{y}}/{{y}}-{{MM}}-{{dd}}/{{filename}}
```

Managed media therefore follows a date-based hierarchy such as:

```text
Immich/library/admin/2026/2026-08-16/IMG_2736.heic
```

This keeps the Immich-managed upload area human-readable while leaving the existing master photo collection untouched.

## Mobile Backup

The iOS Immich application is being configured for automatic photo backup.

Validated upload path:

```text
 iPhone
   -> Immich mobile backup
   -> lab-core01
   -> /mnt/immich-photo/Immich
   -> Zyxel NAS `/i-data/cfb9d897/photo/Immich`
```

A test HEIC upload was successfully verified on the NAS before beginning the full camera-roll upload.

The initial mobile migration includes more than 30,000 files and is being allowed to run unattended. Video backup will be enabled separately after photo backup completes successfully.

## Performance Considerations

Thumbnails intentionally remain on local NVMe storage. Moving the thumbnail cache to the NAS would introduce unnecessary NFS latency during normal Immich browsing.

Encoded video is stored on the NAS because it is substantially larger and is less latency-sensitive than thumbnail access.

The local VM disk was previously at approximately 83% utilization. Moving Immich encoded video and backup data to the NAS reclaimed approximately 15.8 GB and reduced local filesystem utilization to approximately 62%.

## Backup and Recovery

Immich VM backups are stored on the NAS through the Proxmox NFS backup storage.

A manual backup of `lab-kali01` was successfully created at the NAS before removing the inactive VM from the local storage footprint.

Before the Immich storage migration, the existing 14 GB encoded-video data and 1.8 GB of Immich backups were copied to the NAS and verified by file count. The original local copies were retained temporarily, then removed after the migrated Immich server was verified healthy and functional.

## Operational Notes

Do not manually move or rename files within Immich's managed storage. Immich tracks managed media paths in its database and Storage Template operations should be used for managed media changes.

The existing external library may be reorganized independently because it remains outside Immich's managed storage.
