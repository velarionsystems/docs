---
title: "Virtual machines"
description: "The VM lifecycle, states, and what decides where a guest runs."
status: published
---

Virtual machines are KVM/QEMU guests. VS-HCI defines them, places them, and tells the host
agent what to run; the hypervisor on the host does the running.

## States

| State | Meaning |
|---|---|
| `CREATING` | Being built — disks provisioned, network attached |
| `RUNNING` | Executing |
| `STOPPED` | Powered off by someone |
| `PAUSED` | Execution suspended, memory still resident |
| `SUSPENDED` | State written out, memory released |
| `MIGRATING` | Moving between hosts |
| `CRASHED` | The guest died |
| `ERROR` | The platform could not do what was asked |
| `PENDING` | The platform is **not going to start it**, and says why |
| `DELETED` | Removed |
| `DR_PLACEHOLDER` | The replica of a VM that runs at the paired site |

Two of these are worth understanding properly.

**`PENDING` is not `STOPPED`.** A stopped VM is off because a person turned it off, and a
person can turn it back on. A pending VM is one the platform has declined to start — usually
because no host in its availability zone could take it, but also when the platform cannot act
safely: a host it could not fence, or a zone whose policy is to raise an alarm rather than
restart. Nothing restarts a `PENDING` VM automatically. It carries a reason; read it.

**`DR_PLACEHOLDER` cannot be started here.** It is the receiving end of a live mirror from the
paired site, with real disks, NICs and a capacity reservation in its zone. Starting it would
put a second writer on images the other site's hypervisor is still writing to. It becomes
startable only when a promotion moves ownership to this site. See
[Disaster recovery](/data-protection/disaster-recovery/).

## Lifecycle operations

From the console, the CLI or the API:

| Operation | Effect |
|---|---|
| Start | Power on |
| Stop | Graceful shutdown; force-stop if the guest ignores it |
| Pause / Resume | Freeze and unfreeze, memory stays resident |
| Reboot | Restart the guest |
| Migrate | Move to another host — see [Live migration](/vms/live-migration/) |
| Resize | Change CPU and memory — see [Resizing](/vms/resizing/) |
| Snapshot | Capture a point in time — see [Snapshots](/vms/snapshots/) |
| Delete | Remove the VM and, optionally, its disks |

```
Hyperion[vsnode1]> show vm
NAME     STATUS   VCPUS  RAM (MB)  HOST
drnode1  STOPPED  4      8192      vsnode1
drnode2  STOPPED  4      8192      vsnode2
drnode3  STOPPED  4      8192      vsnode3
ubuntu1  STOPPED  4      4096      vsnode3

Hyperion[vsnode1]> vm start ubuntu1
Hyperion[vsnode1]> vm migrate ubuntu1 to vsnode2
```

## Where a VM runs

This is the part that surprises people coming from a plain hypervisor.

**If the VM is in an availability zone, the zone picks the host.** You do not choose. The
scheduler is the only thing that knows how much capacity has been held back for a failover, so
letting an operator pin a guest to a host would quietly spend the headroom that keeps the zone
able to restart things.

**If the VM is not in a zone, you pick the host.** Deployments with no zones defined work this
way throughout.

You give a VM *either* an availability zone *or* a host when creating it — never both.

See [Availability zones](/high-availability/availability-zones/).

## Placement controls

| Control | What it does |
|---|---|
| **Anti-affinity group** | VMs in the same group are kept on different hosts. Use it for the members of a cluster you built yourself |
| **Evictable** | This VM may be powered off to make room for a more important one during a failover |
| **HA enabled** | Restart this VM elsewhere when its host fails |
| **HA priority** | The order in which HA-enabled VMs are restarted |
| **DR capable** | Replicate this VM to the paired site |
| **Auto-start** | Start when its host comes up |

Anti-affinity is the one most often forgotten. Three database replicas that all land on the
same server are three replicas of nothing.

## Disks and network

A VM's disks can be backed by:

- a **storage pool** on a host (local),
- an **iSCSI LUN**, either newly provisioned or attached (shared),
- an **RBD image** in a replicated storage cluster (shared).

What kind matters for migration: a VM on local disk cannot move to another host until its disks
move to shared storage first.

NICs attach either to a host bridge or to a distributed switch. See
[Networking](/networking/).

## Importing existing VMs

Two routes for guests that already exist:

- **Discover** — scan a host for VMs the platform did not create and adopt them.
- **Import** — bring in a VM definition.

Adopted VMs behave like any other afterwards, but check their disk backing: one on local disk
will not migrate until you move it.

## Editing the definition

Most properties are edited from the VM's screen. For anything the console does not expose, the
underlying libvirt domain XML can be read and replaced directly.

> **Warning** — Editing domain XML by hand bypasses every check the platform makes. It is a
> supported escape hatch, not a normal workflow, and a malformed domain will simply fail to
> start.
