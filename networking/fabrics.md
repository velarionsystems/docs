---
title: "Fabrics"
description: "Isolated routed domains — VRFs — and the two types they come in."
status: published
---

A fabric is a **VRF**: an isolated network domain containing one or more networks. Two fabrics
do not reach each other unless you deliberately connect them, even where they use the same
addresses.

Use fabrics to separate tenants, environments or security zones that must not see each other.

## Two types

### L2_VLAN

Each network gets a port that hands its VLAN to your physical switches. There is no
encapsulation, and **your** firewall or router is the gateway.

The platform provides the switching; everything north-south is yours. Traffic leaving a VM
reaches your network as tagged Ethernet and is routed by whatever routes your VLANs today.

### L3_OVERLAY

East-west traffic is Geneve-encapsulated between hosts and tenant networks have no port onto
the physical VLANs. The platform routes **inside** the fabric, and traffic leaves through a
deliberate exit — see [Routing and NAT](/networking/routing-and-nat/).

This gives isolation the physical network does not need to know about, and overlapping address
space between fabrics.

## Which to choose

| | L2_VLAN | L3_OVERLAY |
|---|---|---|
| Gateway | Yours | The platform |
| North-south control | Your firewall | The platform, plus your firewall beyond it |
| Physical switch config | Trunk every VLAN | Route between hosts; pass UDP 6081 |
| Overlapping subnets between fabrics | No | Yes |
| Default security posture | Anti-spoof and established only | Full ladder including default-deny |

Choose **L2_VLAN** when your network team owns routing and policy, and you want the platform to
provide switching only. Choose **L3_OVERLAY** when you want isolation and routing without
changing the physical network for every new segment.

## The control plane stays out of the data path

The platform distributes topology to every host and does not forward packets. Two VMs on the
same fabric talk host-to-host directly.

A control-plane outage therefore does not stop existing traffic. You cannot *change* the network
while it is down, but what is running keeps running.

## Networks inside a fabric

A fabric contains networks. In an L2_VLAN fabric each is a VLAN segment handed to your
switches; in an L3_OVERLAY fabric each is a routed segment the platform owns.

VMs attach to networks, not to fabrics directly.

## Security posture

A fabric's default security rules are chosen for its type, and the difference is honest about
what the platform can actually enforce.

An L2_VLAN fabric's north-south traffic never passes anything the platform controls, so it gets
anti-spoofing and established-connection handling but **no default drop** — the policy point is
your firewall, and pretending otherwise would be a false sense of security.

An L3_OVERLAY fabric routes through the platform, so it gets the full ladder including
default-deny.

See [Security policies](/networking/security-policies/).

## Fabrics and distributed switches

Fabrics and distributed switches overlap: a Layer-3 distributed switch keeps a fabric behind it
so that the fabric-keyed features — L3-out, BGP, NAT, security policy, load balancing — work
against it unchanged.

For new work, build on [distributed switches](/networking/distributed-switches/) and use the
fabric view for routing, BGP and policy.
