# Operations and Recovery

## Normal checks

Useful checks after maintenance, host reboot or an unexpected outage:

1. confirm the Proxmox host is reachable;
2. confirm expected LXC containers and VMs are running;
3. verify AdGuard DNS resolution;
4. verify Omada Controller and wireless access points;
5. verify reverse proxy and tunnel-backed services;
6. verify Uptime Kuma monitoring;
7. verify NAS connectivity and backup storage;
8. verify public services from outside the LAN where appropriate.

## Guest restart strategy

Application workloads should normally be restarted at the guest or service level rather than rebooting the complete Proxmox host.

For systemd-managed services:

```bash
sudo systemctl status <service>
sudo systemctl restart <service>
```

For Docker-based workloads, use the compose / container management method documented for that guest.

## After adding a new guest

Check the following:

- Start at boot is enabled if required.
- Static / reserved addressing is configured where the service requires a predictable address.
- DNS name is created if needed.
- Monitoring is added to Uptime Kuma.
- Backups include the new guest.
- Required storage mounts survive reboot.
- Firewall and VLAN access are no broader than necessary.
- Documentation is updated in this repository.

## Recovery order

For a large outage, restore foundational services before application services:

```text
Network / gateway
      ↓
Proxmox host
      ↓
DNS (AdGuard)
      ↓
Network management (Omada)
      ↓
Storage / NAS connectivity
      ↓
Reverse proxy / tunnels
      ↓
Application services
      ↓
Monitoring
```

The precise order can vary depending on which components remain available.

## Secrets

Do not add the following to this repository:

- passwords;
- API tokens;
- GitHub runner registration tokens;
- Cloudflare tunnel credentials;
- VPN credentials;
- SSH private keys;
- Vaultwarden secrets;
- public IP history;
- detailed private network topology unless intentionally sanitized.
