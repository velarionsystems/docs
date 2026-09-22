---
title: "A tour of the console"
description: "What each screen in the VS-HCI console is for, and which one to reach for."
status: published
---

The console is the web interface to the whole deployment. It is served over HTTPS on port 443
at the cluster's virtual IP, from whichever controller is currently active.

Everything in it is the REST API underneath — there is nothing the console can do that the
[API](/reference/api/) cannot, and nothing it hides from the [CLI](/cli/).

## The layout

A fixed left-hand navigation, grouped by what each screen acts on. The groups follow the shape
of the platform rather than the shape of the menu:

| Screen | Acts on | Page |
|---|---|---|
| **Dashboard** | The fleet, at a glance | [Dashboard](/console/dashboard/) |
| **Infrastructure** | Servers, availability zones, datacenters | [Infrastructure](/console/infrastructure/) |
| **Approvals** | Nodes waiting to join | [Approvals](/console/approvals/) |
| **VMs** | Virtual machines | [Virtual machines](/vms/) |
| **Containers** | Container workloads | [Containers](/workloads/containers/) |
| **Kubernetes** | Managed clusters | [Kubernetes](/workloads/kubernetes/) |
| **Networks** | Uplinks, switches, fabrics, routers | [Networking](/networking/) |
| **Storage** | Pools, volumes, replicated storage, exports | [Storage](/storage/) |
| **Tasks** | Asynchronous work in flight | [Tasks](/console/tasks/) |
| **Alerts** | Rules and the events they raised | [Alerts](/monitoring/alerts/) |
| **Backups** | Control-plane backup and restore | [Backup and restore](/data-protection/backups/) |
| **Software** | Versions, packages, upgrades | [Updating](/updating/) |
| **Settings** | Everything administrative | [Settings](/console/settings/) |

## Things that behave the same everywhere

### Work happens as tasks

Anything that takes more than an instant — creating a VM, migrating one, building a storage
cluster, upgrading a node — becomes a **task**. The console returns immediately with the task
created, and progress streams in live over a WebSocket.

This means a slow operation never blocks the screen, and closing the browser does not cancel
anything. See [Tasks](/console/tasks/).

### The active controller serves writes

The console is served by whichever controller holds the virtual IP. If you reach a standby
directly — by browsing to a node's own address rather than the VIP — reads work, but a write
returns **HTTP 409** naming the active node instead of committing.

Use the VIP or the console name and this never arises.

### Permissions are enforced by the API

What you can see and do is decided server-side, per domain. A user with read-only rights over
networking sees networking and is refused when they try to change it — the button may be there,
the answer is still no. See [Users and roles](/administration/users-and-roles/).

### Cluster-only features hide themselves

On a single-node deployment the screens that only mean something with peers — Approvals,
controller HA, live migration — are not shown. They appear when the deployment becomes a
cluster.

## Your own account

The **Profile** screen holds your password, your two-factor enrolment and your API keys.
Anything that affects other people is under **Settings** instead.

## Keyboard and search

The sidebar has a filter box that narrows the navigation as you type — faster than hunting
through groups once the deployment has a lot in it.
