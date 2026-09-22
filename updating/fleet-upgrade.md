---
title: "Upgrading the fleet"
description: "Running a rolling upgrade across controllers and hosts without dropping workloads."
status: published
---

An upgrade moves the fleet from one version to the next, a node at a time, while workloads keep
running.

## Before you start

| Check | Why |
|---|---|
| A recent [backup](/data-protection/backups/) that completed | The only way back from a bad upgrade |
| Every controller `ALIVE` and `SYNC OK` | A standby that is behind cannot take over |
| Spare capacity in each zone | Hosts are emptied one at a time |
| [Release notes](/support/release-notes/) read | Version-specific steps |
| A window | Nothing here should drop a VM, but "should" is not "will" |

## Step 1 — Download the version

From **Software**, download the release you want. This stages the artefacts; nothing is applied.

At an air-gapped site, upload the bundle instead. See
[Air-gapped sites](/updating/air-gapped/).

## Step 2 — Upgrade the controllers

Start the orchestrator pipeline. On a cluster it rolls:

1. Each **standby** is upgraded in turn — OS, packages, agent, orchestrator — rebooting and then
   syncing its database back up.
2. A **graceful failover** moves the active role to an upgraded node.
3. The node that used to be active is upgraded last.

The console is briefly unavailable during the failover in step 2 — seconds, at the virtual IP,
while it moves. Running VMs are unaffected throughout; the control plane is not in their data
path.

Watch each node reach `SUCCESS` and leave `SYNCING` before the next begins. If a node reaches
`FAILED`, stop and resolve it rather than continuing — a partially upgraded control plane is the
state you least want to extend.

## Step 3 — Upgrade the hosts

Start the hosts pipeline. Hosts are upgraded **one at a time**: OS, packages, agent.

For each host, that means workloads move off, the node reboots, and it rejoins. This is why zone
capacity matters — a zone with no headroom cannot empty a host without disturbing something.

Watch for VMs that will not move. A VM with a passthrough device or on local disk cannot
migrate, and will be stopped rather than moved. Know which those are **before** you start; see
[Live migration](/vms/live-migration/).

## Step 4 — Verify

```
Hyperion[vsnode1]> show cluster status
Hyperion[vsnode1]> show host
Hyperion[vsnode1]> show vm
Hyperion[vsnode1]> show version
```

- Every controller `ALIVE`, `SYNC OK`, exactly one `ACTIVE`.
- Every host `ONLINE` with the expected agent version.
- Every VM that was running, running.
- Storage healthy — check the storage cluster, not just the hosts.
- No new [system errors](/console/settings/).

## If a node fails

The upgrade stops on that node. It does not roll on and leave you with a fleet in three
different states.

1. Read the failure. The node's status carries it.
2. Check the node — is it reachable, did it reboot, is its agent connected?
3. Fix, then retry that node.
4. If it cannot be fixed, cancel the upgrade and get the fleet to a consistent state before
   trying again.

> **Warning** — Do not leave a fleet half-upgraded over a weekend. Mixed versions are supported
> *during* an upgrade, not as a steady state.

## Maintenance mode

For work that is not an upgrade — hardware, cabling, firmware — use maintenance mode directly:

```
Hyperion[vsnode1]> host maintenance vsnode2 on
   ... do the work ...
Hyperion[vsnode1]> host maintenance vsnode2 off
```

Nothing new is scheduled onto the host and its workloads move off. Remember to take it out
again; a host left in maintenance is capacity you are not using and headroom the zone is not
counting.

## Rolling back

There is no automatic rollback. Recovery from a bad control-plane upgrade is
[restoring a backup](/data-protection/backups/), which is why step one is confirming you have
one.

Note the schema rule: a backup can be restored onto the **same or a newer** version, never an
older one. A backup taken before the upgrade restores onto the upgraded control plane.
