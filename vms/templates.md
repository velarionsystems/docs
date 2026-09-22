---
title: "Templates and images"
description: "Built-in templates, the marketplace, and creating your own."
status: published
---

A template is a saved set of VM properties — CPU, memory, disk size, OS type and variant — so
that creating a standard guest is one choice rather than twenty.

## Built-in templates

Every deployment ships with templates for common guests:

| Template | OS variant | vCPUs | Memory | Disk |
|---|---|---|---|---|
| Ubuntu 22.04 LTS | `ubuntu22.04` | 2 | 2048 MB | 20 GB |
| Ubuntu 24.04 LTS | `ubuntu24.04` | 2 | 2048 MB | 20 GB |
| Debian 12 | `debian12` | 2 | 2048 MB | 20 GB |
| CentOS Stream 9 | `centos-stream9` | 2 | 2048 MB | 20 GB |
| Windows Server 2022 | `win2k22` | 4 | 8192 MB | 60 GB |
| Windows 11 | `win11` | 4 | 8192 MB | 60 GB |

These are **shapes, not installed systems**. Creating from one gives you a correctly configured
empty VM; you still install the guest from an ISO.

## Why the OS variant matters

The OS type and variant are not labels. They decide what virtual hardware the guest is given —
which devices, which clock source, which defaults the hypervisor picks. A Windows guest built
with a Linux variant works badly in ways that are hard to attribute later.

Pick the variant that matches what you are actually installing.

## The marketplace

The marketplace lists templates published by Velarion that the deployment can fetch, rather
than the ones already local. It is served from the software repository your deployment is
pointed at.

An air-gapped site has no marketplace. Templates there are the built-in set plus whatever you
define yourself. See [Air-gapped sites](/updating/air-gapped/).

## Creating your own

**VMs → Templates → Create**. Give it a name, description, OS type and variant, and the default
CPU, memory and disk size.

Define templates for the shapes your organisation actually uses — "standard application server",
"database node" — so that the decision about how big things are is made once, in one place,
rather than by whoever is creating the VM that afternoon.

## Guest tools

The platform can mount a guest tools ISO into a running VM from its console screen, and eject
it again. Install the tools inside the guest for:

- clean shutdown when the platform asks for one, rather than the equivalent of holding the
  power button,
- the guest reporting its IP addresses and filesystem state back,
- quiesced snapshots — see [Snapshots](/vms/snapshots/).

> **Note** — Without guest tools, a "graceful shutdown" depends on the guest responding to an
> ACPI power button event. Most Linux guests do. A Windows guest sitting at a login screen may
> not, and the platform will eventually force-stop it.

## Operating system images

Distinct from templates: images are installable media and disk images the deployment keeps, and
they are synced from the software repository. Templates say what shape a VM is; images and ISOs
are what you install into it.

See [The ISO library](/storage/iso-library/).
