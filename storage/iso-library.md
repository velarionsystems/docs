---
title: "The ISO library"
description: "One ISO library the whole fleet sees at the same path."
status: published
---

Installation media lives in a single library that every host sees **at the same path**. A VM
definition naming an ISO stays valid no matter which host boots it.

That property is the entire point. An ISO reachable on one host and not another produces VMs
that start in some places and not others, for reasons nobody enjoys tracking down.

## How it works

The library is a plain directory, `/var/lib/hyperion/isos`, present on every controller — so it
has as many copies as you have controllers.

The **active controller** exports it over NFS, bound to the **cluster VIP**. Every
non-controller host mounts that export at the same path.

```
/var/lib/hyperion/isos/ubuntu.iso      ← resolves on every host
```

Binding the export to the VIP rather than to a controller's own address is what makes it survive
failover: when the active controller changes, the VIP moves and the mounts keep resolving.
Pointing hosts at a fixed controller would make every controller failover a fleet-wide ISO
outage.

## Replication between controllers

Controllers replicate the library over the authenticated channel they already use to talk to
each other. A peer lists what it has; whatever it is missing, it pulls.

This is deliberately the low-tech option. ISOs are large, immutable and re-downloadable, so
pull-what-you-lack is the right amount of machinery — no extra trust relationship, no additional
daemon.

## Checking it

The library reports its path, the VIP it is exported from, how many ISOs it holds, and a per
controller view:

```
path:          /var/lib/hyperion/isos
vip:           10.2.32.240
exportSource:  10.2.32.240:/var/lib/hyperion/isos
isoCount:      2

REPLICAS
NODE     ROLE     REACHABLE  ISOs  MISSING
vsnode1  ACTIVE   true       2     —
vsnode2  STANDBY  true       2     —
vsnode3  STANDBY  true       2     —
```

**Missing** is the column that matters. A controller missing ISOs is one that has not caught up;
if it became active, those ISOs would stop resolving fleet-wide.

## Adding an ISO

Upload it from the storage screen. It lands on the controller you uploaded to and replicates to
the others.

Large uploads take a while and replication follows, so an ISO is not instantly present
everywhere. Check the replica view before relying on it.

## Using an ISO

- **When creating a VM** — attach it as installation media.
- **On a running VM** — attach or change it from the VM's console screen, without stopping the
  VM.

Guest tools can also be mounted from the console screen. See
[Console and terminal](/vms/console-access/).

## Capacity

The library has its own logical volume, `/var/lib/hyperion/isos`, sized by the installer.

Each controller stores the whole library, so the space is consumed as many times as you have
controllers. Delete media you no longer need — an old point release nobody installs is costing
space on every controller.

## Operating system images

Distinct from ISOs you upload: the deployment can sync operating system images from the software
repository it is pointed at.

An air-gapped site has no such repository and uses uploaded ISOs only. See
[Air-gapped sites](/updating/air-gapped/).
