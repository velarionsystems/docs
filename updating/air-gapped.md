---
title: "Air-gapped sites"
description: "Running and upgrading a deployment with no route to the internet."
status: published
---

VS-HCI is designed to run disconnected. The installation medium carries everything a node needs,
and nothing is fetched from the internet during installation.

## What works with no connectivity

| | |
|---|---|
| Installation | Fully — the medium carries an offline package set |
| All platform features | Yes |
| Built-in VM templates | Yes |
| Upgrades | Yes, by uploading bundles |
| Licensing | Yes — the request and the licence are files |

## What does not

| | Alternative |
|---|---|
| The template marketplace | Built-in templates, and your own |
| Syncing OS images | Upload ISOs to the [library](/storage/iso-library/) |
| Automatic version listing | Upload upgrade bundles |
| NTP to a public server | An internal time source |
| Alert email, webhooks, Slack | Internal endpoints |

## Time

NTP is the one that catches people out. The setup wizard defaults to a public pool, which an
air-gapped site cannot reach.

Point every node at an **internal** time source during setup. Clustering and TOTP both depend on
clocks agreeing, and a fleet with drifting clocks produces problems that look like everything
except a clock problem.

## Connectivity checks during setup

The installer tests the gateway, DNS and NTP, and a failure is a warning rather than a block —
precisely so that an air-gapped site can continue.

Read the results rather than clicking past them. `NTP FAIL` against an internal server means the
server is wrong, not that the site is disconnected.

## Upgrades

Upgrade bundles can be **uploaded** rather than downloaded. Fetch them on a connected machine,
bring them in, upload them from **Software**, then run the upgrade exactly as a connected site
would.

Both pipelines behave identically — the only difference is where the artefacts came from. See
[Upgrading the fleet](/updating/fleet-upgrade/).

Check the integrity of bundles you transfer. A truncated copy on removable media is a real
failure mode.

## Installation media

The medium is self-contained. Keep a copy of the exact version the fleet runs — rebuilding a
node at an air-gapped site means the medium you have is the only medium you have.

## Licensing

Licensing does not need connectivity. The deployment produces a request; you carry it out,
and carry the licence back in. Both are files.

See [Licensing](/administration/licensing/).

## ISOs and templates

Upload ISOs directly to the [ISO library](/storage/iso-library/), which replicates across
controllers and is exported to every host. Define your own
[templates](/vms/templates/) for the shapes you use.

## Backups

The backup target is an SCP destination, which can be entirely internal. Nothing about
[backup and restore](/data-protection/backups/) needs the internet — and the requirement that
backups live **off the cluster** applies exactly as it does anywhere else.

## Partially connected sites

Where a deployment has no internet but does have an internal mirror, point the version
repository host at it under Software settings. The deployment then lists and downloads versions
normally.

## Operating notes

- **Version the media.** Label which release each stick or image holds.
- **Keep release notes with the bundles.** You cannot look them up later.
- **Keep the signing key accessible.** [Root shell access](/cli/root-shell/) needs a key holder,
  and an air-gapped site is where you are most likely to need the operating system.
- **Test restores.** There is no support engineer who can dial in.
