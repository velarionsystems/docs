---
title: "Approvals"
description: "Admitting nodes that have enrolled, and managing the tokens that let them enrol."
status: published
---

A server that runs first-boot setup with a valid join token does not join the fleet. It
*enrols* — it registers itself and then waits. **Approvals** is where a human lets it in.

The split is deliberate. A token is typed at a server console and is short enough to be read
off a screen, so it is not a credential you would want to be sufficient on its own. It buys a
place in a queue; admission is a separate, audited decision.

## The two queues

### Hypervisors (agents)

Servers that will run workloads. Approving one admits it as a host: the control plane starts
sending it commands, provisions its key, and it appears in
[Infrastructure](/console/infrastructure/) as `ONLINE` once its agent connects.

### Standby controllers

Servers that will also run the control plane. Approving one admits it to the HA cluster, after
which it replicates the database and becomes eligible to take the virtual IP.

A server installed as **controller + host** appears in **both** queues and must be approved in
both. Approving only the controller entry gives you a control-plane peer that cannot run VMs;
approving only the host entry gives you a hypervisor that is not part of the HA cluster.

Both queues refresh by themselves every ten seconds.

## From the CLI

```
Hyperion[vsnode1]> show host pending-approval
Hyperion[vsnode1]> host approve <name>

Hyperion[vsnode1]> show cluster nodes pending
Hyperion[vsnode1]> cluster node approve <id|ip>
```

## Join tokens

The screen also manages the tokens themselves.

The cluster token generated during first-boot setup serves both roles for the whole cluster.
You can see it here, and from the CLI:

```
Hyperion[vsnode1]> show cluster join-token
```

Additional **host join tokens** can be issued and revoked independently. This is worth doing
when someone else is racking servers: give them a token scoped to that rollout, and revoke it
when they are finished, without disturbing the original.

> **Note** — Revoking a token does not affect nodes that already enrolled with it. It only
> stops further enrolments.

## What to check before approving

The queue shows each node's name and address, and for a controller its priority. Before you
approve:

- **Is this a node you installed?** An entry you do not recognise means someone has a valid
  token. Revoke it.
- **Is the address right?** A controller's address is what its peers will use.
- **Is the priority right?** A joining controller should have a *higher* number than the
  running master — lower is more preferred. A joiner that outranks the master can take the
  virtual IP before its database has synced.

Priority can be changed later, but before admission is the cheapest time.

## On a single node

The Approvals screen is hidden on a standalone deployment — there is nothing to approve. It
reappears as soon as something is actually waiting in it, so the second node you add is never
stranded with no way to be admitted.
