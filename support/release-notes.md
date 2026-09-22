---
title: "Release notes"
description: "Where to find what changed, and what to read before upgrading."
status: published
---

Release notes for each version are published alongside the release and are listed in the console
under **Software** for versions your deployment can see.

An [air-gapped site](/updating/air-gapped/) has no version listing — keep the notes with the
bundles you bring in, because you cannot look them up afterwards.

## Reading them before an upgrade

Look for four things:

1. **Behaviour changes** — anything that works differently after the upgrade.
2. **Required steps** — actions to take before or after, in order.
3. **Deprecations** — things still working now that will not be.
4. **API changes** — if you automate against the
   [REST API](/reference/api/).

## Versions in a deployment

A deployment carries several versions, and they move together during an upgrade:

| Component | Where |
|---|---|
| OS bundle | Every node |
| Packages | Every node |
| Agent | Every host |
| Orchestrator | Controllers |

```
Hyperion[vsnode1]> show version
Controller version: 2.0.0
```

A host's agent version is on its detail page in
[Infrastructure](/console/infrastructure/), and the fleet's versions are under **Software**.

## Upgrade order

> **Warning** — Controllers before hosts. A newer agent talking to an older control plane is
> not a combination to discover during a maintenance window.

See [Upgrading the fleet](/updating/fleet-upgrade/).

## Before any upgrade

1. A [backup](/data-protection/backups/) that **completed**.
2. Every controller `ALIVE` and `SYNC OK`.
3. Spare capacity in each zone, because hosts are emptied one at a time.
4. The release notes.

## Restoring across versions

A backup restores onto the **same or a newer** version. A **newer** archive onto an older
control plane is refused — it would replace the database and then fail to start.

So a backup taken before an upgrade restores onto the upgraded control plane, which is the
direction you need.

## Getting support

Quote:

- your **installation identity**, from Settings → Licensing,
- the **version** each component is running,
- the **error ID** or **task ID**, if there is one.

Your support contact is the one on your agreement with Velarion Systems.
