---
title: "Snapshot policies"
description: "Taking snapshots on a schedule and pruning old ones automatically."
status: published
---

A snapshot policy takes [snapshots](/vms/snapshots/) of a VM on a schedule and deletes the
oldest when there are too many. It exists so that the discipline does not depend on anyone
remembering.

## What a policy holds

| Field | Notes |
|---|---|
| Name | Yours |
| VM | The policy applies to one VM |
| Schedule | When snapshots are taken |
| Retention count | How many to keep. Default **5** |
| Disk only | Default **on** |
| Enabled | Policies can be paused without deleting them |

The policy also records when it last ran and when it next will, so you can see whether it is
actually firing.

## Retention

When a policy takes a snapshot and the count exceeds the retention, the oldest policy-created
snapshot is removed.

Retention is a **count**, not an age. A policy that runs hourly and keeps 5 gives you five
hours of history; the same retention on a daily schedule gives you five days. Pick the pair
together — the question is how far back you need to reach, and the count is just arithmetic
against the interval.

> **Note** — Policies prune **their own** snapshots. A snapshot you took by hand is not
> deleted by a policy, which is what you want: a manual snapshot taken before a risky change
> should not disappear because a schedule fired.

## Disk only

On by default, and usually right. A memory snapshot is larger by the size of the guest's RAM
and takes longer to take — acceptable occasionally, expensive every hour.

Turn it off only where the state you are protecting genuinely lives in memory.

## What this protects against

Snapshot policies protect against **change**: a bad upgrade, a mistaken deletion, a
configuration that broke something.

> **Warning** — They do **not** protect against loss. Snapshots live on the same storage as
> the VM; a failure that takes out the storage takes them with it. For protection against loss
> use [replicated storage](/storage/replicated/) and
> [disaster recovery](/data-protection/disaster-recovery/).

## Cost

Snapshots consume space in proportion to how much the VM changes after they are taken. A busy
database with hourly snapshots and a long retention can consume more than the VM itself.

Watch pool capacity and set an [alert](/monitoring/alerts/) on it. A pool filled by snapshot
retention fails writes for every guest in it, not just the one being snapshotted.

## Consistency

A scheduled snapshot of a running VM is crash-consistent unless guest tools are installed and
able to quiesce the filesystems. With them, the guest flushes and freezes first, producing a
clean image.

For anything transactional, install guest tools. See [Templates and images](/vms/templates/).

## A sensible starting point

| Workload | Schedule | Retention |
|---|---|---|
| Production application | Daily | 7 |
| Database with its own backups | Daily | 3 |
| Development | Weekly | 2 |
| Disposable | None | — |

Snapshots are a fast local undo. They are not the whole of a backup strategy, and a policy on
every VM with a long retention is a good way to run out of storage.
