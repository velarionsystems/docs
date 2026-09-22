---
title: "Creating a VM"
description: "Every choice the create form asks for, and which ones you cannot change later."
status: published
---

**VMs → Create**. The form is long because a virtual machine has a lot of properties; most have
sensible defaults and only a handful genuinely need a decision.

## Identity and placement

| Field | Notes |
|---|---|
| Name | Must be unique |
| Availability zone **or** host | One or the other, never both |
| Tags, notes | Free-form, for your own organisation |

In a deployment with zones, pick the **zone** and let the scheduler place the guest. Pick a
**host** only where no zones are defined. See
[Availability zones](/high-availability/availability-zones/).

> **Note** — If the zone has to move or stop other guests to fit this one in, the form tells
> you which ones **before** you confirm. That consent is part of the request; the platform will
> not reshuffle other people's workloads behind your back.

## CPU and memory

| Field | Notes |
|---|---|
| vCPUs | Virtual CPUs |
| Memory | In MB |
| Sockets / cores / threads | Optional explicit topology |

Leave the topology alone unless the guest's licensing or NUMA behaviour depends on it. Some
operating systems and databases are licensed per socket, and presenting 8 sockets rather than
1 socket × 8 cores can be expensive.

**Hugepages** reduces memory-management overhead for large guests. It requires hugepages to be
configured on the host and reserves memory up front.

## Firmware

| Field | Notes |
|---|---|
| Firmware | BIOS or UEFI |
| Machine type | The emulated chipset |
| Secure Boot | UEFI only |

> **Warning** — Firmware is effectively permanent. An operating system installed under BIOS
> will not boot when switched to UEFI. Decide before you install the guest, not after. Modern
> guests want UEFI; Secure Boot on top of it if the guest supports it.

## Disks

Add one or more. Per disk:

| Field | Notes |
|---|---|
| Size | In GB |
| Backing | Storage pool, iSCSI LUN, or replicated storage image |
| Format | Disk image format |
| Bus | `virtio` unless the guest cannot |
| Cache mode | Leave at the default unless you know why |
| Bootable | Which disk the firmware boots |

**Backing decides migration.** A disk on a host's storage pool is local: the VM cannot move to
another host until the disk moves. A disk on an iSCSI LUN or a replicated storage image is
shared, and the VM can migrate freely.

Choose shared backing for anything you expect to keep running through a host failure.

Use `virtio` for the bus. Other buses exist for guests whose installer has no virtio driver —
older Windows, chiefly — and cost performance.

## Network

Add one or more NICs. Per NIC:

| Field | Notes |
|---|---|
| Attachment | A distributed switch, or a host bridge and port group |
| Model | `virtio` unless the guest cannot |
| MAC address | Generated; override only if something depends on it |
| Queues | Multi-queue for high-throughput guests |
| Rate limit | Cap in Mbps |

Attach to a **distributed switch** to get a network that follows the VM across hosts. A host
bridge ties the VM to that host's physical layout. See
[Distributed switches](/networking/distributed-switches/).

## Installation media

Attach an ISO from the [ISO library](/storage/iso-library/) to install from. The library is
shared across the cluster, so the same ISO is available wherever the VM is placed.

You can also start from a [template](/vms/templates/), which skips installation entirely.

## PCI passthrough

Assign a physical device — a GPU, an HBA, a NIC — directly to the guest. The form lists devices
the host has available.

> **Warning** — A VM with a passthrough device is bound to the host holding that device. It
> cannot live-migrate, and it cannot be restarted elsewhere by HA. Passthrough buys performance
> at the cost of mobility.

## Availability options

| Field | Notes |
|---|---|
| HA enabled | Restart elsewhere if the host fails |
| HA priority | Restart order among HA VMs |
| Anti-affinity group | Keep apart from others in the same group |
| Evictable | May be powered off to make room for something more important |
| Auto-start | Start when the host boots |
| DR capable | Replicate to the paired site |

**DR capable** is applied at the end of the build, once the disks exist. See
[Disaster recovery](/data-protection/disaster-recovery/).

## What happens next

Creating a VM returns immediately and does the work as a [task](/console/tasks/): disks are
provisioned, the network is attached, the domain is defined. Watch the task for progress; a
failure there carries the reason.

If **auto-start** is off — the default — the VM is created stopped. Start it and open its
[console](/vms/console-access/) to install the guest operating system.

## What you cannot change later

Most things are editable. These are painful or impossible:

- **Firmware type** — reinstall the guest.
- **Machine type** — changing it can make an installed guest unbootable.
- **Boot disk bus** — the guest needs a driver for whatever it boots from.

Everything else — CPU, memory, extra disks, NICs, tags, HA settings — can be changed afterwards.
