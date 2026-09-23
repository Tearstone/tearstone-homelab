# infra-vpn01

## Purpose

`infra-vpn01` is the dedicated Tailscale subnet router for remote access to the home lab. It allows approved household devices to reach the private LAN without installing Tailscale on every internal system.

The subnet router is intentionally deployed as a small, unprivileged Debian LXC rather than adding the remote-access role to an application or monitoring host.

## Deployment

| Setting | Value |
| --- | --- |
| Proxmox ID | CT 205 |
| Hostname | `infra-vpn01` |
| Proxmox node | `pve01` |
| Platform | Proxmox LXC |
| Operating system | Debian GNU/Linux 13 (Trixie) |
| CPU | 1 core |
| Memory | 512 MB |
| Swap | 512 MB |
| Root disk | 4 GB |
| Container security | Unprivileged |
| Nesting | Enabled |
| `onboot` | Enabled |
| IPv4 address | `<VPN_HOST_IP>/<PREFIX>` |
| Gateway | `<LAN_GATEWAY>` |
| DNS server | `<INTERNAL_DNS_SERVER>` |
| Search domain | `<INTERNAL_SEARCH_DOMAIN>` |

## Tailscale

Tailscale 1.102.4 is installed directly in the LXC. The node advertises the home LAN as a subnet route:

```text
<LAN_CIDR>
```

The advertised route was approved in the Tailscale administration console. Full access to the routed LAN is intentional for the two enrolled household phones.

## TUN Device Passthrough

Tailscale requires access to `/dev/net/tun`. The following entries are present in the Proxmox LXC configuration for CT 205:

```text
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

These entries pass the host TUN device into the unprivileged container without converting the LXC to a privileged container.

## IPv4 Forwarding

IPv4 forwarding is enabled persistently inside the LXC so traffic received from the Tailscale interface can be routed to the LAN after a reboot.

```text
net.ipv4.ip_forward=1
```

## Validation

Remote access was validated from a phone using cellular service rather than the home Wi-Fi network. The test confirmed access through the subnet route to:

* Prometheus
* Uptime Kuma
* Immich

Two household phones are enrolled in the tailnet. Both are intentionally permitted to reach the full home LAN through `infra-vpn01`.

## Monitoring and Operations

`prometheus-node-exporter` is installed on `infra-vpn01`, allowing Prometheus to collect operating-system metrics on TCP port 9100.

Uptime Kuma monitors the LXC with an ICMP Ping monitor. Homepage includes a Tailscale tile under the Management group for access to the Tailscale administration interface.

## Security Notes

* The LXC remains unprivileged.
* Only the TUN character device required by Tailscale is passed through.
* Route advertisement requires explicit approval in the Tailscale administration console.
* The current full-LAN access policy is a deliberate household trust decision, not an accidental default.
* If less-trusted devices are enrolled later, Tailscale ACLs or grants should be introduced before those devices receive routed LAN access.
