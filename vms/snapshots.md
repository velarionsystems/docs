---
title: "Snapshots"
description: "Point-in-time captures of a VM, what they include, and what they are not."
status: published
---

A snapshot captures a VM at a moment so you can return to it. Take one before an upgrade,
a configuration change, or anything you might want to undo.

## Disk-only or with memory

| Kind | Captures | Reverting gives you |
|---|---|---|
| **Disk only** | The disks | The VM as if it had been powered off at that moment |
| **Disk and memory** | Disks plus RAM | The VM running, exactly as it was |

Disk-only is the default and the right choice most of the time. A memory snapshot is larger by
the size of the guest's RAM and takes longer, but it restores a *running* machine — no boot, no
service restart, no lost in-memory state.

Take a memory snapshot when the thing you are protecting is in RAM. Take a disk snapshot when
the thing you are protecting is on disk, which is nearly always.

## Snapshots form a tree

Each snapshot records its parent. Taking a snapshot, reverting, then taking another gives you a
branch rather than a line. The VM's snapshot list shows the relationships.

This matters when deleting: removing a snapshot with children is not the same as removing a
leaf.

## Creating one

Open the VM, choose **Snapshots → Create**, give it a name and a description, and decide
whether to include memory.

Name them for *why*, not when — the timestamp is already recorded. "before 14.2 upgrade" is
useful in three weeks; "snapshot3" is not.

## Reverting

Reverting discards everything written since the snapshot was taken.

> **Warning** — There is no undo. A revert throws away all changes after that point, including
> data written by applications that believe it is safely committed. Shut the guest down cleanly
> first if you can, and be certain you have the right snapshot.

## Deleting

Snapshots consume space that grows as the VM diverges from them. Delete them when the change
they were protecting has proved itself.

## Quiescing

A snapshot of a running VM captures the disk as it is at that instant — which may include a
database mid-write. With guest tools installed, the platform can ask the guest to flush and
freeze its filesystems first, producing a consistent image.

Without guest tools, a disk snapshot of a running VM is *crash-consistent*: equivalent to
pulling the power. Most filesystems and databases recover from that, but recovery is not the
same as consistency. See [Templates and images](/vms/templates/) for installing guest tools.

## What snapshots are not

> **Warning** — **Snapshots are not backups.** They live on the same storage as the VM. A
> failure that takes out the storage takes the snapshots with it. They protect against *change*,
> not against *loss*.

For protection against loss, use replicated storage plus one of:

- [Disaster recovery](/data-protection/disaster-recovery/) — replication to a second site.
- [Backup and restore](/data-protection/backups/) — for the control plane's own database.

Nor are snapshots a substitute for capacity planning: a VM left with months of snapshots
consumes storage proportional to how much it has changed.

## Automatic snapshots

A snapshot policy takes them on a schedule and prunes old ones, so the discipline does not
depend on anyone remembering. A policy has a schedule, a retention count — how many to keep,
default 5 — and whether snapshots are disk-only, which they are by default.

See [Snapshot policies](/data-protection/snapshot-policies/).
