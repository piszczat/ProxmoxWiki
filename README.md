# ProxmoxWiki

Documentation for my Proxmox-based homelab, including architecture, services, networking, backups, monitoring and deployment workflows.

> This repository is public. Internal IP addresses, credentials, tokens, private keys and other environment-specific secrets are intentionally not stored here.

## Current architecture

The homelab is built around a Proxmox VE host running a mixture of LXC containers and virtual machines.

### Core services

| ID | Type | Service | Purpose |
|---:|---|---|---|
| 100 | LXC | Plex | Media server |
| 101 | LXC | Omada Controller | Network / AP management |
| 102 | LXC | AdGuard Home | DNS filtering and local DNS |
| 103 | LXC | Uptime Kuma | Service monitoring |
| 104 | LXC | Homepage | Homelab dashboard |
| 105 | LXC | Nginx Proxy Manager | Internal reverse proxy and TLS |
| 106 | LXC | Komodo | Container / server management |
| 107 | LXC | Vaultwarden | Password manager |
| 108 | LXC | Website | Production portfolio website |
| 109 | LXC | GitHub Runner | Self-hosted deployment runner |
| 200 | VM | Home Assistant | Home automation |

## Documentation

- [Architecture](docs/architecture.md)
- [Networking](docs/networking.md)
- [Services](docs/services.md)
- [Backups](docs/backups.md)
- [Website deployment](docs/website-deployment.md)
- [Operations and recovery](docs/operations.md)

## Design principles

- Public services are exposed through secure tunnels or reverse proxies rather than direct inbound access where possible.
- Internal DNS is handled centrally.
- Infrastructure services start automatically after host reboot.
- Production deployment uses CI validation before deployment.
- The self-hosted GitHub runner is used only for trusted deployment workloads.
- Backups are stored separately from the Proxmox host.
- Secrets and internal addressing are kept outside this public repository.

## Release path for the portfolio website

```text
dev
  ↓ Pull Request + CI
test
  ↓ Pull Request + CI
main
  ↓ successful CI
self-hosted GitHub runner
  ↓ SSH deployment
website LXC
  ↓ health check
marcinp.com
```

More detail is available in [Website deployment](docs/website-deployment.md).
