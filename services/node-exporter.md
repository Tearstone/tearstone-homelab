# Node Exporter

## Purpose

Node Exporter exposes Linux host metrics for collection by Prometheus. It provides CPU, memory, filesystem, network, load, and system-uptime data on TCP port 9100.

## Installation

Install the Debian package and enable the service:

```bash
apt update
apt install -y prometheus-node-exporter
systemctl enable --now prometheus-node-exporter
```

## Configuration

The Debian package supplies the systemd service and default collector configuration. No service-specific credentials are required.

Add each host to the appropriate sanitized target group in `/etc/prometheus/prometheus.yml`:

```yaml
- job_name: linux-lxc
  static_configs:
    - targets: ['<NODE_EXPORTER_HOST>:9100']
      labels:
        name: '<HOSTNAME>'
```

Reload Prometheus after validating the configuration.

## Validation

Verify the service and local metrics endpoint:

```bash
systemctl status prometheus-node-exporter --no-pager
ss -lntp | grep ':9100'
curl http://localhost:9100/metrics
```

In Prometheus, open **Status → Targets** and confirm the host reports `UP`.

## Monitoring and Integrations

Node Exporter is installed across the monitored Linux hosts, including `infra-vpn01`. Prometheus scrapes each instance on TCP port 9100, and Grafana visualizes the collected metrics.

## Security

* Node Exporter provides metrics without application authentication; TCP port 9100 should remain restricted to the trusted monitoring network.
* Private host addresses are represented by placeholders in public examples.
* No credentials or private network identifiers belong in this repository.

## Lessons Learned

* The Debian package provides a simple, repeatable installation and systemd integration.
* The local metrics endpoint should be verified before adding a host to Prometheus.
* Prometheus target state confirms end-to-end reachability beyond the local service check.
