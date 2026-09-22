---
title: "Replicated storage"
description: "Building a Ceph cluster across converged nodes, and the replicated-pair alternative."
status: published
---

Replicated storage is what makes a set of servers into hyperconverged infrastructure. Disks in
each node are pooled; data is replicated across nodes; every host can reach every image.

A VM whose disks are on replicated storage can live-migrate anywhere and be restarted anywhere
after a host failure.

## Ceph — the normal choice

Three or more converged nodes contribute their disks. The platform bootstraps the cluster,
creates the pool VM disks live in, and distributes the access secret to every compute host.

That secret is distributed with an **identical identifier everywhere**, which is what lets live
migration work: the receiving host can open the same image with the same credentials mid-move.

### Before you build

Run the storage **preflight** probe from a host's storage screen. It checks the hardware is fit
and refuses the build with an actionable message if it is not.

Requirements:

| | |
|---|---|
| Nodes | Three or more, converged |
| Disks | Whole disks, unpartitioned, not otherwise in use |
| Network | A storage network, ideally on its own uplink |

> **Warning** — Disks contributed to a storage cluster are **claimed whole and wiped**. Do not
> point it at a disk with anything on it.

### Building it

**Storage → Clusters → Create**. Choose the nodes and the disks each contributes.

The build runs as a [task](/console/tasks/) and takes minutes: bootstrapping, then creating a
storage daemon per disk. Watch the task rather than the screen.

### Adding capacity

Add disks to existing nodes, or add nodes. The cluster rebalances by itself — data is moved so
that it is spread across the new shape.

Rebalancing is heavy I/O for as long as it takes. Do it when the cluster is not busy, and put
replication on [its own uplink](/networking/distributed-switches/) so that the rebuild does not
degrade the storage it is rebuilding.

### Health

The cluster reports its health, its storage daemons, its pools and its images. Health is the
number to watch: a cluster that is repairing is working but has spent its redundancy.

Set an alert on it. See [Alerts](/monitoring/alerts/).

### Using it

A VM disk backed by replicated storage is an image in the cluster's pool. Create a new one with
the disk, or attach an existing image.

Images can be resized from the storage screen; the guest still has to extend its filesystem.

### Attaching a cluster you already run

An existing Ceph cluster the platform did not build can be attached. The platform cannot derive
its monitors or its keyring, so you supply them — everything asked for is readable from that
cluster's own configuration.

## A replicated pair

Where there are exactly two storage nodes, the alternative is a mirrored pair with a storage VIP
that follows whichever node is active.

The platform renders the configuration and drives the tooling, but **it does not perform the
failover**. A cluster resource manager owns promotion and the VIP. The platform configures and
observes.

That separation is deliberate: two things that can both decide which node is primary is exactly
how a split brain happens.

### Fencing is mandatory

> **Warning** — Fencing is **verified before any change is made**. No fencing, no cluster.
> This is not a check you can skip. Two nodes that can each believe they are primary will
> both write, and the data they were protecting is then gone.

### The initial sync

Building the pair copies everything from one node to the other. On real disks that is **hours**.
It runs as a task and the cluster is usable in a degraded state while it proceeds.

## Choosing

| | Ceph | Replicated pair |
|---|---|---|
| Nodes | 3+ | Exactly 2 |
| Scaling | Add disks or nodes | Fixed |
| Failure tolerance | Configurable | One node |
| Failover | Inherent | Cluster resource manager |
| Fencing | Not required in the same way | **Mandatory** |

With three or more nodes, use Ceph. The pair exists for deployments that have two storage nodes
and need them redundant.

## Keeping storage off the management link

By default storage addresses resolve to the hosts' management addresses, so storage I/O shares a
link with the control plane — and a management problem becomes a storage outage.

A storage distributed switch gives every host an address on a storage subnet and moves that
traffic. A replication switch on a dedicated uplink goes further and separates rebuild traffic
from client I/O physically.

See [Distributed switches](/networking/distributed-switches/).
