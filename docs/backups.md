# Backups

## Backup target

Proxmox backups are stored on a Synology NAS over NFS rather than on the Proxmox host itself.

## Current approach

- Proxmox backup storage points to a dedicated NAS backup share.
- Backup mode uses snapshots where supported.
- Compression uses ZSTD.
- Backups run on a recurring schedule.
- Retention currently keeps seven restore points.
- The selection policy is intended to include all relevant guests.

## Why this layout

Keeping backups off-host protects against local storage failure and makes guest restore simpler after host replacement or rebuild.

## Restore principle

A normal guest restore should recreate the LXC container or VM from the latest known-good backup, followed by validation of:

1. network connectivity;
2. application service status;
3. mounted storage;
4. reverse proxy / tunnel path;
5. monitoring status;
6. application-specific data integrity.

## Important checks

Whenever a new VM or LXC is added, verify that it is included in the Proxmox backup job. A guest existing in Proxmox does not automatically guarantee that the intended backup policy covers it.

## Public documentation note

NAS paths, credentials, IP addresses and any recovery secrets are intentionally omitted from this public repository.
