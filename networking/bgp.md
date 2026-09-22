---
title: "BGP"
description: "Peering the fabric edge with your routers, and how learned routes reach the fabric."
status: published
---

BGP lets an overlay fabric exchange routes with your routers dynamically, instead of you
maintaining static routes on both sides.

## Where BGP runs

Only on the **two L3-out chassis**, and only peering with **your** routers.

There is no BGP between VS-HCI hosts. The platform's own control plane distributes internal
topology; using a routing protocol for that as well would be two mechanisms answering the same
question. BGP exists at the edge, where you need to talk to equipment the platform does not
control.

## Prerequisites

An L3-out must exist on the fabric, with its transit network working. See
[Routing and NAT](/networking/routing-and-nat/).

Each chassis peers from its own transit address, which is also its default router ID.

## Configuring a session

**Fabric → L3-Out → BGP**. Set the local AS and add neighbours.

| Setting | Notes |
|---|---|
| Local AS | Your fabric's AS number |
| Neighbour address | Your router, inside the transit CIDR |
| Remote AS | Your router's AS |
| MD5 password | Optional; stored encrypted and never returned by the API |
| BFD | Optional; sub-second failure detection |
| Maximum prefix | A ceiling on what a neighbour may send |
| Advertise fabric networks | **Off by default** |

Sessions are configured on both chassis. A standby that is down does not block the active one
from being configured.

> **Note** — Configuration changes are applied by **reloading** the routing daemon, never
> restarting it. Established sessions survive a configuration change.

### Advertising your networks

`Advertise fabric networks` is off by default, so by default the fabric **learns** routes and
advertises nothing. Turning it on advertises the fabric's networks to your routers.

Leaving it off is the right starting point: learning routes is safe, and advertising is a
decision your network team should make knowingly.

## How learned routes reach the fabric

The platform polls both chassis every 30 seconds for session state and learned routes.

1. Session state is polled from **all** chassis, because a standby has to be verifiably up
   before it can be failed over to.
2. The **active** chassis is identified from the edge's current binding.
3. Learned routes are read from the active chassis only.
4. Those are reconciled into the fabric router's route table as **learned** entries.

Static routes you entered are never touched by this process.

### Redistribution modes

| Mode | Installs |
|---|---|
| **Default only** | Exactly one default route via the first established peer, whatever prefixes arrive |
| **Specific** | One route per learned prefix |

*Default only* is the right choice when your routers are simply the way out — it keeps the
fabric's route table at one entry regardless of how large your routing table is. *Specific* is
for when the fabric genuinely needs to know which next hop serves which prefix.

### Flap protection

A route is withdrawn only after **two consecutive** polls showing the session down. A single
missed poll does not tear routes out of a working fabric.

Fast failure detection is BFD's job, not the poller's. BFD reacts in sub-second time; the poller
is there to keep the route table correct, not to be quick.

## Checking a session

The BGP view shows per-neighbour state and the routes learned from each. A session that never
leaves its initial state usually means one of:

| Cause | Check |
|---|---|
| Transit VLAN not trunked to the chassis | Your switch configuration |
| Address mismatch | Neighbour address inside the transit CIDR |
| AS mismatch | Remote AS matches what your router is configured with |
| Password mismatch | MD5 set on one side only |
| Prefix limit hit | The maximum-prefix ceiling |

## Removing BGP

Tearing down the configuration removes the sessions and the routes learned through them. Any
static routes remain — they were yours, not the protocol's.

If the fabric's only way out was a learned default route, removing BGP removes it. Add a static
default route first if you want the fabric to keep working.
