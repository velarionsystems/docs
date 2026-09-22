---
title: "Live migration"
description: "Moving a running VM to another host — when it is lossless, when it is not, and what else it disturbs."
status: published
---

Live migration moves a running guest from one host to another with its memory copied across
while it runs. Done properly the guest does not notice.

It is how you empty a host for maintenance without a maintenance window.

## The four situations

Before offering you anything, the platform works out which situation the VM is in. A dialog
that just lists hosts would be asking the wrong question.

| Situation | What you are offered |
|---|---|
| **Zone-managed** | The zone picks the host. You consent; you do not choose |
| **Host choice** | The VM is outside a zone on shared storage — pick from the hosts that can take it |
| **Storage required** | The VM is on local disk. Nothing can take it until the disk moves to shared storage |
| **Blocked** | Nothing can be offered, and the reason is stated |

The screen also tells you whether the move is **lossless** — whether the guest keeps running
throughout — and if not, why not.

## Storage decides mobility

This is the rule that catches people out.

| Disk backing | Migratable |
|---|---|
| Replicated storage image | Yes — every host can reach it |
| iSCSI LUN | Yes — every host can reach it |
| Host storage pool (local) | **No** — the disk exists on one server only |
| Mixed | No — one local disk is enough to pin the VM |

A VM on local disk is not stuck forever: **move its storage first**. The platform offers the
targets its disks could move to, and once they are on shared storage the VM migrates normally.

That move copies data and takes as long as the data takes.

## What else the move disturbs

Inside an availability zone, fitting a guest onto a host can require moving other guests out of
the way — or powering off an [evictable](/vms/) one.

The platform works this out in advance and lists every affected workload **before** you confirm.
Read it. A migration that quietly stops somebody else's VM is not a migration you want to
discover from a task log afterwards.

## Performing a migration

From the console, open the VM and choose **Migrate**. From the CLI:

```
Hyperion[vsnode1]> vm migrate ubuntu1 to vsnode3
```

The move runs as a [task](/console/tasks/). The VM shows `MIGRATING` while it is in flight.

## What blocks a live migration

| Cause | Fix |
|---|---|
| Disks on local storage | Move storage to shared backing first |
| A PCI passthrough device | Cannot be migrated. Detach the device, or accept a cold move |
| Target host lacks capacity | Free capacity, or let the zone choose |
| Target host not `ONLINE` | Fix the host |
| Incompatible CPU features | The target must offer what the guest was started with |
| Hosts cannot hand the domain over | The hosts must trust each other; check host status |

PCI passthrough is the common permanent one. A device is physically in one server, so a guest
using it cannot move. That is the trade you accepted when you assigned it.

## Cold migration

A stopped VM moves trivially — there is no memory to copy and no guest to keep alive. The
platform treats that as lossless by definition.

If a running VM cannot move live and you can accept the downtime, stop it, move it, start it.

## Storage move

Moving a VM's disks between backings is its own operation, available whether or not you then
migrate the VM. Use it to:

- get a VM off local disk so it becomes migratable,
- move a VM onto replicated storage,
- evacuate a storage pool you intend to remove.

It copies data. Size the window accordingly.

## Emptying a host

For planned maintenance, put the host into maintenance mode rather than migrating guests one at
a time:

```
Hyperion[vsnode1]> host maintenance vsnode2 on
```

Nothing new is scheduled onto it, and its workloads move off. See
[Upgrading the fleet](/updating/fleet-upgrade/).
