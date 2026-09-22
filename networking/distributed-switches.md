---
title: "Distributed switches"
description: "The five kinds of global distributed switch, and which one a given job needs."
status: published
---

A **Global Distributed Switch** (GDS) provides networks that exist identically on every host
that is a member of its uplink. A VM attached to one keeps its network wherever it runs, which
is what makes live migration work without touching guest configuration.

There are five kinds. They are not five variations on a theme — each exists for a specific job.

| Kind | Carries | Encapsulated | Uplink requirement |
|---|---|---|---|
| **L2GDS** | Tenant VLANs handed off to your switches | No | L2-ready |
| **L3GDS** | Routed overlay networks | Yes (Geneve) | Underlay IPs + MTU probe |
| **MGDS** | The management network | No | The installer's mgmt uplink |
| **SGDS** | Storage traffic | No | Any |
| **RGDS** | Storage replication | No | Must be **dedicated** |

A live cluster shows one of each kind in use:

```
NAME         KIND     STATUS
mgmt         MGMT     ACTIVE    Networks on the management / OOB link
l2apps       DATA     ACTIVE
storage      STORAGE  ACTIVE
replication  REPL     ACTIVE
```

## L2GDS — Layer 2

The simplest and the one most deployments start with. It is defined by what it does **not**
need: no tunnel identifiers, no underlay addressing, no MTU probe, no subnet, no DHCP, no
router, no NAT.

You give it an uplink whose bond is up, and a set of VLANs. Each VLAN becomes one network,
presented as a tagged port on the uplink's bridge, landing on exactly the uplink's member hosts.

Traffic is handed to your physical switches as ordinary 802.1Q. **Your** router is the gateway;
the platform does no routing and sees no north-south traffic.

Use an L2GDS when:

- you already have VLANs and a router, and want VMs on them,
- your network team owns addressing and routing,
- you want the least new machinery between a VM and the wire.

> **Note** — The uplink's switch ports must trunk the VLANs you use. A VLAN that is not on the
> trunk produces a network that exists in the console and passes nothing.

## L3GDS — Layer 3 overlay

A Geneve overlay fabric. Creating one is what brings the underlay into existence: each member
host gets an address on a `vtep-<uplink>` interface, an MTU probe mesh verifies the path can
carry the frames, and the host's tunnel endpoint moves from its placeholder management address
onto that VTEP address.

After that, overlay traffic rides the bond. A management NIC failure no longer touches tenant
forwarding — which is the main reason to do it.

Use an L3GDS when:

- you want networks that are not VLANs on your physical switches,
- you want tenant isolation the physical network does not need to know about,
- you want the platform to route between tenant networks.

The addressing decision lives here, not in the uplink. That is deliberate: underlay IPs are an
overlay concern, and an uplink that is only ever going to carry VLANs never needs them.

## MGDS — Management

The management network, adopted from what the installer built. It exists so that the management
link is a first-class object like everything else rather than a special case outside the model.

You do not usually create one; it is adopted.

## SGDS — Storage

One per cluster, carrying exactly one network, whose subnet gives every host an address.

The point is separation. Without it, the iSCSI portal, the storage VIP, the storage monitors
and the addresses hypervisors connect to all resolve to the host's **management** address — so
storage I/O shares a link with the control plane, and a management problem becomes a storage
outage.

No policy routing is needed to make this work: the storage VIP comes from the same pool as the
host addresses, so each host's connected route for that subnet already beats the management
default route. A gateway is optional and only needed for an external array or an off-subnet
initiator.

Structurally an SGDS *is* an L2GDS with an address pool on top.

## RGDS — Replication

Physical isolation for replication traffic.

The problem it solves: a resync runs at line rate for hours, and storage rebuilds happen
whenever a disk is replaced. If replication shares wires with client I/O, a rebuild degrades
the very storage it is rebuilding — and the two contend exactly when both are busiest.

A second VLAN on the storage network gives a separate broadcast domain and separate addresses,
but the same cables. An RGDS goes further: **its uplink must be dedicated**, so replication
rides different NICs into different switch ports and cannot contend at all.

The platform refuses to build an RGDS on a shared uplink. Offering VLAN isolation while calling
it physical isolation would leave you believing you had bought separation you do not have.

An RGDS carries up to 32 networks — a segment per storage cluster rather than one for the
fleet.

## Choosing

| You want | Use |
|---|---|
| VMs on existing VLANs | L2GDS |
| Networks independent of the physical switches | L3GDS |
| Storage off the management link | SGDS |
| Replication off the client storage path | RGDS + a dedicated uplink |

Most deployments end up with an L2GDS for workloads, an SGDS for storage, and an RGDS once
storage is busy enough to matter.

## VLAN allocation

VLAN IDs are tracked centrally, so two switches cannot claim the same VLAN on the same uplink.
A conflicting request is rejected rather than silently producing two networks that interfere.

## MTU on switch ports

Non-encapsulated networks — storage, replication, management — carry the uplink's **wire** MTU
and follow it when it changes. You can override the MTU per network, but not above what the
uplink's trunk carries.

The 802.1Q tag is not subtracted: a tagged port gets the same MTU as an untagged one.
