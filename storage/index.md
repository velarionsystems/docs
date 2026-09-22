---
title: "Storage"
description: "How VS-HCI stores VM disks — local pools, replicated clusters, and exported LUNs."
status: published
---

Storage in VS-HCI comes in layers, and the one you choose for a VM's disks decides whether that
VM can move between hosts.

## The choice that matters

| Backing | Reachable from | VM can migrate |
|---|---|---|
| A **pool** on one host | That host only | No |
| A **replicated storage** image | Every host | Yes |
| An **iSCSI LUN** | Every host with a session | Yes |

A VM on local disk is pinned to its server. It cannot live-migrate, and high availability
cannot restart it elsewhere, because its disk exists in one place. That is fine for something
you can rebuild and wrong for anything you cannot.

See [Live migration](/vms/live-migration/).

## Storage modes

Three ways to provide shared storage, chosen per deployment:

### Replicated storage — Ceph

Three or more converged nodes pool their disks into a single cluster. Data is replicated across
nodes, every host can reach every image, and losing a node loses no data.

This is the normal choice for VS-HCI, and the one that makes the platform hyperconverged rather
than a set of servers with local disks.

See [Replicated storage](/storage/replicated/).

### A replicated pair

Two storage nodes mirroring each other, with a storage VIP that follows whichever node is
active. Failover is managed by a cluster resource manager rather than by the platform.

Suited to a deployment that has exactly two storage nodes and needs them redundant.

> **Warning** — A replicated pair requires working **fencing**, and the platform verifies it
> before making any change. No fencing, no cluster. Two nodes that can both believe they are
> primary will corrupt the data they are protecting.

### Standalone

Local pools on each host, no replication. Appropriate for a single-node deployment, for
scratch, and for workloads you can rebuild.

## Preflight

Before any storage mode is built, a **preflight probe** examines the hardware and reports
whether it is fit. Each mode has its own gates, and an unfit deployment is refused with a
message saying what is wrong.

The intent is to fail at onboarding rather than during a failover at three in the morning. Run
it from a host's storage screen.

## Pools

A pool is storage on one host that VM disks live in.

| Type | What it is |
|---|---|
| `DIR` | A directory on the host's filesystem |
| `LVM` | An LVM volume group |
| `ZFS` | A ZFS pool |
| `NFS` | A mounted NFS export |
| `ISCSI` | An attached iSCSI target |
| `CEPH` | A replicated storage pool |

Pool status is `ACTIVE`, `INACTIVE`, `BUILDING`, `DEGRADED` or `ERROR`. See
[Pools](/storage/pools/).

## Exports

The platform can also *serve* storage: iSCSI targets exported from storage nodes, consumed by
hosts and by machines outside the deployment. One VM disk is one LUN.

See [Exports](/storage/exports/).

## The ISO library

One library the whole fleet sees at the same path, so a VM definition naming an ISO stays valid
wherever the VM runs. See [The ISO library](/storage/iso-library/).

## Storage on its own network

By default, storage would share the management link — and a management problem would become a
storage outage. A storage distributed switch gives every host a second address and moves storage
traffic onto it.

Replication is worth separating again: a rebuild running at line rate for hours degrades the
storage it is rebuilding. A replication switch on a dedicated uplink puts it on different wires
entirely.

See [Distributed switches](/networking/distributed-switches/).

## Where to look

| Question | Page |
|---|---|
| What pools does this host have? | [Pools](/storage/pools/) |
| How do I add a disk to a VM? | [Volumes](/storage/volumes/) |
| How do I build shared storage? | [Replicated storage](/storage/replicated/) |
| How do I serve storage out? | [Exports](/storage/exports/) |
| Where do ISOs live? | [The ISO library](/storage/iso-library/) |
