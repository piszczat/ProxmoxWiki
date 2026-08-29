# Backup audit

This page tracks what should be protected in the homelab and what still needs to be verified on the live Proxmox host.

> This repository is public. Internal IP addresses, credentials, tokens, private keys and other environment-specific values are intentionally omitted.

## Current Proxmox backup design

Proxmox uses a NAS-backed NFS storage target for guest backups.

Verified live configuration:

- Backup job is enabled.
- Backup storage points to the Synology-backed storage target.
- Schedule: daily at 02:30.
- Backup mode: snapshot.
- Compression: ZSTD.
- Retention: keep last 7 backups.
- Selection mode: all guests.
- CT 100-109 are included.
- VM 200 is included.
- Plex has an external bind mount to the NAS; this mount is not a Proxmox-managed volume and is therefore not included in the guest backup. This is expected because the media itself already resides on the NAS.

The live host should still be checked after major topology changes, but with selection mode set to all, newly added guests are expected to be picked up automatically unless explicitly excluded.

## Guest inventory

| ID | Guest | Type | Backup priority | Notes |
|---:|---|---|---|---|
| 100 | Plex | LXC | Medium | Root filesystem is backed up. External NAS media bind mount is intentionally outside the Proxmox backup. |
| 101 | Omada Controller | LXC | High | Network controller configuration should be recoverable quickly. |
| 102 | AdGuard Home | LXC | High | DNS service is important for normal LAN operation. |
| 103 | Uptime Kuma | LXC | Medium | Monitor definitions and notification config should be preserved. |
| 104 | Homepage | LXC | Low | Mostly convenience / dashboard state. |
| 105 | Nginx Proxy Manager | LXC | High | Internal proxy hosts and certificate state are important. |
| 106 | Komodo | LXC | Medium | Management configuration should be backed up. |
| 107 | Vaultwarden | LXC | Critical | Database and attachments are critical; restore must be tested. |
| 108 | Website | LXC | Medium | Application source is in GitHub, but host configuration, service unit and tunnel configuration still matter. |
| 109 | GitHub Runner | LXC | Medium | Runner can be rebuilt, but SSH trust and service setup must be recreated. |
| 200 | Home Assistant | VM | Critical | Automations, integrations and device state/configuration are high-value. |

## Configuration backups beyond Proxmox snapshots

Guest-level Proxmox backups are useful, but selected services should also have application-level exports or documented rebuild procedures.

### Critical application-level backups

- **Vaultwarden**
  - database
  - attachments
  - configuration / environment values stored privately
  - restore procedure tested periodically

- **Home Assistant**
  - native Home Assistant backup
  - copy of backups stored outside the VM

- **Omada Controller**
  - periodic controller configuration export

- **Nginx Proxy Manager**
  - database / application data
  - certificate configuration where applicable

- **AdGuard Home**
  - configuration export / config file backup

### Rebuild-oriented services

The following can usually be restored either from a Proxmox guest backup or rebuilt from documentation:

- Homepage
- Komodo
- Uptime Kuma
- GitHub Runner
- Website application runtime

## Backup audit checklist

The following has been verified directly on the Proxmox host:

- [x] Scheduled backup job exists and is enabled.
- [x] Backup target points to the Synology-backed storage.
- [x] Backup job selection includes CT 100-109 and VM 200.
- [x] Snapshot mode is enabled.
- [x] ZSTD compression is enabled.
- [x] Retention is set to keep the last 7 backups.
- [ ] The most recent run completed successfully.
- [ ] Every guest has at least one recent backup on the NAS.
- [ ] NAS free space is sufficient for retention growth.
- [ ] A restore test has been performed for at least one non-critical guest.
- [ ] A restore test has been performed for Vaultwarden or Home Assistant using a safe isolated procedure.

## Recommended restore-test cadence

- Monthly: verify backup job status and storage capacity.
- Quarterly: restore one disposable/non-critical LXC to a temporary ID and confirm it boots.
- Every 6 months: test recovery of one critical application using an isolated restore.
- After major infrastructure changes: confirm all new guests are included in the backup job.

## Important limitation

A backup that has never been restored is not yet a proven recovery path. The goal is not only to create archives, but to verify that the homelab can actually be rebuilt from them.
