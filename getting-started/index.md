---
title: "What VS-HCI is"
description: "Compute, storage and networking converged onto the same servers, run from one control plane."
status: published
---

VS-HCI is a hyperconverged infrastructure platform. You install it on bare metal, and each
server contributes CPU and memory for virtual machines, disks for shared storage, and network
interfaces for the virtual networks those machines sit on. There is no separate SAN, no
separate management server, and no hypervisor licence to track per socket.

One control plane — the **console** — runs on the servers themselves and manages all of them.

## What runs on a server

Every server in a VS-HCI cluster runs the same operating system image, **VSOS-HCI**, a hardened
Debian-based system that you do not administer directly. On top of it:

| Component | What it does |
|---|---|
| Hypervisor | KVM/QEMU, running your virtual machines |
| Host agent | Executes instructions from the control plane on this server |
| Virtual switch | Open vSwitch with OVN, providing the virtual networks |
| Storage | Ceph, pooling the disks across servers into replicated storage |
| Control plane | The console, API and scheduler — on controller nodes only |

A server that runs both workloads and the control plane is **hyperconverged**. That is the
normal shape, and the one the installer produces by default.

## Two deployment modes

The same installation medium produces either mode; you choose during first-boot setup.

**VS-HCI** is the clustered product. Three or more servers, a control plane that survives
losing one of them, shared storage replicated across nodes, and live migration between hosts.

**VS-Hypervisor** is a single server. Same console, same virtual machines, same networking —
but with one node there is nothing to fail over to, so controller high availability, live
migration and replicated storage do not apply.

> **Note** — This library documents VS-HCI. Where a single-node deployment behaves
> differently, the page says so.

## How you drive it

There are three ways in, and they talk to the same API.

- **The console** — the web interface, served over HTTPS on port 443 at the cluster's virtual
  IP or DNS name. This is where nearly everything happens.
- **The CLI** — SSH to any node lands you in the VS-HCI shell rather than a Linux prompt. It
  is the tool for checking state when the console is unreachable, and for getting to the
  operating system underneath when you genuinely need to. See [The VS-HCI shell](/cli/).
- **The REST API** — everything the console does, it does through this. See
  [REST API](/reference/api/).

## What a cluster looks like

A minimal production cluster is three servers. That number is not arbitrary: the control plane
elects an active node, and electing anything requires a majority, which needs an odd number
greater than one.

```
                    ┌─────────────────────────┐
                    │   Virtual IP (console)  │
                    └───────────┬─────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
   ┌────▼─────┐           ┌─────▼────┐            ┌─────▼────┐
   │  node1   │           │  node2   │            │  node3   │
   │ ACTIVE   │           │ STANDBY  │            │ STANDBY  │
   │          │           │          │            │          │
   │ VMs      │           │ VMs      │            │ VMs      │
   │ storage  │◄─────────►│ storage  │◄──────────►│ storage  │
   └──────────┘           └──────────┘            └──────────┘
```

One node holds the virtual IP and serves the console; the others stand by with a synchronised
copy of the database. All three run virtual machines. If the active node fails, a standby takes
the virtual IP and the console comes back at the same address.

## Where to start

1. [Requirements](/getting-started/requirements/) — what each server needs.
2. [Creating install media](/getting-started/install-media/) — writing the image.
3. [Installing the first server](/getting-started/first-server/) — first-boot setup.
4. [Adding servers](/getting-started/adding-servers/) — growing to a cluster.
5. [First sign-in](/getting-started/first-sign-in/) — what to do once the console is up.
