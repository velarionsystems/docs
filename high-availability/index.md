---
title: "How high availability works"
description: "Controller failover, the virtual IP, database replication and why three controllers."
status: published
---

Two separate things are called high availability, and they protect different failures:

- **Controller HA** keeps the *management plane* available when a controller fails. This page.
- **Workload HA** restarts *virtual machines* when the host running them fails. See
  [Availability zones](/high-availability/availability-zones/).

A cluster can lose a controller without losing its VMs, and lose a host without losing its
console. They are independent.

## Controller HA

One controller is **active**. It serves the console and the API, owns the virtual IP, and makes
every decision. The others are **standbys**: they hold a continuously updated copy of the
database and are ready to take over.

```
Hyperion[vsnode1]> show cluster status
CLUSTER  [OK]  3+ healthy controllers — quorum margin is healthy.

Enabled:             true
Cluster ID:          cluster-5c189bef
VIP:                 10.2.32.240 (mgmt-mgmt/24)
VIP alive:           true
VIP owner:           vsnode1
This node:           10.2.32.101
Term:                1
Isolated from peers: false

Members
NODE     ROLE     ADDRESS      PRIORITY  ALIVE  SYNC  SELF
vsnode1  ACTIVE   10.2.32.101  100       true   OK    true
vsnode2  STANDBY  10.2.32.102  200       true   OK    false
vsnode3  STANDBY  10.2.32.103  200       true   OK    false
```

## The virtual IP

The VIP is the address of the deployment. It follows the active controller: when the active node
changes, the VIP moves and the console comes back at the same address.

Always use the VIP, or a DNS name pointing at it. A controller's own address works only while
that controller happens to be active.

> **Note** — Reaching a standby directly and attempting a write returns **HTTP 409** naming the
> active controller, rather than accepting a change it cannot commit. Reads work anywhere.

## Database replication

Standbys replicate from the active controller by **streaming physical replication**. The standby
database is in recovery and therefore **read-only as a property of the database engine** — not
as a rule the application tries to enforce.

That distinction matters: a guard the application implements can have gaps; a database in
recovery simply cannot accept a write.

Replication is asynchronous, so a standby can be a little behind. The `SYNC` column reports
whether each member is keeping up.

> **Warning** — A standby that is `ALIVE` but not in sync **cannot safely take over**. This is
> the column to look at during an incident, and the one to check after any maintenance.

## Priority, and why lower wins

Each controller has a priority, and **lower is more preferred**. A cluster with a master at 100
and joiners at 150 or 200 will elect the master whenever it is healthy.

The inversion trips people up. Think of it as a ranking — node 1 is first choice — rather than
as a weight.

> **Warning** — A newly joined controller must have a **higher** number than the running
> master. A joiner that outranks the master can win an election and take the virtual IP before
> its database has finished syncing.

## Why three controllers

Electing an active node requires a **majority**, with no special case for two.

| Controllers | Tolerates | Automatic failover |
|---|---|---|
| 1 | Nothing | No |
| 2 | Nothing — two cannot form a majority when they disagree | **No** |
| 3 | One failure | Yes |

Two controllers give you a warm spare with a synchronised database, which has value — but it
does **not** give you automatic failover. If the two cannot see each other, neither can know
whether the other is dead or merely unreachable, and a system that guesses in that situation
produces two active controllers and a split database.

Three is the first count at which failover is automatic.

## What happens on failover

1. The active controller stops responding to its peers.
2. The remaining members, if they form a majority, elect a new active — preferring the lowest
   priority among those whose databases are fresh.
3. The new active takes the virtual IP.
4. The console returns at the same address.

Running VMs are **not affected**. They are running on hosts; the control plane is not in their
data path. You lose the ability to *manage* the deployment for the duration, not the workloads.

## Term and isolation

**Term** increments each time a new active is elected. A term climbing on its own indicates
repeated elections — usually a network problem between controllers rather than controllers
failing.

**Isolated from peers** on a node means it cannot see the others. On the active node that is a
warning; on a standby it means that member is not a candidate.

## Maintenance

Suspend a controller before working on it, so the cluster stops considering it a candidate, and
resume it afterwards.

```
Hyperion[vsnode1]> cluster node suspend <id|ip>
Hyperion[vsnode1]> cluster node resume <id|ip>
```

To work on the **active** controller, let the cluster elect another one first — suspending the
active node triggers an election.

See [Upgrading the fleet](/updating/fleet-upgrade/).

## On a single node

A standalone deployment has no controller HA — there is nothing to fail over to. The console is
available exactly while the node is. Everything else in the product works.

## Health checklist

| Check | Healthy |
|---|---|
| One member `ACTIVE` | Exactly one |
| All members `ALIVE` | `true` |
| All members `SYNC` | `OK` |
| `VIP alive` | `true` |
| `Isolated from peers` | `false` |
| `Term` | Stable |
