# Prometheus

## Installation

Installed from Debian packages.

```bash
apt install -y curl wget vim htop net-tools unzip
useradd prometheus
passwd prometheus
mkdir /etc/prometheus
mkdir /var/lib/prometheus
chown prometheus:prometheus /etc/prometheus
chown prometheus:prometheus /var/lib/prometheus
apt policy prometheus
apt install -y prometheus
systemctl start prometheus
systemctl status prometheus
```

Validation/Troubleshooting

Log check for errors`
```bash
journalctl -u prometheus -n 100 --no-pager | grep panic
```

Validate local Prometheus server is presenting metrics
```bash
curl -X POST http://localhost:9090/-/reload
```

## Current Targets

- pve - physical node - Proxmox
- infra-prometheus01 - LXC - Debian 13
- infra-grafana01 - LXC - Debian 13
- lab-kali01 - VM - Kali Linux
- lab-core01 - VM - Debian 13

## Validation

http://localhost:9090 - Local prometheus LXC
http://{host-ip}:9100 - Remote targets running node exporter

Status → Targets

Should show all targets UP.

## Configuration

Location

/etc/prometheus/prometheus.yml

``` yaml
# Sample config for Prometheus.

global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'example'

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.

#
# Prometheus Server
#
  - job_name: 'prometheus'

    scrape_interval: 5s
    scrape_timeout: 5s
    static_configs:
      - targets: ['localhost:9090']

#
# Proxmox Hypervisor
#

  - job_name: proxmox
    static_configs:
      - targets: ['192.168.x.x:9100']
        labels:
          name: pve

#
# Linux Virtual Machines
#

  - job_name: linux-vm
    static_configs:
      - targets: ['192.168.x.x:9100']
        labels:
          name: lab-core01

      - targets: ['192.168.x.x:9100']
        labels:
          name: lab-kali01

#
# Linux Containers (LXC)
#

  - job_name: linux-lxc
    static_configs:
      - targets: ['192.168.x.x:9100']
        labels: 
          name: infra-grafana01
```
