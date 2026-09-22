---
title: "Configuration"
description: "What is configurable, where each setting lives, and the defaults."
status: published
---

Most configuration is in the console. A small amount is per node, set during first-boot setup
and managed by the platform afterwards.

## Where settings live

| Setting | Where |
|---|---|
| Console name, certificates | Settings → Certificates |
| Users, roles, directory, API keys | Settings → Users & Access |
| Licensing | Settings → Licensing |
| Controller priorities, VIP | Settings → Orchestrator Cluster |
| Backup target and schedule | Settings → Backup & Restore |
| DR pairing | Settings → DR Site |
| Version repository | Software settings |
| Alert rules | Alerts |
| Hostname, management network, NTP | First-boot setup |

> **Warning** — Node-level configuration is held in files the platform owns and reconciles.
> Editing them by hand is not supported: the platform rewrites what it manages, and the login
> banner on every node says as much. Change these through the console.

## Defaults worth knowing

### Networking

| | |
|---|---|
| Console and API port | 443 |
| Ports redirecting to it | 80, 8080 |
| Agent port | 9090 |
| Geneve overhead | 58 bytes |
| Default fabric MTU | 8942 (underlay 9000) |
| Selectable fabric MTUs | 9158, 8942, 4442, 1442 |

### Timing

| | |
|---|---|
| Metric collection | 10 seconds |
| Host heartbeat | 30 seconds |
| Host declared dead after | 30 seconds |
| Cluster heartbeat | 10 seconds |
| Alert evaluation | 60 seconds |
| Metric retention | 168 hours (7 days) |
| System error retention | 30 days |

### Sessions and authentication

| | |
|---|---|
| Access token lifetime | 24 hours |
| Refresh token lifetime | 7 days |
| Idle timeout | Per user, default 30 minutes |
| Account lockout | 5 failed attempts, 15 minutes |
| Sign-in rate limit | 10 per minute |
| API rate limit | 1000 per minute |

### Root shell

| | |
|---|---|
| Challenge validity | 10 minutes |
| Session lifetime | 60 minutes |

### Storage and VMs

| | |
|---|---|
| Local VM storage | `/var/lib/hyperion/vms` |
| ISO library | `/var/lib/hyperion/isos` |
| Snapshot policy retention | 5 |
| Snapshot policy default | Disk only |

### Licensing

| | |
|---|---|
| Evaluation period | 90 days |
| Licensed unit | Processor cores across the fleet |

## Disk layout

The installer creates an LVM volume group, `vshci-vg`:

| Mount | Volume | Minimum |
|---|---|---|
| EFI system partition | — | 1 GiB |
| `/boot` | — | 2 GiB |
| `/` | `vshci-os` | 8 GiB |
| `/var/lib/hyperion/vms` | `vshci-vm-local-storagepool` | 8 GiB |
| `/var/lib/hyperion/isos` | `vshci-iso` | 4 GiB |
| `/opt/hyperion` | `vshci-swrepo` | 4 GiB |
| `/var/tmp` | `vshci-scratch` | 2 GiB |
| swap | `vshci-swap` | 2 GiB |

Each volume gets its minimum, and what remains is shared out proportionally — so a larger system
disk grows all of them.

## Unattended install

A seed file at `/etc/hyperion/wizard-seed.conf` configures a node without prompting. See
[Creating install media](/getting-started/install-media/).

## Ports

See [Requirements](/getting-started/requirements/) for the full port matrix. VS-HCI does not
configure a host firewall — these are the ports it listens on, so you can configure the network
around it.

## Changing the console name

Set it under Settings → Certificates. It reissues the certificate, so the name you browse by and
the name on the certificate stay in step. See [Certificates](/administration/certificates/).
