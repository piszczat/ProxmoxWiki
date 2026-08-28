# Services

## LXC 100 — Plex

Media server workload. Media itself is stored separately from the Proxmox host where possible.

## LXC 101 — Omada Controller

Central management for TP-Link Omada access points and related network configuration.

## LXC 102 — AdGuard Home

Provides DNS filtering, local DNS records and name resolution for homelab services.

## LXC 103 — Uptime Kuma

Monitors internal and public endpoints. Telegram notifications are configured for important service outages.

## LXC 104 — Homepage

Dashboard / landing page for homelab services.

## LXC 105 — Nginx Proxy Manager

Reverse proxy for internal services and TLS termination. Public website traffic does not depend on this container because the website uses Cloudflare Tunnel.

## LXC 106 — Komodo

Used for container / server management and visibility across selected homelab workloads.

## LXC 107 — Vaultwarden

Self-hosted password manager. Public sign-up is disabled after initial account creation.

## LXC 108 — Website

Runs the production portfolio website. The application is managed by systemd and listens locally on its application port. Cloudflare Tunnel provides the public path to the service.

Deployment is performed from GitHub after successful CI validation.

## LXC 109 — GitHub Runner

Dedicated self-hosted GitHub Actions runner used only for trusted deployment jobs. General CI validation remains on GitHub-hosted runners.

The runner connects to the Website container through restricted SSH and performs the production update there.

## VM 200 — Home Assistant

Dedicated Home Assistant virtual machine for home automation workloads.

## Operational convention

The Proxmox host should remain as clean as possible. Application services belong in guests rather than being installed directly on the hypervisor.
