---
title: "Availability zones"
description: "Grouping hosts into failure domains, and the capacity lock that makes failover real."
status: published
---

An availability zone is a group of hosts that share a failure domain, and the unit within which
the platform places and restarts workloads.

A VM belongs to a zone. When its host fails, it is restarted on another host **in the same
zone** — so if your zones follow your real failure boundaries, a VM is never restarted onto the
failure that just hit it.

## Designing zones

Put hosts into separate zones when they share something that can fail:

| Boundary | Zone per |
|---|---|
| Power feed | Feed |
| Rack | Rack, when racks have independent power and switching |
| Room or floor | Room |
| Nothing | One zone is fine |

A single zone across all hosts is a legitimate design for a deployment in one rack on one feed —
there is no boundary to model, and pretending otherwise adds constraint without adding safety.

> **Note** — Zones only help if they match reality. Three zones whose hosts share one power
> distribution unit give you three names for one failure domain.

## The reservation is a lock

A host joins a zone by **locking** memory and vCPUs into it. The sum of a host's locks across
every zone may never exceed what the host actually has.

This is enforced everywhere — the reservation write and VM creation both refuse to
oversubscribe. A zone that could be told "you have 400 GB" by two zones sharing the same hosts
would report failover headroom that does not exist, and would report it right up until the
failure it was supposed to survive.

Two consequences worth expecting:

- A zone's reported capacity is **real**, and worth trusting.
- Creating a VM can be refused even when a host shows free memory, because that memory is
  locked into a different zone.

## Placement

Inside a zone, **the zone picks the host** — you do not.

The scheduler is the only thing that knows how much capacity is being held back for a failover.
Letting an operator pin a guest to a host would quietly spend that headroom, and the zone would
go on claiming it could absorb a host failure when it no longer could.

Outside a zone, you pick the host. Deployments with no zones defined work this way throughout.

You give a VM *either* a zone *or* a host — never both.

## Admission

Before a VM is created in a zone, the zone answers whether it can take it. This is what makes
the answer to "can I run this here?" available while everything is healthy, rather than during
an outage.

You can ask the question explicitly — an admission check — before committing.

## What a move disturbs

Fitting a VM onto a host may require moving others out of the way, or powering off an
[evictable](/vms/) one.

That plan is computed **before** anything happens and shown to you. The same routine computes
it for the migration dialog and for the failover pipeline, deliberately: if the two disagreed,
the preview would describe a different migration from the one that ran, and the point of the
preview is that a person is consenting to the side effects.

## Rebalancing

A zone can propose a rebalance plan — how it would redistribute workloads across its hosts.
VMs do not move back on their own after a host recovers, so this is how you restore an even
spread after a failure.

Review the plan before applying it. It is a list of live migrations, and each one has the same
side effects any migration does.

## Pending VMs

A VM the zone could not place is `PENDING`, with the reason recorded. The zone view lists them.

`PENDING` means the platform has decided not to start it — most often because no host in the
zone could take it, but also where it cannot act safely, such as a host that could not be
fenced. Nothing restarts a `PENDING` VM automatically.

## Migration networks

A zone can be given a specific network for migration traffic. Moving a running VM copies its
memory between hosts, which is a lot of traffic; putting it on a network of its own keeps it off
the management link.

## Zones and disaster recovery

Zone links are the unit of DR: a main-site zone is linked to a DR-site zone, and protected VMs
in the first are replicated into the second.

The DR zone holds a placeholder for each protected VM, with a host assigned from that zone's own
placement — which is what makes its memory genuinely reserved rather than merely promised. See
[Disaster recovery](/data-protection/disaster-recovery/).

## Adding and removing hosts

Adding a host to a zone locks its capacity into that zone. Removing one takes that capacity out.

> **Warning** — Removing a host from a zone reduces the zone's failover headroom. Check what
> the zone can still absorb afterwards — the moment to find out that it can no longer tolerate a
> host failure is not during one.
