---
title: "Routing and NAT"
description: "Getting traffic in and out of an overlay fabric, and translating addresses at the edge."
status: published
---

An L3 overlay fabric routes between its own networks by itself. Reaching anything **outside**
the fabric needs a deliberate exit: an **L3-out**.

This page applies to overlay fabrics. A VLAN fabric hands traffic to your physical switches and
your router is already the way out — there is nothing here to configure.

## East-west comes free

Inside an overlay fabric, every host runs a copy of the fabric's router. Traffic between two
networks in the fabric is routed on the host it starts from and tunnelled directly to the host
it is going to.

There is no central router to become a bottleneck, and no hairpin through a gateway node.

## L3-out: the way out

An L3-out is the fabric's edge. It is built on **exactly two hosts**, both members of the
uplink, and it is the one place the overlay touches your physical network.

Two, not one, because a single exit is a single point of failure. Two, not many, because the
edge is where policy and peering live and spreading that across the fleet makes it harder to
reason about, not easier.

### What you supply

| Field | Notes |
|---|---|
| Two chassis | Both must be members of the fabric's uplink |
| Transit VLAN | A VLAN that reaches your router, checked for conflicts on that uplink |
| Transit CIDR | The subnet shared with your router. **Minimum /30, /29 recommended** |
| Chassis addresses | One per chassis, inside the transit CIDR |
| Default route | Optional; the next hop must be inside the transit CIDR |

The fabric's router takes the **first usable address** in the transit CIDR. Your router takes
another. The two chassis addresses are used for peering and health checking.

A /30 leaves no room for anything beyond the bare minimum, which is why /29 is the
recommendation.

### The transit network

The transit segment is the only network in an overlay fabric with a port onto the physical
VLANs. Tenant networks deliberately have none — that absence is what keeps east-west traffic on
the overlay.

It is not listed as a tenant network and VMs do not attach to it.

### Failover

The two chassis are an HA pair. Only one is active at a time; the other stands ready. BFD runs
between the edge and your router so that a failure is detected in sub-second time rather than
by waiting for a routing protocol to time out.

`GET` the L3-out's status to see per-chassis BFD state. A standby that is not verifiably up is
not a standby.

## Static routes

Add static routes to the fabric's router to reach networks beyond your immediate next hop.

The route table distinguishes **static** routes, which you entered, from **learned** routes,
which came from BGP. The platform manages learned routes and never touches static ones. You can
delete a static route; you cannot delete a learned one, because it would come straight back.

See [BGP](/networking/bgp/).

## NAT

NAT rules are configured per fabric, at the edge.

| Type | Use |
|---|---|
| **SNAT** | Many internal addresses leave as one external address |
| **DNAT** | An external address reaches one internal address |
| **DNAT and SNAT** | A one-to-one mapping, both directions — a floating IP |

SNAT is what lets a fabric of private addresses reach the outside. A floating IP is how you
publish one VM without exposing the fabric.

> **Note** — NAT happens on the active L3-out chassis. All translated traffic goes through it,
> so it is the one place in an overlay fabric where throughput concentrates. East-west traffic
> is unaffected — it never goes near the edge.

## MTU inside the fabric

Guests on an overlay fabric are told the fabric MTU over DHCP. With the default profile that is
**8942** bytes.

A guest with a **static** address is not told anything and will use its own default, normally
1500 — which is safe. What is not safe is configuring a guest above the fabric MTU: those
packets have nowhere to go.

If you set MTUs inside guests by hand, keep them at or below the fabric MTU.

## Checking it works

| Question | Where to look |
|---|---|
| Is the L3-out up? | Its status, including per-chassis BFD |
| Are routes present? | The fabric's route table — static and learned |
| Is the transit VLAN trunked? | Your switch, and the uplink's MTU probe |
| Is NAT applying? | The rules list, and the guest's apparent source address |

A fabric that routes internally but cannot reach outside is almost always the transit VLAN not
being trunked to the two chassis, or a default route whose next hop is not reachable.
