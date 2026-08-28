# Homelab Architecture

## Overview

The environment uses Proxmox VE as the virtualization platform. Services are separated into dedicated LXC containers where practical, with a VM used where a full virtual machine provides better appliance compatibility.

```text
Internet
   |
ISP modem / router path
   |
TP-Link gateway
   |
Managed LAN / VLANs
   |
Proxmox VE host
   |
   +-- LXC 100 Plex
   +-- LXC 101 Omada Controller
   +-- LXC 102 AdGuard Home
   +-- LXC 103 Uptime Kuma
   +-- LXC 104 Homepage
   +-- LXC 105 Nginx Proxy Manager
   +-- LXC 106 Komodo
   +-- LXC 107 Vaultwarden
   +-- LXC 108 Website
   +-- LXC 109 GitHub Runner
   +-- VM  200 Home Assistant

Synology NAS
   +-- Proxmox backup target
   +-- Media / personal storage
```

## Virtualization model

Most lightweight services run in unprivileged Debian LXC containers. Containers are configured to start automatically with the Proxmox host. Nesting is enabled where Docker or similar container workloads are required.

Home Assistant runs as a dedicated VM, keeping the appliance-style installation isolated from the general-purpose Linux containers.

## Service separation

Services are intentionally separated rather than installed directly on the Proxmox host. This gives:

- smaller failure domains;
- easier backup and restore;
- independent upgrades and reboots;
- easier resource allocation;
- simpler troubleshooting;
- a clearer security boundary between services.

## Public vs internal services

Internal administration interfaces remain on the private network. Public access is provided selectively. The portfolio website uses Cloudflare Tunnel, so no direct inbound web port forwarding is required for the site.

Nginx Proxy Manager is retained for internal HTTPS / reverse-proxy use where appropriate.

## Security note

This public repository deliberately omits real internal addressing, credentials, tokens, SSH private keys and tunnel secrets. Environment-specific values should be stored in the relevant platform configuration, secret store or GitHub repository variables rather than committed to Git.
