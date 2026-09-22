---
title: "Exports"
description: "Serving storage out over iSCSI — targets, LUNs, CHAP and multipath."
status: published
---

As well as consuming storage, VS-HCI can serve it. A storage node exports iSCSI targets that
hosts in the deployment — or machines outside it — attach to.

**One VM disk is one LUN.**

## When to use an export

- Shared block storage without a full replicated storage cluster.
- Giving a machine outside the deployment access to storage on it.
- A VM that needs raw block storage rather than an image file.

If you are choosing storage for VM disks generally, [replicated
storage](/storage/replicated/) is the better answer in a three-node deployment.

## Targets

A target is the thing initiators connect to. It has an IQN, one or more portals, and the LUNs
beneath it.

Creating a target from the storage screen sets up the target side on the storage node. Its IQN
is derived deterministically, so a retry after a failure re-uses the same target rather than
leaving an orphan behind.

## LUNs

A LUN is a block device under a target. Create one per VM disk.

LUN identifiers are allocated under a uniqueness constraint, so two simultaneous requests cannot
be handed the same one.

> **Warning** — Deleting a LUN destroys its data, and the platform requires an explicit force
> flag for that reason. Detach it from whatever is using it first.

## CHAP

Authentication is per target.

| | |
|---|---|
| **Incoming** | The initiator authenticates to the target |
| **Mutual** | The target also authenticates to the initiator |

Secrets are encrypted at rest and never returned by the API — you can set them, you cannot read
them back.

Mutual CHAP is worth the extra configuration where the storage network is not fully trusted: it
stops an initiator being pointed at a target that is not the one it thinks.

## Multipath

A target with several portals gives an initiator more than one path to the same LUN. Multipath
survives losing a NIC, a switch or a path without the LUN disappearing.

For this to be redundancy rather than decoration, the portals must be on genuinely different
paths — different NICs into different switches. Two portals on the same link is one path
advertised twice.

## Sessions

Hosts log themselves in to the targets the platform creates; you do not attach each host by
hand. Sessions are persisted so that they return automatically after a reboot — a host that
came back without its storage would be a host with VMs that cannot start.

The fleet view shows which hosts have sessions to which targets. The useful question is not
"who is logged in" but "is anyone missing" — a host without a session cannot run VMs whose disks
are on that target.

### External arrays

An array the platform did not create can be attached. Discovery runs from one host as a
read-only probe, and the array is then logged in per host.

These are managed differently from targets the platform owns, and the fleet view distinguishes
them.

## Attaching a LUN to a VM

Either provision a new LUN under a target as part of adding a disk, or attach an existing one.

The disk appears to the guest as raw block storage.

## Failure behaviour

| Symptom | Cause |
|---|---|
| VM will not start, disk missing | The host has no session to the target |
| I/O errors under load | A path failed and multipath is not configured |
| LUN visible but not writable | Attached elsewhere, or a CHAP mismatch after a change |
| Sessions gone after reboot | Persistence not applied — check the host's sessions |

A LUN attached to two hosts at once is not shared storage. Unless a cluster filesystem is in
play, two writers will corrupt it. The platform tracks LUN-to-host mapping so that this does not
happen by accident.
