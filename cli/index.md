---
title: "The VS-HCI shell"
description: "What SSH into a node gives you, how it authenticates, and why it keeps working when the console does not."
status: published
---

SSH into a VS-HCI node and you land in the platform's own shell, not a Linux prompt.

```
$ ssh vsadmin@10.2.32.101

VSOS-HCI 4.0.1  vsnode1 x86_64

  Hyperion Enterprise Hypervisor
  Authorized access only. All activity is logged.

Logged in as vsadmin (SUPER_ADMIN).
Authenticated on this host (hostlocaldb/hostglobaldb).
Scope: host vsnode1 — 'scope orchestrator' for cluster-wide commands.
'?' or 'help' lists commands. 'exit' logs out.

Hyperion[vsnode1]>
```

> **Note** — The banner and prompt show `Hyperion`, the platform's internal engineering name.
> The product is VS-HCI; the two refer to the same thing. Transcripts in this documentation show
> what you will actually see.

## Why a shell of its own

The shell is not a restricted Bash. There is **no command dispatch to a shell at all** — `bash`,
`sudo`, backticks and `$(...)` are simply words that are not in the command tree, and produce
"unknown command" the same way any other unknown word does.

That is a stronger guarantee than a blocklist, because there is no allowlist to get wrong.

Reaching the operating system underneath is a separate, deliberate, audited act. See
[Root shell access](/cli/root-shell/).

## Authentication

Signing in takes two things.

**The Linux account** proves you reached the box. **Your VS-HCI account** decides what you may
do — the shell holds a session for it, and every command is authorized server-side by the same
checks the console and API use.

So a `VIEWER` gets a clean refusal from the API rather than from client-side logic that could
drift, and the [audit log](/administration/audit-log/) names the person rather than a shared
login.

### When the control plane is unreachable

This is the part that matters most at the worst time.

If the virtual IP is down, there is nowhere to authenticate — and an operator would be locked
out of the box during exactly the outage they need to fix.

So each host keeps a **local account database**. The shell tries the control plane first; only
when it is genuinely unreachable does it fall back to authenticating against that database. The
role it grants is one of the same roles the control plane uses, so permissions behave
identically online and offline.

The banner tells you which happened. `Authenticated on this host` means the local database
answered.

These accounts are managed centrally — a host's **Users** tab in
[Infrastructure](/console/infrastructure/) — and pushed to each host, so a credential accepted
online is accepted offline.

While offline, cluster and API commands fail; there is no control plane for them to talk to. The
point of the local login is to get you in, and to
[root shell access](/cli/root-shell/), so the box can be recovered.

## Scope

Many operations are per-host, so the shell tracks what you are acting on. The prompt carries it:
`Hyperion[vsnode1]>` means commands act on `vsnode1`.

```
scope orchestrator      act on the cluster
scope host <name>       act on one host
show scope              what am I acting on
```

A node running the hypervisor starts in **host scope, on itself** — someone who has SSHed into a
hypervisor is almost always there about that hypervisor. A controller-only node starts in
**orchestrator scope**.

## Getting around

`?` or `help` at any depth lists what may follow. Tab completes, including live VM and host
names.

```
Hyperion[vsnode1]> help
  show       Display operational state
  scope      Change what commands act on
  vm         Act on a virtual machine
  host       Act on a host
  cluster    Act on the HA cluster
  task       Act on a task
  alarm      Act on alerts
  storage    Act on storage
  run        Run a read-only operation
  request    Request a system operation
  rootshell  Request authorized access to the Linux shell
```

See [Command reference](/cli/commands/).

## The browser terminal

Each host also has a terminal on its detail screen in the console. It opens the same shell — not
a Linux prompt — and sessions are audited when they open and close.

## What SSH will not do

`ssh host <command>` is **refused**. So are `scp` and `rsync`.

SSH runs a non-interactive command by invoking the login shell with it; honouring that would
hand every SSH user arbitrary execution on the node and make the rest of the design
decorative. The shell must be driven interactively.

## Accounts and the sudo group

Operator accounts are removed from the `sudo` group when a node is put into appliance mode.

> **Warning** — An account left in `sudo` can run `sudo bash` and never sees an authorization
> challenge, which makes the whole mechanism ornamental while everything still *looks* correctly
> configured. If you add accounts to a host by hand, check their group membership.

The platform's own service identity on each host deliberately keeps a normal shell — the control
plane reaches hosts as that identity — and is protected by having no usable password instead.
It is not an account for people, and setup refuses to let a human take that name.
