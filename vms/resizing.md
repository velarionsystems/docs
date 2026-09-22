---
title: "Resizing CPU and memory"
description: "Changing a running VM's vCPUs and RAM, and the ceiling that limits it."
status: published
---

CPU and memory can be changed on a **running** VM. No reboot, no downtime — within a limit that
is worth understanding before you need it.

## The maximum matters

A VM is defined with a **maximum vCPU count**. Live resize can move the current count anywhere
up to that maximum, and no further.

```
vCPUs (12) exceeds this VM's maximum of 8 — raise Maximum vCPUs in Edit VM
(applies on the VM's next start) first
```

Raising the ceiling is a **definition change**, and definition changes apply on the VM's next
start. So:

- **Within the maximum** — takes effect immediately, guest running.
- **Above the maximum** — raise the maximum in **Edit VM → CPU & Memory**, then restart the VM,
  then resize.

> **Note** — Set a realistic maximum when you create a VM, above what you expect to need. A
> maximum that is too low is the difference between a live resize and a restart at the moment
> you are trying to get out of trouble.

## Resizing

Open the VM and choose **Resize**. Set vCPUs, memory, or both.

Memory is adjusted live through the guest's balloon driver. That requires the guest to have the
driver — it is part of guest tools, and present by default in most modern Linux distributions.

## What the guest does with it

Adding resources to the hypervisor's view of a VM is not the same as the guest using them.

**CPU.** Most Linux guests bring new vCPUs online automatically. Some require the new CPU to be
onlined inside the guest. Windows Server editions support hot-add; client editions generally do
not.

**Memory.** Ballooning returns memory to the host or hands it over without the guest rebooting,
but a guest that has already allocated what it had will not shrink gracefully. Growing works
far more reliably than shrinking.

> **Warning** — Reducing a running VM's memory below what the guest is using invites the
> out-of-memory killer. Shrink with the guest stopped unless you know its working set.

## Disks

Disk capacity is separate from CPU and memory. Adding a disk, or growing an existing one, is
done from the VM's storage section. Growing the virtual disk does not grow the filesystem
inside it — the guest still has to extend its partition and filesystem.

## Capacity and zones

Resizing a VM in an availability zone consumes capacity the zone was accounting for. A zone
holds back headroom so that it can restart workloads when a host fails; growing VMs into that
headroom quietly reduces what the zone can absorb.

Check the zone's capacity after a significant resize. See
[Availability zones](/high-availability/availability-zones/).

## From the CLI and API

Resize is available through the API as a single call taking vCPUs and memory, and is recorded
as a [task](/console/tasks/) and in the [audit log](/administration/audit-log/) like any other
change.
