# Changelog

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
