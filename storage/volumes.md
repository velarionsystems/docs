---
title: "Volumes"
description: "VM disks — adding, growing, moving and detaching them."
status: published
---

A volume is a disk attached to a VM. Where it lives decides what the VM can do; how it is
attached decides how well the guest performs.

## Adding a disk

From the VM's storage section, or when creating it.

| Field | Notes |
|---|---|
| Size | In GB |
| Backing | Pool, iSCSI LUN, or replicated storage image |
| Format | The disk image format |
| Bus | `virtio` unless the guest cannot |
| Cache mode | Leave at the default unless you know why |
| Bootable | Which disk the firmware boots from |

### Backing

| Backing | Shared | Notes |
|---|---|---|
| Storage pool | No | Fast, local, pins the VM to its host |
| iSCSI LUN | Yes | New LUN, or attach an existing one |
| Replicated storage image | Yes | New image, or attach an existing one |

One local disk is enough to make the whole VM unable to migrate. If mobility matters, all of a
VM's disks need shared backing.

### Bus

Use `virtio`. It is a paravirtual interface, so the guest talks to the hypervisor directly
instead of through an emulated controller.

Other buses exist for guests whose installer has no virtio driver — older Windows, chiefly. The
usual approach is to install with a driver disk attached rather than to accept emulated hardware
permanently.

## Growing a disk

Growing a volume is two steps, and the second happens **inside the guest**:

1. Grow the volume in the console.
2. Extend the partition and filesystem inside the guest.

Step 1 alone gives the guest a bigger disk with the same partition table. Nothing appears to
change until the guest is told.

> **Warning** — Shrinking is not offered. Reducing a disk below what its filesystem occupies
> destroys data, and nothing outside the guest can know what is safe. Create a smaller disk and
> copy.

## Moving a disk

Disks can be moved between backings while the VM stays defined. Use it to:

- get a VM off local storage so it can migrate,
- move a VM onto replicated storage,
- empty a pool you want to remove.

The move copies data. A running VM's disk can be moved, but the copy competes with the guest's
own I/O.

## Detaching and deleting

Detaching removes a disk from the VM without destroying it. Deleting destroys the data.

The platform requires an explicit force flag for a deletion that destroys data, so that removing
a LUN cannot happen as a side effect of tidying up.

> **Warning** — A detached disk still consumes space. Detaching is not cleanup; it is
> unplugging.

## Thin provisioning

Disks can be thin-provisioned, consuming space as the guest writes rather than up front. It lets
you allocate more than you have.

That is useful and it is a liability. Monitor the **pool's** free space, not the guests' — a
guest believes it has the space it was given, and a thin pool that fills makes writes fail
underneath a filesystem that has no idea. Set an alert on pool capacity. See
[Alerts](/monitoring/alerts/).

## Snapshots

A VM snapshot captures its disks together, which is what you want — a snapshot of one disk of a
multi-disk VM is a torn image. See [Snapshots](/vms/snapshots/).

## Passthrough

A physical disk can be given directly to a guest. It performs like the hardware, because it is
the hardware — and it pins the VM to that host permanently. See
[Creating a VM](/vms/creating/).
