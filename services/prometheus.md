# Prometheus

## Purpose

Prometheus collects and stores time-series metrics from the home lab. It scrapes Node Exporter on the monitored Linux hosts and supplies the metrics queried by Grafana.

## Installation

Install Prometheus from the Debian package repository and enable the service:

```bash
apt update
apt install -y curl wget vim htop net-tools unzip
apt policy prometheus
apt install -y prometheus
systemctl enable --now prometheus
```

The package creates the Prometheus service account, configuration directory, data directory, and systemd unit.

## Configuration

The primary configuration file is:

```text
/etc/prometheus/prometheus.yml
```

Sanitized example:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: homelab

rule_files:
  # Add rule files when Prometheus alerting is implemented.

scrape_configs:
  - job_name: prometheus
    scrape_interval: 5s
    scrape_timeout: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: proxmox
    static_configs:
      - targets: ['<PVE01_ADDRESS>:9100']
        labels:
          name: pve01
      - targets: ['<PVE02_ADDRESS>:9100']
        labels:
          name: pve02

  - job_name: linux-vm
    static_configs:
      - targets: ['<LAB_CORE01_ADDRESS>:9100']
        labels:
          name: lab-core01
      - targets: ['<LAB_KALI01_ADDRESS>:9100']
        labels:
          name: lab-kali01
      - targets: ['<PROD_WEB01_ADDRESS>:9100']
        labels:
          name: prod-web01

  - job_name: linux-lxc
    static_configs:
      - targets: ['<INFRA_GRAFANA01_ADDRESS>:9100']
        labels:
          name: infra-grafana01
      - targets: ['<INFRA_VPN01_ADDRESS>:9100']
        labels:
          name: infra-vpn01
```

Check the configuration before reloading Prometheus:

```bash
promtool check config /etc/prometheus/prometheus.yml
systemctl reload prometheus
```

## Validation

Verify the systemd service, health endpoint, metrics endpoint, and recent logs:

```bash
systemctl status prometheus --no-pager
curl http://localhost:9090/-/healthy
curl http://localhost:9090/metrics
journalctl -u prometheus -n 100 --no-pager
```

Open `http://<PROMETHEUS_HOST>:9090`, select **Status → Targets**, and confirm all expected targets report `UP`.

## Current Targets

* `pve01` — physical Proxmox node
* `pve02` — physical Proxmox node
* `infra-prometheus01` — Debian 13 LXC
* `infra-grafana01` — Debian 13 LXC
* `infra-vpn01` — Debian 13 LXC
* `lab-kali01` — Kali Linux VM
* `lab-core01` — Debian 13 VM
* `prod-web01` — Debian 13 VM

## Monitoring and Integrations

Grafana uses Prometheus as its metrics data source. Homepage queries the Prometheus targets API to display target counts, and Uptime Kuma independently checks the Prometheus `/-/healthy` endpoint.

## Security

* Prometheus is an internal monitoring service and should not be exposed directly to the public internet.
* Private target addresses are replaced with descriptive placeholders in this repository.
* Authentication secrets and environment-specific configuration remain outside public documentation.

## Lessons Learned

* Validate configuration with `promtool` before reloading the service.
* A local health check confirms the server is running, while **Status → Targets** confirms end-to-end scrape health.
* Node Exporter and Prometheus provide metrics; Uptime Kuma remains the independent availability and notification layer.
