```mermaid

flowchart LR
    NAS["Zyxel NAS326  192.168.12.168"]
    
    subgraph Storage["NAS Storage"]
        NFS["NFS Share  homelab"]
    end

    CORE["lab-core01  192.168.12.244  Debian 13 / Docker"]

    NAS --> NFS
    NFS -->|NFS| CORE
```

The `homelab` NFS share is hosted on the Zyxel NAS326 and is currently restricted to `lab-core01`. The share will provide network storage for applications running on the Docker host.
