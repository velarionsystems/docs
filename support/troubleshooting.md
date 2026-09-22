---
title: "Troubleshooting"
description: "Where to look first, by symptom."
status: published
---

## Start here

Four places, in this order:

1. **[Tasks](/console/tasks/)** — a failed task usually *is* the explanation.
2. **Settings → System Errors** — faults the platform found in itself.
3. **[Diagnostics](/monitoring/diagnostics/)** on the host involved — read-only, safe.
4. **[The CLI](/cli/)** — works when the console does not.

```
Hyperion[vsnode1]> show system status
Hyperion[vsnode1]> show cluster status
Hyperion[vsnode1]> show task
```

## The console is unreachable

Sign in over SSH to any node.

```
Hyperion[vsnode1]> show cluster status
```

| Finding | Meaning |
|---|---|
| A node is `ACTIVE`, `VIP alive: true` | The control plane is up — the problem is between you and the VIP |
| No `ACTIVE` member | No majority. Check how many controllers are alive |
| `Isolated from peers: true` everywhere | The controllers cannot see each other — a network problem |
| An `ACTIVE` node but `VIP alive: false` | The address is not up on the interface |

Two controllers cannot elect an active node between them. If one of three is down you are fine;
if two are down, you are not. See [High availability](/high-availability/).

The CLI authenticates against the host itself when the control plane is unreachable, so it keeps
working during exactly this outage.

## A host is offline

1. Is it powered and on the network?
2. `run diagnostic svc.agent` — is the agent running?
3. `run diagnostic svc.failed` — any failed services?
4. `run diagnostic stor.df` — a full filesystem makes a host behave strangely.

A host that is `ONLINE` but not accepting work is usually in **maintenance mode**.

## A VM will not start

| Check | |
|---|---|
| Its state | `PENDING` means the platform declined, and carries a reason |
| Its host | Offline hosts cannot start anything |
| Hardware virtualization | `kvmEnabled: false` — enable it in firmware |
| Storage | Can the host reach the disk's backing? |
| Capacity | The zone may have no room |
| Tasks | The start task carries the failure |

`PENDING` is not `STOPPED`. Nothing restarts a `PENDING` VM automatically.

## A VM will not migrate

Almost always storage. A VM with any disk on a host-local pool cannot move until that disk
moves to shared backing.

Otherwise: a PCI passthrough device pins a VM permanently; the target may lack capacity; the
target must be `ONLINE`. The migration screen states which of the four situations the VM is in.
See [Live migration](/vms/live-migration/).

## Networking

| Symptom | Check |
|---|---|
| A new network passes nothing | Is the VLAN trunked to the uplink's switch ports? |
| Large packets disappear | Run the uplink's **MTU probe** — this is what it is for |
| Overlay fabric silent | `run diagnostic net.geneve` — silence means broken |
| Guest has no address | DHCP blocked; check the security ladder is intact |
| Guest reaches its own network only | Overlay default drop with no matching allow rule |
| Connections hang rather than refuse | Established replies not permitted |
| Uplink stuck `PENDING` | Plumbed but unprobed — run the probe |
| Bond not redundant | `run diagnostic net.bonds` — are both members up? |

A jumbo-frame MTU that the physical switches do not carry is the single most common networking
problem, and it looks like everything except an MTU problem.

## Storage

| Symptom | Check |
|---|---|
| Fleet-wide slowness | Is a rebuild or rebalance running? |
| Pool `DEGRADED` | A failed disk — `run diagnostic stor.smart` |
| Writes failing, guests confused | A thin pool that filled |
| VM cannot see its iSCSI disk | Does the host have a session to the target? |
| Sessions gone after reboot | Check session persistence |

Watch **allocated**, not used: thin provisioning lets guests believe in space that is not there.

## Storage cluster will not build

Run **preflight**. It exists to fail at onboarding rather than during a failover, and its
message says what is unfit.

For a replicated pair, the usual answer is **fencing**. No fencing, no cluster — and that is not
a check to work around.

## Workloads did not restart after a host failed

Check the **fence**. If fencing failed, the pipeline stops deliberately and leaves the VMs
down: a host that is unreachable but not provably stopped may still be writing, and two writers
destroy the disk.

Then check: was HA enabled on those VMs? Was there capacity? Are they `PENDING` with a reason?

## Certificates

| Symptom | Cause |
|---|---|
| Warning on a name that works elsewhere | That name is not on the certificate |
| Some clients trust it, others do not | The intermediate chain was not installed |
| Everything broke at once | The certificate expired |

## Upgrades

A node at `FAILED` stops the pipeline. Read the failure, fix the node, retry that node. Do not
leave a fleet half-upgraded.

`SYNCING` is a controller catching up; it is not ready until it leaves that state.

## Gathering information for support

1. The **error ID** from a system error, or the failing **task ID**.
2. `show cluster status` and `show system status`.
3. Relevant [diagnostics](/monitoring/diagnostics/) from the affected host.
4. What changed, and when.
5. Your **installation identity** from Settings → Licensing.

## When you need the operating system

Use [diagnostics](/monitoring/diagnostics/) first — most questions are answered by a read-only
check. When they are not, [root shell access](/cli/root-shell/) is the supported route, and
needs a key holder to sign the challenge.
