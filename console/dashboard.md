---
title: "Dashboard"
description: "Fleet health at a glance: capacity, hosts, storage, networking, alerts and controller state."
status: published
---

The Dashboard is the landing screen. It answers one question — *is anything wrong* — and gives
you the path to whatever is.

## The top row

Six counters across the top:

| Counter | Meaning |
|---|---|
| **Online Hosts** | Servers online, out of the total registered |
| **Running VMs** | Virtual machines currently running |
| **Containers** | Container workloads running |
| **Storage** | Capacity across the fleet |
| **Networks** | Virtual networks defined |
| **Active Alerts** | Alert events not yet acknowledged |

A non-zero **Active Alerts** is the one to look at first.

## Capacity

Three meters showing what is committed against what exists:

- **vCPU allocation** — virtual CPUs assigned to VMs against physical cores.
- **Memory allocation** — memory assigned against installed memory.
- **Live CPU load** — what the fleet is actually using right now.

Allocation above 100% is normal for vCPU — virtual machines rarely all run flat out, and
oversubscribing CPU is the point of virtualization. Memory is different: allocating more memory
than the fleet has means a host that loses a peer may not have anywhere to restart its VMs.

> **Note** — Read allocation alongside live load. High allocation with low load is healthy
> consolidation. High allocation with high load means you are out of headroom.

## Host health

One row per server, each with its status and three utilization bars — CPU, RAM and DISK —
plus a temperature reading where the hardware reports one.

The bars change colour as they climb: amber from 75%, red from 90%. Temperature follows the
same idea, turning amber at 68 °C and red at 78 °C.

Click a row to open that server — see [Infrastructure](/console/infrastructure/).

## Block storage

A summary of iSCSI exports: targets, storage nodes, LUNs, how much is provisioned, how much is
attached, and whether multipath is in play. See [Exports](/storage/exports/).

## Networking

Counts of fabrics, switches and routers in the virtual network layer. See
[Networking](/networking/).

## Running VMs

The virtual machines currently running, with the host each is on. Empty on a new deployment.

## Alerts

The most recent alert events, coloured by severity — emergency, critical and error in red,
warning in amber, informational in blue. See [Alerts](/monitoring/alerts/).

## Recent tasks

The last operations the platform ran, with status and progress. A failed task here is usually
the explanation for whatever else looks wrong. See [Tasks](/console/tasks/).

## Controller HA

Which controller is **ACTIVE**, which are **STANDBY**, and whether each one's database is in
sync. On a single-node deployment this panel is not shown.

This is the panel that matters during an incident: a standby that is alive but not in sync
cannot take over. See [High availability](/high-availability/).

## Licensing

The current licence state and what it entitles. See [Licensing](/administration/licensing/).

## Refresh

The Dashboard polls on its own; you do not need to reload it. Live metrics arrive over a
WebSocket, so the numbers move without the page flickering.
