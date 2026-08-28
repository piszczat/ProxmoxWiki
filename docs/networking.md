# Networking

## Overview

The homelab network is built around a TP-Link gateway and Omada-managed wireless access points. Proxmox guests are connected to the LAN and selected VLANs according to service requirements.

## Logical layout

```text
Internet
   |
ISP modem in passthrough / modem mode
   |
TP-Link gateway
   |
   +-- Main LAN
   +-- VPN VLANs
   +-- IoT VLAN
   |
Omada access points
   |
Clients / homelab services
```

## DNS

AdGuard Home provides DNS filtering and local DNS resolution. Local hostnames use a private home namespace rather than public DNS records.

The gateway distributes AdGuard Home as the DNS server to clients through DHCP.

## Omada

The Omada Controller runs in its own LXC container and manages the wireless access points. The environment also uses separate VLANs / SSIDs for selected traffic such as IoT devices and VPN-routed networks.

## Service discovery

mDNS / Bonjour forwarding is enabled where needed so discovery-based services can work across selected VLAN boundaries, for example media and smart-home devices.

## Internal TLS

Nginx Proxy Manager provides reverse proxying and HTTPS for selected internal services. A wildcard certificate is used for the internal service naming scheme where appropriate.

## Public DNS and Cloudflare

The public domain is managed through Cloudflare. The production website is exposed through Cloudflare Tunnel instead of direct inbound NAT rules.

## Security notes

This public documentation does not contain real LAN addresses, public IP addresses, MAC addresses, VPN credentials, tunnel tokens or private DNS records.
