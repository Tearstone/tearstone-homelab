```mermaid
graph TD
    Internet --> Router

    Router --> pve (Proxmox)

    Proxmox --> lab-core01
    Proxmox --> lab-kali01
    Proxmox --> lab-qualys01
    Proxmox --> infra-grafana
    Proxmox --> infra-prometheus
```
