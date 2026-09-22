---
title: "Virtual USB"
description: "Presenting files to a running guest as a USB mass-storage device."
status: published
---

Virtual USB presents a file to a running VM as if someone had plugged a USB stick into it. The
guest sees ordinary removable storage — no network, no shared folder, no drivers beyond what it
already has for USB.

It solves a specific problem: getting something *into* a guest that has no working network yet.

## When to use it

- A driver a freshly installed guest needs before its NIC works.
- A licence file or activation key.
- A configuration bundle for a first boot.
- Getting a small file out of an isolated guest.

For anything routine, use the network. Virtual USB is for the cases where the network is the
thing that does not work.

## Using it

Upload the file to the deployment's virtual USB store, then attach it to a running VM from the
VM's console screen. The guest sees a new removable device.

Detach it when you are done. A guest left with a virtual USB device attached keeps it across
reboots, which is rarely what you want and occasionally confusing.

The store is managed centrally: you can list what has been uploaded, fetch a file back, and
delete files you no longer need.

## Constraints

| | |
|---|---|
| VM state | The VM must be **running** to attach or detach |
| Guest support | Any guest with USB mass-storage support, which is all of them |
| Size | Suited to files, not bulk data — use a disk or the network for that |

> **Note** — Attaching a device to a running guest does not force the guest to mount it. A
> Linux guest may need the filesystem mounting; Windows normally assigns a drive letter by
> itself.

## Alternatives

| Need | Better tool |
|---|---|
| Install an operating system | Attach an ISO — see [The ISO library](/storage/iso-library/) |
| Add persistent capacity | Add a disk — see [Volumes](/storage/volumes/) |
| Move bulk data | The guest's network |
| Run a command in the guest | The guest's own remote access |

## Audit

Attaching and detaching a device are recorded in the
[audit log](/administration/audit-log/), with the user who did it. Moving files into and out of
guests is exactly the kind of thing that should leave a trail.
