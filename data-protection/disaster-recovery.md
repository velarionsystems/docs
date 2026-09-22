---
title: "Disaster recovery"
description: "Pairing two deployments, replicating VMs between them, and bringing them up at the other site."
status: published
---

Disaster recovery pairs this deployment with **one other** VS-HCI deployment, replicates
selected VMs to it, and lets you bring them up there — either deliberately while both sites are
healthy, or after the first site is gone.

## Pairing two sites

Pairing is **symmetric**. Neither site is "the main site"; the pairing says only that the two
deployments know each other. Which one runs a given workload is a property of each *zone link*,
and one site can hold main zones and DR zones at the same time.

That is what lets a workload migrated to DR replicate back without re-pairing — a failback is a
second migration in the other direction, not an inversion of the relationship.

### The handshake

1. **Site A** mints a pairing token containing the pair identity, a shared secret, A's virtual
   IP and console name, and A's certificate authority.
2. **You carry the token to site B by hand.**
3. **Site B** accepts it: it fetches A's CA independently to confirm what the token claims, then
   contacts A — verified against that CA and authenticated with the secret — handing back its
   own virtual IP and CA.
4. **Site A** stores B's half. Both sides are paired.

The token travels through a person because there is no prior relationship to protect it with.
These are two deployments that have never met; any channel that could carry the token safely
would already be the trust the pairing is trying to establish.

### The trust is separate

Site-to-site traffic uses its **own** trust anchor and its **own** secret, distinct from the
ones controllers of the same cluster use between themselves.

That separation is deliberate. If the far site could authenticate as a cluster peer, it could
enrol itself into the local cluster, take the virtual IP and be handed the master key on the way
in. Two trust domains, two anchors, two secrets.

A deployment can be paired with exactly **one** other.

## Linking zones

Pairing connects the deployments; a **zone link** connects a main-site availability zone to a
DR-site zone. That is the unit of failover.

Each side checks its own half of the preconditions, and the receiving side re-runs the
equivalent checks rather than taking the initiator's word — a link accepted on a claim is a link
that fails during the recovery it was created for.

| Precondition | Why |
|---|---|
| One zone is MAIN, the other DR | Two mains would be two owners; two DRs, none |
| **Ceph on both sides** | The only backing with a per-VM replication unit |
| Replication addresses on every host | Mirror traffic has to have a path |

> **Warning** — DR requires replicated storage at **both** sites. An iSCSI zone's LUNs are
> logical volumes on one shared resource, so there is nothing to replicate per VM. A pair with
> different backings cannot replicate at all.

## Protecting a VM

Marking a VM **DR capable** does four things, in this order:

1. **Ask the far site whether it has room** — before committing, not during an outage.
2. **Enable mirroring on its images** — the data starts moving.
3. **Ask the far site to build the placeholder** — reserving its memory for real.
4. **Mark the local copy primary.**

Step 1 is first because its answer decides whether the rest should happen at all, and the time
to discover that DR has no room is while everything is healthy.

### The placeholder is a real VM

At the far site the VM appears as `DR_PLACEHOLDER`. It has the **same identity** as the original
— both rows are the same VM — with real disks, real NICs, and a host assigned by that zone's own
placement.

The host assignment is what makes its memory genuinely reserved rather than merely promised: the
zone's capacity arithmetic counts VMs per host, so a placeholder without one would be invisible
and the far site would cheerfully promise the same gigabyte twice.

**It is never started on that side.** Its disks are the receiving end of a live mirror;
starting it would put a second writer on images the other site's hypervisor is still writing.

## How replication works

**Snapshot-based, not journal-based.** Journal mirroring writes every operation to a journal
before the image takes it. It buys a near-zero recovery point and costs roughly double the write
latency **on the production side**, and it stalls when the far cluster cannot keep up — a DR
feature that degrades the thing it protects, at the site that is still healthy.

Snapshot mode ships deltas between scheduled mirror snapshots. Your recovery point becomes the
schedule rather than zero, which is the honest trade for a link between two buildings, and a
slow link shows up as **observable lag** instead of as latency inside your guests.

**Per VM, not per pool.** Only the VMs you protect are replicated. Pool-wide mirroring would
spend the inter-site link on guests nobody asked to protect.

The console reports replication progress and how far behind each protected VM is. Lag is the
number to watch — it is your actual recovery point, as opposed to the one you configured.

## Planned migration

Both sites healthy, and you want the workload to run at the other one — a rehearsal, a
maintenance window, a failback.

```
1  shut the guest down here          graceful, then forced after a timeout
2  final mirror snapshot, and wait   drives the recovery point to zero
3  tell the far site it is taking over
4  demote here, promote there        refused if step 3 did not really land
5  start it there
6  reverse the mirror                this copy becomes the replica
```

**Any step that does not succeed aborts the whole thing** and leaves the guest running where it
was. Both sites are up, so there is always the option of changing nothing.

Nothing is lost: step 2 takes a final snapshot before the handover.

## Unplanned recovery

The main site is gone.

### It is manual, deliberately

When the links between sites are cut, neither site can distinguish "the peer is dead" from "I am
the one that has been cut off". Products that automate this put a witness in a third failure
domain to break the tie. **There is no third site here**, so nothing could make an automatic
decision correct — and one is not offered.

What there is instead is a person who can see more than either controller can: whether the
building is actually on fire.

### The button is at the DR site

Recovery is performed **at the DR site's own console**, on the DR zone. That is the only console
still reachable when the main site is gone. A button at the main site would be unusable in
exactly the situation it exists for.

### The guard

At the moment you click, the DR site **re-probes the main site's virtual IP** and refuses if it
answers.

A cached line state is not good enough: the whole hazard is that the state was gathered while
*this* site was the isolated one. A stale "the peer is down" is precisely what would put two
copies of a guest on one mirror.

If the peer answers, planned migration is available and is what you should use.

### Two independent lines

The link's health is watched on two separate signals: whether the zone's hosts can still reach
the far site's storage monitors, and whether the control planes can reach each other.

Two signals, because one signal wearing two labels is not redundancy. A zone is considered
isolated only when both are down.

## When the other site comes back

Every promotion increments **that guest's** generation on both sides. When the sites meet again,
the higher generation is the side that acted last for that VM.

The lower one never starts it, and is marked as needing a resync.

> **Warning** — A resync discards whatever the losing copy wrote after the split. That is a
> real loss of data, so it is left to an operator rather than taken automatically. Look at what
> that copy did before you accept it.

## Failing back

A failback is a **planned migration in the other direction**. Once the original site is healthy
and replication has caught up, migrate the workload back. No re-pairing, no reconfiguration —
the pairing was symmetric from the start.

## What DR does not cover

- **Unprotected VMs.** Only VMs marked DR capable are replicated.
- **Configuration.** The far site is told which zone, which images, which VM and who owns it —
  and nothing else. No hosts, no networks in general, no users, no licences. For the control
  plane's own configuration, see [Backup and restore](/data-protection/backups/).
- **More than one peer.** One pairing per deployment.
