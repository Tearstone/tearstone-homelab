# Roadmap

## Phase 1 — Virtualization

- [x] Procure HP EliteDesk 800 G5 Mini
- [x] Install Proxmox VE
- [x] Debian 13 VM installed
- [x] Qualys Scanner Appliance installed
- [x] Kali Linux VM installed
- [ ] Create VM templates
- [ ] Document VM and LXC standards

## Phase 2 — Monitoring

- [x] Prometheus installed
- [x] Node Exporter installed
- [x] Grafana installed
- [x] Dashboards created
- [x] Deploy Uptime Kuma
- [x] Monitor network and infrastructure services
- [x] Monitor internal application services
- [x] Monitor public web services
- [x] Configure email alerting for availability monitoring
- [ ] Configure Prometheus alerting and Alertmanager
- [ ] Blackbox Exporter

Prometheus Alertmanager and Blackbox Exporter are intentionally deferred. Uptime Kuma currently provides the lab's availability monitoring and notification requirements. Prometheus alerting will be revisited when metric based operational alerts become necessary.

## Phase 3 — Network and Core Services

- [x] Procure managed network switch
- [x] Connect additional lab systems
- [x] Establish network performance baseline
- [x] Deploy internal DNS
- [ ] Configure internal hostname resolution
- [ ] Deploy Certificate Authority
- [ ] Establish internal PKI
- [ ] Configure internal TLS certificates
- [ ] Implement VLANs
- [ ] Implement network segmentation

## Phase 4 — Storage and Backup

- [x] Connect Zyxel NAS 326
- [x] Configure NFS storage
- [x] Benchmark NAS/NFS performance
- [ ] Configure automated Proxmox backups
- [ ] Implement backup verification
- [ ] Document backup and recovery procedures

**Current project:** Proxmox Backup and Disaster Recovery. The existing NAS/NFS infrastructure will be used as the initial backup target. The project will establish scheduled backups, retention, verification, restore testing, and documented recovery objectives.

See [Backup and Recovery](docs/backup.md) for the project plan.

## Phase 5 — Applications and Services

- [x] Install Docker
- [ ] Deploy Home Assistant
- [x] Deploy Immich
- [ ] Deploy reverse proxy
- [ ] Configure HTTPS for internal services
- [ ] Integrate internal DNS with applications
- [ ] Integrate Certificate Authority with applications
- [x] Establish Cloudflare Tunnel pattern for public web applications

## Phase 6 — Advanced Infrastructure

- [x] Procure second Proxmox node
- [x] Configure Proxmox cluster
- [ ] Explore high availability
- [x] Expand storage infrastructure
- [ ] Implement centralized logging
- [ ] Explore Ansible and infrastructure automation
- [ ] Explore infrastructure as code
- [ ] Expand security testing environment
- [ ] Experiment with local LLM infrastructure

## Current Infrastructure Milestones

- [x] Create two-node `nexus` Proxmox cluster
- [x] Add `pve02` to the cluster as a peer node
- [x] Migrate `lab-core01` from `pve01` to `pve02`
- [x] Complete controlled `lab-core01` A/B benchmark between `pve01` and `pve02`
- [x] Establish G5-specific host benchmark baselines for `pve01` and `pve02`
- [ ] Measure whole-system power consumption for both G5 nodes
- [x] Install 1 TB Optimus 5001 NVMe in `pve02`
- [x] Expand `pve01` from 16 GB to 32 GB RAM
- [x] Expand `pve02` from 16 GB to 32 GB RAM
- [x] Benchmark the upgraded `pve01` configuration
- [x] Benchmark the upgraded `pve02` configuration
- [x] Deploy dedicated Homepage infrastructure LXC
- [x] Deploy dedicated Uptime Kuma infrastructure LXC

## Future Ideas

- [ ] DHCP services
- [ ] Network intrusion detection
- [ ] SIEM
- [ ] Advanced monitoring and observability
- [ ] Container orchestration
- [ ] Automated patch management
- [ ] Advanced network segmentation
- [ ] Disaster recovery testing
- [ ] Additional compute nodes
- [ ] 2.5 GbE or faster networking

## Documentation

- [x] Establish GitHub homelab documentation
- [x] Document G5 hardware configuration
- [x] Establish G5 performance baseline
- [x] Document network architecture
- [x] Document service architecture
- [x] Document storage architecture
- [ ] Document backup architecture
- [ ] Document security architecture
- [x] Maintain benchmark history
- [x] Document Homepage deployment
- [x] Document Uptime Kuma deployment
- [x] Document monitoring architecture
- [x] Define backup and recovery project plan

## Notes

The roadmap was updated on September 6, 2026 to reflect the current infrastructure and the start of the backup and recovery project. The two-node Proxmox cluster, G5 hardware upgrades, workload benchmarking, dedicated Homepage dashboard, and Uptime Kuma availability monitoring are complete. The lab now has separate monitoring layers for infrastructure metrics and service availability. Backup automation and recovery validation are now the immediate infrastructure priority. High availability, centralized logging, network segmentation, and advanced security infrastructure remain future work.
