---
title: "Adding servers"
description: "Growing a cluster: joining nodes with the token, and admitting them from Approvals."
status: published
---

Every server after the first joins an existing cluster. Joining is deliberately a two-step
process: the new server presents a token and enrols itself, then a human admits it from the
console. A token alone is not enough to get into the fleet.

## Before you start

You need the **cluster VIP** and the **join token** from the first server's setup summary.

If you no longer have the token, read it from the console under **Approvals**, or from the CLI:

```
Hyperion[vsnode1]> show cluster join-token
```

One token serves the whole cluster and covers both roles. It is short by design — six
characters — because it is typed at a server console, and it only gets a node into a queue that
a human still has to clear.

## Step 1 — Run setup on the new server

Install from the same medium and work through [first-boot setup](/getting-started/first-server/)
as before. At the deployment mode section choose **cluster**, then **join**, and decide what the
server is for:

| Join as | Runs | Use for |
|---|---|---|
| **Controller + host** | Control plane and workloads | Reaching three controllers, so the cluster tolerates a failure |
| **Host only** | Workloads | Every server after the third |

Then supply the cluster VIP and the join token. A controller also asks for a node priority —
leave it at the default **150**, which is higher (therefore less preferred) than a master at 100.

> **Warning** — Priority is inverted: **lower is more preferred**. A joining controller with a
> priority below the master's can win an election and take the virtual IP before its database
> has finished syncing.

Apply. The server configures itself, contacts the VIP, and enrols. It then waits.

## Step 2 — Admit it from the console

Open **Approvals**. Nodes that have enrolled appear in one of two queues:

- **Hypervisors (agents)** — servers that will run workloads.
- **Standby controllers** — servers that will also run the control plane.

A server joining as *controller + host* appears in **both**, and needs approving in both. One
makes it a member of the HA cluster; the other admits it as a host that can run VMs.

Approve each entry. The queues refresh on their own, and the host moves to **Infrastructure**
with status `ONLINE` once its agent connects.

From the CLI:

```
Hyperion[vsnode1]> show host pending-approval
Hyperion[vsnode1]> host approve <name>
Hyperion[vsnode1]> show cluster nodes pending
Hyperion[vsnode1]> cluster node approve <id|ip>
```

## Step 3 — Confirm the cluster is healthy

```
Hyperion[vsnode1]> show cluster status
CLUSTER  [OK]  3+ healthy controllers — quorum margin is healthy.

Enabled:             true
Cluster ID:          cluster-5c189bef
VIP:                 10.2.32.240 (mgmt-mgmt/24)
VIP alive:           true
VIP owner:           vsnode1
This node:           10.2.32.101
Term:                1
Isolated from peers: false

Members
NODE     ROLE     ADDRESS      PRIORITY  ALIVE  SYNC  SELF
vsnode1  ACTIVE   10.2.32.101  100       true   OK    true
vsnode2  STANDBY  10.2.32.102  200       true   OK    false
vsnode3  STANDBY  10.2.32.103  200       true   OK    false
```

What to check:

- Every member is `ALIVE true`.
- Every member is `SYNC OK` — a standby whose database has not caught up cannot take over.
- Exactly one member is `ACTIVE`.
- `Isolated from peers: false`.

## Promoting a standalone node

A server installed as standalone becomes a cluster when a second node is admitted. The
**Approvals** queue is hidden in standalone mode — with nothing to approve it has no purpose —
but it reappears the moment something is waiting in it, so the second node is never stranded.

Once that node is admitted the deployment reports itself as a cluster, and the features that
need peers — controller failover, live migration, replicated storage — become available.

> **Note** — Going from one controller to two gives you a *warm spare*, not a quorum. Two
> controllers cannot form a majority when they disagree. Three is the first count that
> tolerates a failure.

## Additional join tokens

The cluster token created at setup is not the only one. **Approvals** can issue further host
join tokens, each of which can be revoked independently — useful when a rollout is run by
someone who should not hold the original.

## Removing a server

Put the host into maintenance first so its workloads move off:

```
Hyperion[vsnode1]> host maintenance <name> on
```

Then remove it from **Infrastructure**. A controller must also be removed from the HA cluster.
Never simply power a node off and leave it — the cluster keeps expecting it, and a controller
that is absent rather than removed still counts against quorum.
