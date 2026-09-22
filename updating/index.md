---
title: "Releases"
description: "Versions, where they come from, and the two upgrade pipelines."
status: published
---

**Software** in the console shows what each part of the deployment is running and what is
available.

## Components

A deployment has several versions, and they are not one number:

| Component | Runs on |
|---|---|
| **OS bundle** | Every node — the operating system underneath |
| **Packages** | Every node — platform packages |
| **Agent** | Every host — the process the control plane talks to |
| **Orchestrator** | Controllers — the control plane itself |

```
Hyperion[vsnode1]> show version
Controller version: 2.0.0
```

A host's agent version is on its detail page in [Infrastructure](/console/infrastructure/).

## Where versions come from

Two routes:

- **The version repository** — Velarion's software repository, which the deployment syncs
  available releases from. The host it uses is configurable under Software settings.
- **Upload** — for sites with no route to that repository. See
  [Air-gapped sites](/updating/air-gapped/).

Available versions are listed; downloading one stages it, and it is then available to upgrade
to. Downloading is not upgrading.

## The two pipelines

Upgrades run as one of exactly two pipelines.

**Orchestrator** — `os → packages → agent → orchestrator`, across every controller. On a single
node that is one node; on a cluster it is a **rolling** upgrade: each standby in turn, then a
graceful failover, then the node that used to be active.

**Hosts** — `os → packages → agent`, **one hypervisor at a time**.

Both stage their artefacts onto the target node and hand the sequence to an upgrade manager that
runs under the node's own service manager. That is what lets it survive the reboot the OS stage
triggers — the control plane watches progress; it does not hold the upgrade open over a
connection that a reboot would drop.

## Progress

Each node in an upgrade reports a status:

| Status | |
|---|---|
| `PENDING` | Not started |
| `RUNNING` | In progress |
| `REBOOTING` | Restarting |
| `SYNCING` | Catching up after a restart |
| `SUCCESS` | Done |
| `FAILED` | Stopped with an error |
| `SKIPPED` | Not applicable |

`SYNCING` on a controller is the database catching up. A controller is not ready to be failed
over to until it leaves that state — see [High availability](/high-availability/).

The current upgrade and the history are both available, and an upgrade can be cancelled.

## Order

> **Warning** — Upgrade **controllers before hosts**. The control plane is what drives host
> upgrades, and a newer agent talking to an older control plane is not a combination to
> discover during a maintenance window.

## From the CLI

```
Hyperion[vsnode1]> request upgrade controller <version>
```

## Before upgrading

1. **Take a [backup](/data-protection/backups/)** and confirm it completed.
2. **Check cluster health** — every controller `ALIVE` and `SYNC OK`. An upgrade that begins
   with a standby already behind has less margin than you think.
3. **Check capacity** — hosts are emptied one at a time, so the rest of the zone must absorb
   their workloads.
4. **Read the [release notes](/support/release-notes/).**

## Next

- [Upgrading the fleet](/updating/fleet-upgrade/) — running it.
- [Air-gapped sites](/updating/air-gapped/) — doing it with no internet.
