---
title: "Uplinks and bonds"
description: "Claiming NICs into bonds, the MTU profile, and proving the path before you build on it."
status: published
---

An uplink is the foundation of everything else. It claims physical NICs on each host, bonds
them, and puts a bridge on top. One uplink object spans the fleet; each host contributes a
**member**.

## What an uplink creates on a host

For each member:

- a **bond** over the NICs you named,
- a **bridge** named `br-<uplink>`, carrying the bond as its only port, in trunk mode,
- a bridge mapping so the virtual switching layer can reach it.

A live cluster's uplinks look like this:

| Uplink | Bond | NIC | Bridge | Dedicated |
|---|---|---|---|---|
| `mgmt` | `bond99` | `eno1` | `br-mgmt` | no |
| `data-stor` | `bond100` | `enp2s0` | `br-data-stor` | no |
| `repl` | `bond101` | `enp3s0` | `br-repl` | **yes** |

The `mgmt` uplink is built by the installer. The others you create.

## Creating one

**Networks → Uplinks → Create**. Name it, choose the MTU profile, and say whether it is
dedicated. Then add a member per host, choosing that host's NICs.

Only NICs that are genuinely free are offered: up, without an IP address, not already enslaved
to a bond, and not a member of another uplink. A NIC carrying the management address is not on
the list, and should not be.

### Bond mode

`active-backup` needs no switch configuration and survives a NIC or switch failure. LACP
(`802.3ad`) aggregates bandwidth and requires a matching port-channel on the switch side.

Use LACP where you control the switch configuration and want the throughput; use
`active-backup` where you want resilience without coordinating with the network team.

> **Warning** — LACP with no matching port-channel gives an uplink that looks configured and
> passes traffic unpredictably. Configure the switch first.

### MTU profile

The MTU is a property of the uplink and is chosen from fixed profiles — it is not a free-text
field, because the fabric MTU and the underlay MTU have to stay 58 bytes apart.

| Fabric MTU | Underlay MTU |
|---|---|
| 9158 | 9216 |
| **8942** | **9000** (default) |
| 4442 | 4500 |
| 1442 | 1500 |

### Dedicated

Marking an uplink dedicated declares that its NICs carry nothing else. Replication switches
require a dedicated uplink, and refuse a shared one — a shared uplink would give VLAN
separation while claiming physical separation, which is worse than not offering it at all. See
[Distributed switches](/networking/distributed-switches/).

## Proving the path

An uplink is not `READY` because it was configured. It is `READY` when every member is plumbed
**and** an MTU probe has passed between every pair of member hosts.

The probe sends packets at the configured size with fragmentation forbidden, host to host,
across the mesh. If the path cannot carry them, it fails loudly with the output.

An uplink that is plumbed but unprobed stays `PENDING`. Run the probe from the uplink's screen.

> **Note** — This is the check that catches a switch that was not set to jumbo frames. Without
> it, everything looks correct and large packets disappear silently — which surfaces later as
> storage that is mysteriously slow, or a guest that can ping but cannot transfer.

## Adding a host later

Add it as a member of the uplink. Hosts joining later are plumbed and enrolled the same way as
the ones present at creation — nothing is skipped because a host looks like it was done
already.

Re-run the probe after adding members: the mesh is bigger than it was.

## Changing an uplink

Bond mode and MTU are properties of the uplink and apply to every member. Raising the MTU
raises it on the bond, its members and the non-encapsulated networks that follow the uplink.

> **Warning** — Changing the MTU on a live uplink is disruptive. The physical path must be
> able to carry the new size *before* you change it, or every member loses large packets at
> once. Change the switches first, then the uplink, then re-probe.

## Removing

Removing an uplink tears down the bridge and the bridge mapping and removes its persisted
configuration, **leaving the bond intact**. An uplink that a distributed switch is built on
cannot be removed until that switch is gone.

## The management uplink

The installer builds `mgmt` from the interface you chose during setup, converting it to a
bonded, bridged layout with the management address on an internal port. It is a normal uplink
afterwards and can carry other networks.

Whether you *should* put workload traffic on it is a different question. A management link that
is saturated by a guest is a management link you cannot troubleshoot through.
