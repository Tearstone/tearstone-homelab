```mermaid
flowchart LR
    NAS["Zyxel NAS326  192.168.12.168"]

    subgraph Storage["NAS Storage"]
        NFS["NFS Share  homelab"]
        PHOTO["NFS Export  photo"]
    end

    CORE["lab-core01  192.168.12.244  Debian 13 / Docker"]
    IMMICH["Immich External Library"]

    NAS --> NFS
    NAS --> PHOTO
    NFS -->|NFSv3| CORE
    PHOTO -->|NFSv3| CORE
    CORE -->|read-only bind mount| IMMICH
```

## Homelab Share

The `homelab` NFS share is hosted on the Zyxel NAS326 and is currently restricted to `lab-core01` (`192.168.12.244`).

NAS export:

```text
/i-data/cfb9d897/nfs/homelab
```

Client mount:

```text
192.168.12.168:/i-data/cfb9d897/nfs/homelab -> /mnt/nas
```

The mount is persistent through `/etc/fstab` and uses `_netdev` so system startup waits for the network.

## Photo Export

The existing Zyxel photo share is separately exported over NFS to `lab-core01` so Immich can index the existing collection without moving or reorganizing the underlying files.

NAS export:

```text
/i-data/cfb9d897/photo
```

Client mount:

```text
192.168.12.168:/i-data/cfb9d897/photo -> /mnt/photo-library
```

The photo export is restricted to `192.168.12.244` and uses NFSv3.

Inside the Immich container, the directory is exposed at:

```text
/mnt/photo-library
```

The container mount is read-only. The same underlying `/photo` directory remains available through the existing SMB `photo` share, allowing other users and systems to continue accessing the collection without Immich changing its directory structure.

## Validation

The NFS photo export was validated from `lab-core01` by mounting the export directly and confirming the existing directories and files were visible. The mounted collection is approximately 384 GB and contains tens of thousands of existing photo and video assets.

Immich successfully accessed the collection through the container mount and discovered approximately 94,201 photos and 1,407 videos during the initial scan.
