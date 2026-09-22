---
title: "Metrics"
description: "What the platform measures, how often, and how long it keeps it."
status: published
---

Every host reports metrics to the control plane continuously. They drive the dashboard charts,
the host detail views, and the [alert](/monitoring/alerts/) rules.

## Collection and retention

| | |
|---|---|
| Metric interval | **10 seconds** |
| Host heartbeat | 30 seconds |
| Alert evaluation | Every 60 seconds |
| Retention | **168 hours** (7 days) by default |

Retention is configurable. Ten-second samples for a week is a lot of rows on a large fleet, so
raising it has a real cost in database size — and lowering it below a week means a Monday
investigation can no longer see the previous weekend.

## What is collected

**Per host:** CPU utilization, memory, load, disk usage and I/O, network throughput, and
hardware sensors where the server exposes them — temperatures in particular.

**Per VM:** CPU, memory, disk I/O and network throughput.

**Cluster-wide:** aggregates across hosts, plus vCPU and memory *allocation* against physical
capacity.

## Allocation is not utilization

The dashboard shows both, and the difference matters.

**Allocation** is what has been promised to VMs. **Utilization** is what is being used right
now.

vCPU allocation above 100% is normal and expected — oversubscribing CPU is most of the point of
virtualization. Memory allocation above 100% is a different matter: a host that fails leaves
VMs with nowhere to restart that has the memory they were promised.

Watch allocation for capacity planning and utilization for performance. Neither on its own tells
you much: high allocation with low utilization is healthy consolidation; high allocation with
high utilization means no headroom.

## Where to see them

| View | Shows |
|---|---|
| **Dashboard** | Fleet-wide capacity, live load, per-host bars |
| **Host detail** | CPU/RAM and network I/O over the last six hours, plus sensors |
| **VM detail** | That VM's own usage |

Live values arrive over a WebSocket, so charts update without reloading the page.

## Sensors

Where the hardware reports them, temperatures are shown per host, with thresholds: amber from
**68 °C**, red from **78 °C**.

A host climbing into amber under normal load is usually a cooling problem — a failed fan, a
blocked intake, a rack that is warmer than it should be. It is worth an
[alert](/monitoring/alerts/) rule, because the failure it precedes is abrupt.

## Utilization thresholds

CPU, memory and disk bars change colour at **75%** (amber) and **90%** (red) throughout the
console. These are display thresholds — they do not raise alerts by themselves. Create rules
for that.

## Host health

The heartbeat is what decides whether a host is online. A host that stops heartbeating is
eventually declared `OFFLINE`, which is what starts the
[failover pipeline](/high-availability/failover/).

There is exactly one health check making that decision. A second detector with its own timeout
would eventually disagree with the first, and two mechanisms disagreeing about whether a host is
dead is precisely the condition that corrupts disks.

## External monitoring

The platform exposes metrics in a standard format for scraping, so a deployment can feed an
existing monitoring system rather than being watched only from its own console.

That is worth doing. The console's metrics are stored on the cluster; if the cluster is the
thing having a problem, an external system is what still has the history.

## Reading a problem

| Symptom | Look at |
|---|---|
| A guest is slow | That host's CPU and disk I/O, and its neighbours |
| Everything on one host is slow | Host CPU, memory, and whether storage is degraded |
| Storage is slow fleet-wide | Storage cluster health, and whether a rebuild is running |
| Network throughput capped | Uplink state, bond members, MTU probe result |
| A host climbing in temperature | Sensors, then the hardware |

A rebuild in progress is the most common explanation for storage that is suddenly slower without
anything else changing. See [Replicated storage](/storage/replicated/).
