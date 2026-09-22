---
title: "Containers"
description: "Running containers directly on a host, alongside virtual machines."
status: published
---

Hosts can run containers as well as virtual machines, managed from the same console.

Containers here run **on a host**, not inside a VM. They share the host's kernel and are
scheduled by you onto a specific host — this is not an orchestrator. For that, see
[Kubernetes clusters](/workloads/kubernetes/).

## When to use them

| | Use |
|---|---|
| Needs its own kernel, or full isolation | A [virtual machine](/vms/) |
| Packaged as an image, stateless, host-pinned is fine | A container |
| Needs scheduling, scaling, self-healing | [Kubernetes](/workloads/kubernetes/) |

Containers are a good fit for utilities that sit beside the infrastructure — an internal
registry, a collector, a small internal service. They are a poor fit for anything that needs to
survive its host failing, because nothing restarts them elsewhere.

## Images

Pull images before creating containers from them. The image list shows what each host holds.

Images are per-host: pulled on the host that will run the container.

## Creating a container

Choose the host, the image, and the runtime settings — the command, environment variables, port
mappings, volumes and network.

The console also lists the volumes and networks available on each host, so a container can be
attached to what already exists there.

## Running them

| Action | |
|---|---|
| Start, stop, restart | Lifecycle |
| Logs | The container's output |
| Stats | Live CPU, memory and I/O |
| Exec | Run a command inside it |
| Delete | Remove it |

Logs and exec are the two you will use most — the container equivalent of opening a VM's
console.

## What containers do not get

> **Warning** — Containers are **not** covered by workload high availability. A container on a
> host that fails does not restart elsewhere. They do not live-migrate, and they are not
> replicated by [disaster recovery](/data-protection/disaster-recovery/).
>
> Anything that must survive a host failure belongs in an HA-enabled VM or on a
> [Kubernetes cluster](/workloads/kubernetes/).

## Storage

Container data lives on the host unless you attach it to something shared. A container with
important state on a host's local volume has that state in one place, on one server.

## Networking

Containers attach to the host's container networking, which is separate from the
[distributed switches](/networking/distributed-switches/) VMs use. A container is on its host's
network, not on a fabric that follows it.

## Runtime status

The console reports whether the container runtime is available on each host. A host where it is
not will not run containers, and that is the first thing to check when creation fails.

## Containers or a VM

In a hyperconverged deployment, most things that look like container workloads are better off
in a small VM: the VM gets HA, live migration, snapshots, DR and a network that follows it.

Use containers directly on a host when you want the packaging and do not need any of that.
