---
title: "Diagnostics"
description: "Read-only checks that run on a host and return their output."
status: published
---

Diagnostics are a curated set of read-only commands that run on a host and return what they
printed. They are available from a host's **Diagnostics** tab in
[Infrastructure](/console/infrastructure/), and from the CLI.

```
Hyperion[vsnode1]> show diagnostics
Hyperion[vsnode1]> run diagnostic <key>
```

Every one is read-only and safe to run at any time, including during an incident.

## Why it is a fixed catalogue

The set is fixed, and that is the security boundary rather than an inconvenience.

Diagnostics run with privilege on the host. If the platform accepted an arbitrary command it
would be an arbitrary-execution endpoint wearing a diagnostic label — and the restrictions that
make [root shell access](/cli/root-shell/) an authorized, audited act would be meaningless,
because there would be an unaudited way around them.

The commands are also re-validated on the host, not only chosen in the console. A check the
caller could bypass is not a check.

## The catalogue

### System

| Key | Shows |
|---|---|
| `sys.uptime` | Uptime and load |
| `sys.os` | OS release |
| `sys.kernel` | Kernel version |
| `sys.cpu` | CPU detail |
| `sys.memory` | Memory |
| `sys.processes` | Top processes |
| `sys.logins` | Logins |

### Storage

| Key | Shows |
|---|---|
| `stor.lsblk` | Block devices |
| `stor.df` | Filesystem usage |
| `stor.mounts` | Mounts |
| `stor.smart` | SMART health |
| `stor.lvm` | LVM volumes and groups |
| `stor.zfs` | ZFS pools |
| `stor.mdstat` | Software RAID |
| `stor.iscsi` | iSCSI sessions |

`stor.smart` is the one to run when a disk is suspected. `stor.df` catches the classic cause of
a host behaving strangely: a full filesystem.

### Network

| Key | Shows |
|---|---|
| `net.addr` | Addresses |
| `net.route` | Routes |
| `net.linkstats` | Link statistics |
| `net.bonds` | Bonds |
| `net.arp` | Neighbours |
| `net.ovs` | Virtual switch state |
| `net.ovnsb` | Chassis registration |
| `net.geneve` | Overlay tunnels |
| `net.bfd` | BFD sessions |
| `net.sockets` | Listening sockets |

`net.geneve` is the first thing to check on an overlay problem — **silence on an overlay fabric
means it is broken**. `net.bonds` tells you whether a bond actually has both members up, which
is not the same as the uplink reporting ready.

### Virtualization

| Key | Shows |
|---|---|
| `virt.domains` | Domains on this host |
| `virt.nodeinfo` | Hypervisor node info |
| `virt.versions` | Component versions |
| `virt.capabilities` | What the host can do |
| `virt.iommu` | IOMMU status |

`virt.iommu` is the one to check when PCI passthrough will not work.

### Services and logs

| Key | Shows |
|---|---|
| `svc.agent` | Agent status |
| `svc.libvirtd` | Hypervisor daemon status |
| `svc.failed` | Failed units |
| `svc.journal` | Agent journal |
| `log.agent`, `log.libvirtd`, `log.ovn`, `log.frr`, `log.openvswitch`, `log.iscsi` | Component logs |

`svc.failed` is a good first command on a host that is misbehaving without an obvious cause.

## Packet capture

Separately from the catalogue, a host's **Packet capture** tab captures traffic on an interface
and lets you download it — for when the question is what is actually on the wire.

## Logging diagnostics

The control plane also exposes its own logging state, so support can see what it has been
recording without needing shell access to a controller.

## When diagnostics are not enough

If you need something the catalogue does not cover, that is what
[root shell access](/cli/root-shell/) is for — an explicit, time-limited, audited escalation
rather than a widened diagnostic endpoint.
