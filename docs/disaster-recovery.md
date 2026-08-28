# Disaster recovery plan

This document describes the intended recovery order for the homelab after a major failure such as Proxmox host loss, storage corruption, or replacement of the physical host.

> This repository is public. Internal addresses, credentials, private keys and secrets are deliberately omitted.

## Recovery objectives

The recovery strategy prioritizes services that the rest of the environment depends on.

General order:

```text
Proxmox host
  ↓
NAS / backup storage connectivity
  ↓
Network management + DNS
  ↓
Reverse proxy / certificates
  ↓
Critical applications
  ↓
Monitoring / management
  ↓
Public website deployment tooling
  ↓
Convenience services
```

## Phase 1 — Recover the Proxmox host

1. Install a supported Proxmox VE release on the replacement host.
2. Configure the management network.
3. Confirm access to the Proxmox web interface and shell.
4. Apply updates before restoring workloads where practical.
5. Recreate or import the NAS-backed backup storage.
6. Confirm Proxmox can read the backup archives before attempting restores.

Do not start by rebuilding individual applications manually if valid Proxmox backups are available.

## Phase 2 — Restore infrastructure dependencies

Recommended order:

### 1. Omada Controller — CT 101

Restore early so network and access-point management is available.

Validation:

- controller starts
- managed APs reconnect
- VLAN / WLAN configuration is present

### 2. AdGuard Home — CT 102

DNS is a dependency for many local services.

Validation:

- DNS queries succeed from the LAN
- local DNS records / rewrites are present
- upstream DNS resolution works

If AdGuard cannot be restored immediately, temporarily configure clients/router to use a known working upstream DNS resolver until recovery is complete.

### 3. Nginx Proxy Manager — CT 105

Restore internal reverse-proxy functionality and TLS termination.

Validation:

- proxy host configuration is present
- internal HTTPS services resolve correctly
- certificates are valid

## Phase 3 — Restore critical applications

### Vaultwarden — CT 107

Priority: Critical.

Validation should include more than simply reaching the login page:

- service starts
- database opens successfully
- user can authenticate
- existing vault entries are visible
- attachments work if used

Do not overwrite the only known-good backup while testing recovery.

### Home Assistant — VM 200

Priority: Critical.

Validation:

- VM boots normally
- Home Assistant starts
- integrations reconnect
- automations are present
- critical devices/entities are available

Where possible, maintain Home Assistant native backups in addition to VM-level backups.

## Phase 4 — Restore observability and management

Recommended order:

- Uptime Kuma — CT 103
- Komodo — CT 106
- Homepage — CT 104

Once Uptime Kuma is online, use it to identify remaining failed services instead of testing everything manually.

## Phase 5 — Restore public website infrastructure

### Website — CT 108

The application source of truth is GitHub, so the website can be rebuilt even if the guest backup is unavailable.

A restored or rebuilt website guest needs:

- supported Node.js runtime
- Git
- application checkout
- production dependencies
- systemd service
- Cloudflare Tunnel client and private tunnel credentials
- application listening on the expected local port

Validation:

- local health check succeeds
- systemd service is active
- Cloudflare Tunnel reports healthy
- public HTTPS endpoint returns the expected status

### GitHub Runner — CT 109

The deployment runner can be rebuilt rather than restored if needed.

Required private configuration includes:

- GitHub runner registration
- SSH credentials used for trusted deployment
- known-host verification
- service registration

Never store registration tokens or private SSH keys in this public documentation repository.

## Phase 6 — Restore media and convenience services

### Plex — CT 100

Plex is lower recovery priority than DNS, password management and home automation.

If media itself lives on separate NAS storage, focus recovery on:

- Plex application configuration
- library mappings
- metadata where worth preserving

### Remaining convenience services

Restore any optional dashboards or tools after core functionality is stable.

## Complete host-loss recovery checklist

- [ ] Proxmox installed and updated.
- [ ] Management networking functional.
- [ ] NAS reachable from Proxmox.
- [ ] Backup storage visible.
- [ ] Omada restored and healthy.
- [ ] AdGuard restored and answering DNS queries.
- [ ] Nginx Proxy Manager restored.
- [ ] Vaultwarden restored and contents verified.
- [ ] Home Assistant restored and integrations checked.
- [ ] Uptime Kuma restored.
- [ ] Komodo restored.
- [ ] Website restored/rebuilt and public health check succeeds.
- [ ] GitHub runner restored/rebuilt and deployment tested.
- [ ] Plex restored.
- [ ] Homepage restored.
- [ ] Backup schedule recreated and verified.
- [ ] A fresh post-recovery backup completed successfully.

## Disaster scenarios

### Single LXC failure

Restore only the affected guest to its original ID if safe. Validate application state before deleting any failed copy.

### Proxmox OS failure but disks intact

Prefer recovering Proxmox configuration or reinstalling the host while preserving guest storage where possible. Do not format storage until existing data has been assessed.

### Complete host loss

Follow the phased recovery order in this document using NAS backups.

### NAS backup loss

This is the major remaining risk if the NAS is the only backup destination. Important application data should eventually have a second copy outside the Proxmox host and outside the primary NAS failure domain.

A future improvement should implement a 3-2-1-style backup strategy for critical data.

## Recovery principle

Restore dependencies before applications. A technically successful VM restore is not useful if DNS, networking or required storage dependencies are still unavailable.
