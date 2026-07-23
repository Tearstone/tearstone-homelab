# Node Exporter

## Install

```bash
sudo apt install prometheus-node-exporter
```

## Verify

```bash
systemctl status prometheus-node-exporter
```

## Metrics

http://server:9100/metrics

## Lessons Learned

Node Exporter installed cleanly using the Debian package.

Verified listening on port 9100 before adding to Prometheus.
