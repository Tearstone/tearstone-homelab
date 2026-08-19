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
- [ ] Configure alerting
- [ ] Blackbox Exporter
- [ ] Monitor network and infrastructure services

## Phase 3 — Network and Core Services

- [x] Procure managed network switch
- [x] Connect additional lab systems
- [x] Establish network performance baseline
- [ ] Deploy internal DNS
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

## Phase 5 — Applications and Services

- [x] Install Docker
- [ ] Deploy Home Assistant
- [x] Deploy Immich
- [ ] Deploy reverse proxy
- [ ] Configure HTTPS for internal services
- [ ] Integrate internal DNS with applications
- [ ] Integrate Certificate Authority with applications

## Phase 6 — Advanced Infrastructure

- [x] Procure second Proxmox node
- [x] Configure Proxmox cluster
- [ ] Explore high availability
- [ ] Expand storage infrastructure
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
- [ ] Install 1 TB Optimus 5001 NVMe in `pve01`
- [ ] Expand `pve01` from 16 GB to 32 GB RAM
- [ ] Benchmark the upgraded `pve01` configuration

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

## Notes

The roadmap is updated to reflect the current infrastructure as of August 19, 2026. The two-node Proxmox cluster and initial workload benchmarking are complete. High availability remains exploratory rather than implemented.
