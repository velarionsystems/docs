---
title: "Pools"
description: "Per-host storage pools — the types, creating them, and what they can and cannot do."
status: published
---

A pool is storage on **one host**. VM disks created in it live on that server's disks.

Pools are per-host by nature. That is their limitation and their purpose: they are fast, simple
and local.

## Types

| Type | Backed by | Notes |
|---|---|---|
| `DIR` | A directory on the host filesystem | The simplest. Disk images as files |
| `LVM` | An LVM volume group | Logical volumes as disks; less overhead than files |
| `ZFS` | A ZFS pool | Checksums, compression, snapshots at the filesystem |
| `NFS` | A mounted NFS export | Shared, if the export is reachable from several hosts |
| `ISCSI` | An attached iSCSI target | Shared block storage |
| `CEPH` | A replicated storage pool | Shared; see [Replicated storage](/storage/replicated/) |

## Status

| Status | Meaning |
|---|---|
| `ACTIVE` | Available |
| `INACTIVE` | Defined but not started |
| `BUILDING` | Being constructed |
| `DEGRADED` | Working with a fault — a failed disk in a mirror |
| `ERROR` | Unusable |

A `DEGRADED` pool still serves data and is a warning, not an outage. Treat it as urgent anyway:
degraded means the redundancy you were relying on is already spent.

## The default pool

The installer creates a local pool at `/var/lib/hyperion/vms` on its own logical volume. VMs
created without a specified backing land there.

It is a `DIR` pool on local disk, so anything in it is pinned to that host.

## Creating a pool

From a host's **Storage** tab in [Infrastructure](/console/infrastructure/).

**Directory pool** — give it a name and a path.

**LVM pool** — choose from the host's available disks. The screen lists disks that are genuinely
free: not mounted, not already in a pool, not part of a mirror.

> **Warning** — Building a pool on a disk **destroys what is on it**. The available-disks list
> excludes disks in obvious use, but it cannot know that a disk you attached last week has
> something you want.

## Software mirrors

A host can mirror disks with a software RAID device, and build a pool on the result. Members can
be added and removed, and the mirror can be torn down.

This gives redundancy against a single disk failing **within** a host. It is not a substitute for
replication across hosts: a mirror does not survive the server.

## Capacity

Each pool reports capacity, allocated and available. Usage is allocated against capacity.

Watch allocated rather than used. Thin-provisioned disks can be allocated well beyond what is
physically present, and a pool that runs out of real space while its guests believe they have
room fails in ways the guests handle badly.

## Syncing

Pools can be re-read from the host, which reconciles what the platform believes with what is
actually there. Use it after changing storage outside the platform, or when a pool's reported
state looks wrong.

## Choosing a pool for a VM disk

The create-VM form offers the pools on the host or zone the VM will land on.

Choose a local pool when:

- the VM is disposable, or rebuilt from configuration,
- you want the lowest latency available,
- the deployment has no shared storage.

Choose shared backing when the VM needs to migrate or be restarted elsewhere — which is most
production workloads.

## Moving a VM off a pool

A VM's disks can be moved between backings without recreating it. Use it to free a pool you
intend to remove, or to make a pinned VM migratable.

It copies data and takes as long as the data takes. See
[Live migration](/vms/live-migration/).
