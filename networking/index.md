---
title: "Networking"
description: "How the layers fit together, from physical NICs up to the networks a VM attaches to."
status: published
---

VS-HCI networking is software-defined. The physical network carries packets between servers;
everything a virtual machine sees is built on top by the platform, consistently across every
host.

## The layers

```
   VM  ──►  network on a distributed switch
              │
              ▼
          distributed switch  (L2GDS / L3GDS / SGDS / RGDS / MGDS)
              │
              ▼
          uplink  (a bond of NICs behind a bridge, on each member host)
              │
              ▼
          physical NICs ──► your switches
```

Read it bottom-up:

1. **Uplinks** claim physical NICs, bond them, and put a bridge on top. One uplink spans many
   hosts, with a member on each. See [Uplinks and bonds](/networking/uplinks-and-bonds/).
2. **Distributed switches** sit on an uplink and provide networks that exist identically on
   every member host. See [Distributed switches](/networking/distributed-switches/).
3. **VMs** attach to a network on a switch. A VM keeps its network when it moves hosts,
   because the switch is on all of them.

## Why nothing is configured at install time

The installer configures exactly one thing: the management link. Every other network is created
afterwards, from the console.

That is deliberate. An uplink picks NICs, bond mode, LACP parameters and MTU together and can
verify them against your switches — decisions that need to be made as a set, by someone who can
see the result. A network pre-created during installation would be a guess.

## Encapsulated or not

The distinction that drives most of the design:

| | Not encapsulated | Encapsulated (Geneve) |
|---|---|---|
| Used by | L2 switches, storage, replication, management | L3 overlay fabrics |
| VLANs | Handed off to your switches as 802.1Q | Carried inside tunnels |
| Needs underlay IPs | No | Yes, one per host |
| MTU | The uplink's wire MTU | Wire MTU minus 58 bytes |
| Your switches must | Trunk the VLANs | Route between hosts and pass UDP 6081 |

A VLAN-based network hands traffic to your existing switches and your existing router is the
gateway. An overlay network carries tenant traffic inside tunnels between hosts, and the
platform routes it.

## MTU

Geneve encapsulation costs **58 bytes**. The platform tracks two numbers per uplink:

- **Underlay MTU** — what the physical path must carry.
- **Fabric MTU** — what the overlay can offer, 58 bytes smaller.

Four profiles are selectable:

| Fabric MTU | Underlay MTU | |
|---|---|---|
| 9158 | 9216 | Maximum jumbo |
| **8942** | **9000** | Default — standard jumbo |
| 4442 | 4500 | |
| 1442 | 1500 | Legacy, for a network that cannot do jumbo |

The default assumes your switches carry 9000-byte frames. If they do not, choose 1442/1500 —
and expect storage replication to be slower for it.

> **Note** — Networks that are *not* encapsulated — storage, replication, management — carry
> the uplink's **wire** MTU, not the overlay MTU. They also follow the uplink: raise the
> uplink to jumbo and those ports come with it.

## The control plane stays out of the data path

The platform distributes topology to every host and then gets out of the way. Packets between
two VMs go host-to-host; they do not traverse a controller. A control plane outage does not
stop traffic that is already flowing.

## Where to go next

| Task | Page |
|---|---|
| Claim NICs, build bonds | [Uplinks and bonds](/networking/uplinks-and-bonds/) |
| Give VMs networks | [Distributed switches](/networking/distributed-switches/) |
| Isolated routed domains | [Fabrics](/networking/fabrics/) |
| Get traffic in and out | [Routing and NAT](/networking/routing-and-nat/) |
| Peer with your routers | [BGP](/networking/bgp/) |
| Filter traffic | [Security policies](/networking/security-policies/) |
