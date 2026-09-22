---
title: "Failover and recovery"
description: "What happens when a host dies, the fence that must succeed first, and recovering afterwards."
status: published
---

This page is about **workload** failover: what happens to virtual machines when the host
running them is lost. For the management plane, see
[How high availability works](/high-availability/).

## The pipeline

```
1  a host is declared OFFLINE
2  fence it off the zone's storage
3  list the VMs it was running
4  pick a host for each
5  start it there, or apply the zone's fragmentation policy
```

### 1 — Declaring a host offline

A single health check decides when a host is gone, and the failover pipeline trusts it.

There is deliberately **no second detector**. Two mechanisms with their own timeouts would
eventually disagree — one declaring a host dead while the other still believed it alive — and
that disagreement is the exact condition the whole pipeline exists to avoid.

### 2 — Fencing, and nothing starts before it

The lost host is fenced off the zone's storage before anything is restarted.

> **Warning** — **If the fence fails, the pipeline stops.** The VMs stay down and an operator
> sees an error.
>
> That is the correct outcome. A host that is unreachable but not *provably stopped* may still
> be writing to its disks. The cost of being wrong is not a failed restart — it is a corrupted
> disk with two writers on it.

A zone whose VMs did not come back after a host failure has usually failed at this step. The fix
is to make fencing work, not to bypass it.

### 3–5 — Placing and starting

The VMs that were running on the lost host are placed using the **same routine used when a VM is
created**, so failover placement obeys the same rules and the same capacity arithmetic as
everything else.

If a VM will not fit anywhere, the zone's fragmentation policy decides what happens: make room
by moving or stopping other workloads, or leave it down and raise an alarm.

A VM the platform has decided not to start ends up `PENDING`, with the reason attached. Nothing
restarts a `PENDING` VM automatically — see [Virtual machines](/vms/).

## What gets restarted

Only VMs with **HA enabled**. A VM without it stays down until someone starts it.

**HA priority** sets the order, which matters when capacity is tight — the things that come back
first should be the things you need first.

**Evictable** VMs may be powered off to make room for more important ones.

## Capacity is reserved, not hoped for

A host joins a zone by **locking** memory and vCPUs into it, and the sum of a host's locks across
every zone may never exceed what the host physically has.

Nothing is allowed to oversubscribe that — not the reservation, not VM creation. A zone that
could be told "you have 400 GB" by two zones sharing the same hosts would report failover
headroom that does not exist, and would keep reporting it right up until the failure it was
supposed to survive.

This is why a zone's capacity view is worth trusting, and why creating a VM can be refused even
though a host appears to have free memory.

## Recovering a failed host

1. **Fix the cause.** Power, hardware, network, or whatever the diagnostics say.
2. **Bring it back.** It reconnects and returns to `ONLINE`.
3. **Check for duplicates.** The platform stops a domain that came back on the recovered host
   when it is already running elsewhere — but confirm.
4. **Rebalance if you want to.** VMs do not move back on their own. The zone can propose a
   rebalance plan.

> **Warning** — Do not bring a fenced host back onto shared storage until you know why it was
> lost. A host that was fenced because it was unreachable but alive may still have stale state.

## Planned maintenance is not failover

For anything planned, use maintenance mode:

```
Hyperion[vsnode1]> host maintenance vsnode2 on
```

Nothing new is scheduled onto it, and workloads move off in a controlled way — live, with no
restart. Failover is what happens when you did not get the choice.

See [Upgrading the fleet](/updating/fleet-upgrade/).

## Testing it

Failover you have not tested is a belief. Test it on a zone with workloads you can afford to
restart:

1. Note what is running where.
2. Power off a host at the hardware level — not a clean shutdown, which is the case that
   already works.
3. Watch the zone: the host goes `OFFLINE`, gets fenced, and its HA-enabled VMs restart
   elsewhere.
4. Confirm what did **not** come back, and why.

Step 4 is the valuable one. VMs without HA enabled, VMs with passthrough devices, and VMs on
local disk will not return — and it is much better to discover that in a test.

## Site failure

Host failover restarts VMs on other hosts in the same zone. If the whole site is gone, there is
no other host. That is what [disaster recovery](/data-protection/disaster-recovery/) is for.
