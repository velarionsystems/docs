---
title: "Requirements"
description: "What each server needs before you boot the installation medium."
status: published
---

## Per server

| | |
|---|---|
| CPU | x86-64 with hardware virtualization — Intel VT-x or AMD-V, enabled in firmware |
| RAM | 16 GB to run the platform with room for workloads; more is the usual case |
| Disk | One system disk, plus any number of additional disks for replicated storage |
| Network | At least one interface for management; see below |
| Firmware | UEFI or BIOS — the installer handles both |

> **Warning** — Hardware virtualization must be **enabled in the server's firmware**, not
> merely supported by the CPU. If it is off, the node installs and joins normally and then
> cannot start a single virtual machine.

### System disk

The installer partitions the system disk into an LVM volume group, `vshci-vg`, with a fixed
set of volumes. Each volume has a minimum size and a share of whatever space is left over:

| Mount point | Volume | Minimum |
|---|---|---|
| EFI system partition | — | 1 GiB |
| `/boot` | — | 2 GiB |
| `/` | `vshci-os` | 8 GiB |
| `/var/lib/hyperion/vms` | `vshci-vm-local-storagepool` | 8 GiB |
| `/var/lib/hyperion/isos` | `vshci-iso` | 4 GiB |
| `/opt/hyperion` | `vshci-swrepo` | 4 GiB |
| `/var/tmp` | `vshci-scratch` | 2 GiB |
| swap | `vshci-swap` | 2 GiB |

Those minimums add up to roughly **36 GiB**, which is the floor rather than a
recommendation — a disk at that size leaves nothing for local VM images, ISOs or downloaded
upgrades. Give the system disk a few hundred GB. Everything above each volume's minimum is
shared out proportionally, so a larger disk grows all of them rather than stranding space.

### Disks for replicated storage

Disks you intend to contribute to replicated storage should be left alone — do not partition
them. They are claimed whole when you build a storage cluster. See
[Replicated storage](/storage/replicated/).

## Network

The management network carries the console, the API, the control plane's own traffic and the
agent connections. Every server must reach every other server on it.

| | |
|---|---|
| Addressing | Static is strongly preferred, especially on controllers |
| VLAN | Any ID from 1 to 4094, presented either untagged on an access port or tagged on a trunk |
| Virtual IP | One free address in the management subnet, not assigned to any server |
| DNS | Optional, but a name for the virtual IP is worth having — it goes on the TLS certificate |
| NTP | Required and reachable; the control plane's clustering is time-sensitive |

If the management interface is on a **tagged** VLAN, the switch port must be a trunk carrying
that VLAN before you start. The installer cannot verify connectivity on a tagged VLAN until it
has built the bridge, so a mis-cabled trunk shows up late in the process.

Additional interfaces for workload traffic, storage replication and so on are **not** configured
during installation. You create those afterwards, from the console, as
[uplinks](/networking/uplinks-and-bonds/).

### Ports

Between servers, on the management network:

| Port | Protocol | Purpose |
|---|---|---|
| 443 | TCP | Console and API |
| 80, 8080 | TCP | Redirect to 443 |
| 22 | TCP | SSH — the VS-HCI shell, and control-plane access to hosts |
| 5432 | TCP | Database replication between controllers |
| 5403 | TCP | Cluster quorum |
| 2224 | TCP | Cluster resource manager |
| 111, 2049 | TCP/UDP | NFS, for the shared ISO library |
| 6641, 6642 | TCP | Virtual network control plane |
| 6081 | UDP | Geneve — the tunnels carrying virtual network traffic |
| 3300, 6789, 6800–7300 | TCP | Replicated storage |

From client machines you need only **443** to the virtual IP, and **22** if operators will use
the CLI.

> **Note** — VS-HCI does not configure a host firewall. These are the ports the platform
> listens on, so that you can configure the network between and in front of the servers.

## Overlay MTU

Virtual network traffic is encapsulated in Geneve, which costs 58 bytes. On a standard 1500-byte
underlay the resulting MTU inside virtual networks is **1442**, and that is what the platform
configures by default.

If your physical network supports jumbo frames, raising the underlay MTU raises the overlay MTU
with it, and is worth doing for storage replication in particular.

## Browser

The console is a current-generation web application and expects a current browser — recent
Chrome, Edge, Firefox or Safari. It is served over HTTPS with a certificate the cluster
generates itself, so the first visit warns about an unknown issuer until you either accept it or
install a certificate from your own authority. See [Certificates](/administration/certificates/).

## Cluster shapes

| Shape | Servers | What you get |
|---|---|---|
| Single node | 1 | Everything except controller failover, live migration and replicated storage |
| Cluster | 3 controllers | Control plane survives losing one node; storage replicates; VMs migrate |
| Cluster + hosts | 3 controllers + any number of hosts | As above, with workload capacity that does not run the control plane |

Three controllers is the practical minimum for a cluster, because electing an active node
requires a majority.
