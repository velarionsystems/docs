---
title: "Backup and restore"
description: "Backing up the control plane's configuration, and restoring it onto a rebuilt cluster."
status: published
---

This covers the **control plane's own database** — every definition in the deployment: hosts,
VMs, networks, storage, users, certificates and the secrets that make them work.

> **Warning** — These are **not** backups of your virtual machines' disks. For those, use
> [snapshot policies](/data-protection/snapshot-policies/) and
> [disaster recovery](/data-protection/disaster-recovery/).

## Why a database dump is not enough

It is worth understanding why the platform has its own archive format rather than leaving you to
dump the database.

Sensitive columns — host credentials, the cluster's own SSH identity, storage and BGP secrets —
are encrypted with a master key that is generated **per installation** and stored outside the
database. Restore a plain dump onto a controller with a different key and you get a cluster that
lists every host and cannot authenticate to any of them, unrecoverably.

Some per-node state is also deliberately excluded from replication between controllers, so no
peer holds a copy. Lose the node and the virtual IP, cluster identity and election priorities go
with it.

The archive captures all of it in one file.

## The archive

A single `.hbk` file:

- A **cleartext header** — cluster identity, timestamp, versions and an inventory. It stays in
  the clear so a restore picker can describe archives on a remote server without asking for a
  passphrase per row. It carries no secrets.
- An **optionally encrypted payload** — the full database, the master key and related secrets,
  the per-node cluster state, and a manifest with per-table row counts.

Encryption is a passphrase stretched with PBKDF2 into an AES-256-GCM key. A tampered or
truncated archive fails verification rather than reading as a short success.

## Where backups go

Three routes, one format:

| Route | Use |
|---|---|
| **Download** | Ad hoc, to your own machine |
| **Push** | Manually, to an SCP target |
| **Schedule** | Daily, weekly or monthly to that target |

### The target is proven before it is saved

Saving an SCP target does more than check the credentials. The platform connects, authenticates,
stats the directory, **writes a probe file and deletes it**.

A target that authenticates but lands on a read-only export or a full filesystem would fail at
two in the morning on the night it was needed. "Saved" therefore means "proven to accept a
write".

Uploads are written to a temporary name and renamed on completion, so an interrupted transfer
never looks like a usable backup.

### Scheduled backups require a passphrase

A scheduled backup **must** have a stored passphrase. The archive contains the master key;
writing it unencrypted to an off-cluster server would hand over the cluster to anyone who can
read the file.

The schedule is held as a next-run time rather than an in-memory timer, which means it can be
edited without a restart, a window missed to a restart or a failover is caught up rather than
skipped, and whichever controller is active runs it exactly once.

## Restoring

Restore is ordered so that no step destroys what an earlier step would need to undo it.

1. **Validate first.** Format, authentication tag, digest, required contents and schema
   compatibility — all while the live database is untouched.
2. **A safety archive of the current cluster** is taken, including the *current* master key.
   Restore is the one operation that destroys the evidence needed to reverse it. **If this
   fails, the restore is refused.**
3. **Replication is detached**, so a live apply worker cannot interleave pre- and post-restore
   rows.
4. **The database is replaced in a single transaction.** A failure rolls back and leaves the
   cluster exactly as it was, rather than producing a database that is neither one cluster nor
   the other.
5. **The master key is restored** and the controller restarts.
6. **The key is pushed to every peer controller**, which restart in turn.

Step 6 is not optional housekeeping. Without it, standbys keep their own keys while replicating
rows encrypted under the restored one — and the cluster looks healthy right up until a failover
promotes a controller that cannot decrypt a single credential.

### The schema gate

An **older** archive is fine; it is migrated forward on restart. A **newer** archive is
**refused** — the controller would find migrations it does not have and fail to start, leaving
you with a replaced database and a control plane that will not come up.

Restore onto the same version or newer. Never onto older.

### What is deliberately not restored

The local database's own credentials are captured for reference but not applied. Writing them
would leave the controller unable to reach its own database — turning a recovery into an outage.

## Inspecting an archive

An archive can be inspected before restoring, locally or on the remote target. The cleartext
header tells you which cluster it came from, when it was taken, what version produced it and
what it contains.

Check that before restoring. It is the cheapest step in the whole process.

## What to actually do

1. Configure an SCP target **off the cluster**. A backup on the cluster does not survive losing
   it.
2. Set a schedule with a stored passphrase, and record that passphrase somewhere that is not
   the cluster.
3. Check the history periodically — a schedule that stopped firing is silent.
4. **Test a restore.** An untested backup is a belief, not a capability.
