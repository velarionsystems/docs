---
title: "Kubernetes clusters"
description: "Provisioning k3s clusters across hosts, and driving them from the console."
status: published
---

The platform can provision and manage lightweight Kubernetes clusters — k3s — across your
hosts.

This gives you a Kubernetes control plane without building one by hand, on the same
infrastructure as your VMs, managed from the same console.

## Creating a cluster

**Kubernetes → Create**. Name the cluster and choose the host that will run the server.
Additional hosts join as agents.

Provisioning runs as a [task](/console/tasks/); the cluster reports `CREATING` while it is being
built.

## Status

| Status | |
|---|---|
| `CREATING` | Being provisioned |
| `READY` | Usable |
| `DEGRADED` | Running with a fault |
| `ERROR` | Failed — the cluster carries the reason |

An `ERROR` cluster records what went wrong; read that before deleting and retrying.

## Using it

### From the console

Run `kubectl` commands and apply manifests directly from the cluster's screen. Useful for a
quick look, and for applying something without leaving the console.

### From your own machine

Download the **kubeconfig** and use your normal tooling. This is the right way to work with a
cluster day to day — the console's kubectl is for convenience, not a replacement for your
workflow.

> **Warning** — The kubeconfig is a full credential for the cluster. Treat it like one: it
> belongs in a secret store, not a shared drive.

## Nodes

The cluster's nodes are listed with their state. A node here is a host running part of the
cluster — the relationship to [Infrastructure](/console/infrastructure/) is direct: these are
your servers.

## Where the boundary is

The platform provisions and manages the **cluster**. What runs inside it is yours — workloads,
ingress, storage classes, operators. The console does not attempt to be a Kubernetes management
interface beyond applying manifests and running commands.

## Storage and networking

Kubernetes running on hosts uses those hosts' resources. It is not automatically integrated
with [replicated storage](/storage/replicated/) or the
[distributed switches](/networking/distributed-switches/) — if you want persistent volumes
backed by the cluster's storage, configure that inside Kubernetes as you would anywhere.

## Deleting a cluster

Deleting tears the cluster down on its hosts: the services are stopped and disabled, the
installed components are removed, and the cluster's state directory is deleted.

That last part is not optional tidiness. k3s encrypts its bootstrap data with the join token, so
a state directory left behind makes the host unusable for any future cluster.

## Kubernetes or VMs

| | |
|---|---|
| Existing Kubernetes workloads | A cluster here |
| Needs scheduling and self-healing | A cluster here |
| Traditional applications | [Virtual machines](/vms/) |
| Needs VM-level HA, live migration and DR | [Virtual machines](/vms/) |

Running Kubernetes on bare hosts gives the best performance. Running it in VMs gives you the
platform's own HA, migration and DR underneath it — a real option, and often the better one for
a small cluster you care about.
