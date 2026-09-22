---
title: "Infrastructure"
description: "Servers, availability zones and datacenters — and everything a single host exposes."
status: published
---

**Infrastructure** is the physical side of the deployment: the servers themselves, grouped into
availability zones and datacenters.

## Hosts

Every registered server, with its status, role and utilization.

### Status

| Status | Meaning |
|---|---|
| `PENDING_APPROVAL` | Enrolled with a valid join token, not yet admitted. It may report in; the control plane will not send it commands |
| `PENDING` | Registered, waiting to start bootstrapping |
| `BOOTSTRAPPING` | Being prepared — packages, agent, keys |
| `ONLINE` | Healthy and accepting work |
| `OFFLINE` | Not reachable |
| `MAINTENANCE` | Deliberately excluded from scheduling |
| `DEGRADED` | Reachable but not fully healthy |
| `ERROR` | Failed in a way that needs attention |

### Role

| Role | Runs |
|---|---|
| `HYPERCONVERGED` | Workloads and storage — the normal case |
| `COMPUTE` | Workloads only |
| `STORAGE` | Storage only |
| `ORCHESTRATOR` | Control plane only |

A host's role can be changed from its detail screen. Moving a host out of a storage role has
consequences for replicated storage — read [Replicated storage](/storage/replicated/) first.

### Maintenance mode

Toggling maintenance takes a host out of scheduling. Nothing new is placed on it, and depending
on how you move workloads, running VMs are migrated off.

Use it before any planned disruption — a reboot, a firmware update, physical work. See
[Upgrading the fleet](/updating/fleet-upgrade/).

```
Hyperion[vsnode1]> host maintenance vsnode2 on
```

## A single host

Clicking a host opens its detail screen, with tabs:

### Overview

Hardware and live state — CPU model, cores and threads, memory, disks, kernel and OS version,
whether KVM is enabled, agent version, and how long since the host was last seen. Below that,
CPU/RAM and network I/O charts over the last six hours, a sensor panel, and the VMs running here.

`kvmEnabled: false` on a host that should run VMs means hardware virtualization is off in
firmware. Nothing will start until that is fixed.

### Storage

The storage this host contributes and the pools defined on it. See [Storage](/storage/).

### Network

The host's physical interfaces, bonds, bridges and the uplinks it is a member of. This is where
you confirm a host is actually attached to the uplink you think it is. See
[Uplinks and bonds](/networking/uplinks-and-bonds/).

### Users

Accounts that can sign in to **this host's** CLI. These are managed centrally and pushed to the
host, so they keep working when the control plane does not — which is the point. See
[The VS-HCI shell](/cli/).

### Diagnostics

A curated set of read-only checks that run on the host and return their output. Safe to run at
any time. See [Diagnostics](/monitoring/diagnostics/).

### Packet capture

Capture traffic on a host interface and download it. For chasing a networking problem to
ground.

### Terminal

A shell on the host, in the browser. It opens the VS-HCI CLI, not a Linux prompt — the same
restriction as SSH — and the session is audited when it opens and closes.

## Availability zones

A zone is a group of hosts that share a failure domain, and the unit the platform places and
restarts workloads within.

The Infrastructure screen shows each zone with the hosts in it and how much capacity is
committed. A VM belongs to a zone; when its host fails, the platform restarts it on another
host **in the same zone**.

That is the whole point: put hosts on separate power feeds, racks or rooms into separate zones,
and a VM will not be restarted onto the failure it was just hit by.

See [Availability zones](/high-availability/availability-zones/).

## Datacenters

The outermost grouping — a site. Most deployments have exactly one. Datacenters exist so that a
larger estate can describe where things physically are.

## Removing a host

Put it into maintenance so its workloads move off, then remove it. A host that also runs the
control plane must additionally be removed from the HA cluster.

> **Warning** — Do not simply power off a node and leave it registered. The cluster continues
> to expect it, and an absent controller still counts against quorum. Remove it properly.
