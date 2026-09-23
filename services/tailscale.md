# Tailscale

## Purpose

Tailscale provides authenticated remote access to the home lab without exposing individual internal services directly to the internet.

The service runs on `infra-vpn01`, a dedicated Debian 13 unprivileged LXC on `pve01`. Tailscale 1.102.4 advertises the private LAN as a subnet route, allowing approved tailnet devices to reach internal services.

## Installation

### Proxmox host configuration

Stop CT 205 before changing its LXC configuration:

```bash
pct stop 205
```

Add the following entries to `/etc/pve/lxc/205.conf`:

```text
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Confirm that nesting and start-at-boot are enabled, then start the container:

```bash
pct set 205 --features nesting=1 --onboot 1
pct start 205
```

### Debian LXC installation

Run the following commands inside `infra-vpn01`:

```bash
apt update
apt install -y curl ca-certificates
curl -fsSL https://tailscale.com/install.sh | sh
systemctl enable --now tailscaled
tailscale version
```

The completed deployment reports Tailscale 1.102.4.

Install Node Exporter for host-level Prometheus monitoring:

```bash
apt install -y prometheus-node-exporter
systemctl enable --now prometheus-node-exporter
```

## Configuration

### Persistent IPv4 forwarding

Create `/etc/sysctl.d/99-tailscale.conf`:

```text
net.ipv4.ip_forward=1
```

Load and verify the setting:

```bash
sysctl --system
sysctl net.ipv4.ip_forward
```

### Subnet route advertisement

Authenticate the node and advertise the sanitized LAN placeholder:

```bash
tailscale up --advertise-routes=<LAN_CIDR>
```

Open the Tailscale administration interface, locate `infra-vpn01`, and approve the advertised subnet route. The actual private subnet is intentionally maintained outside this public repository.

The current policy intentionally grants full routed LAN access to two enrolled household phones. If the tailnet later includes devices outside that trust boundary, access should be narrowed with Tailscale ACLs or grants.

## Validation

Verify the service, TUN device, forwarding state, advertised route, and Node Exporter:

```bash
systemctl status tailscaled --no-pager
test -c /dev/net/tun
sysctl net.ipv4.ip_forward
tailscale status
tailscale debug prefs
systemctl status prometheus-node-exporter --no-pager
curl http://localhost:9100/metrics
```

Remote phone access was tested over cellular service to ensure traffic did not remain on the home Wi-Fi network. Prometheus, Uptime Kuma, and Immich were all reached successfully through the approved subnet route.

## Monitoring and Integrations

Prometheus scrapes `prometheus-node-exporter` on TCP port 9100 for operating-system metrics. Uptime Kuma independently monitors `infra-vpn01` using ICMP Ping.

Homepage includes a Tailscale tile in the Management group, providing a direct link to the Tailscale administration interface.

## Security

* The LXC remains unprivileged.
* Only the TUN character device required by Tailscale is passed into the container.
* Route advertisement requires explicit approval in the Tailscale administration interface.
* Exact private addresses, subnet ranges, gateway details, internal DNS information, credentials, and authentication URLs are excluded from this public repository.
* Full routed LAN access is an intentional trust decision for the two currently enrolled household phones.

## Lessons Learned

* Tailscale subnet routing works in an unprivileged LXC when `/dev/net/tun` is passed through explicitly.
* IPv4 forwarding must be persistent so routing survives a container restart.
* Cellular validation is necessary to prove the access path is remote rather than local Wi-Fi.
* Monitoring the subnet router through both Prometheus and Uptime Kuma provides separate performance and availability signals.
