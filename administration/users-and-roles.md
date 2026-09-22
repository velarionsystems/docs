---
title: "Users and roles"
description: "Accounts, the five roles, and the domains that let you scope them."
status: published
---

Console accounts live in the deployment's own database. They are separate from the Linux
accounts that can sign in to a host's CLI — see [The VS-HCI shell](/cli/).

## Roles

| Role | Can |
|---|---|
| `SUPER_ADMIN` | Everything, including root shell access and user management |
| `ADMIN` | Administer the deployment |
| `OPERATOR` | Run workloads — create, start, stop, migrate |
| `VIEWER` | Read only |
| `API` | For automation rather than people |

## Domains

Roles are not only global. A role can be scoped to a **domain**, so someone can administer one
part of the deployment without touching the others.

| Domain | Covers |
|---|---|
| `VIRTUALIZATION` | VMs, containers, Kubernetes, templates, consoles, attached media |
| `NETWORKING` | Fabrics, uplinks, distributed switches, per-host network configuration |
| `STORAGE` | Pools, storage clusters, replicated storage, exports, per-host storage |
| `ORCHESTRATOR` | The control plane itself — cluster, upgrades, licensing, backups, users, tasks, alerts, audit, dashboard |
| `HOSTS` | The servers |

A storage administrator gets `ADMIN` on `STORAGE` and `VIEWER` elsewhere. A network engineer
gets `ADMIN` on `NETWORKING`. Neither can change the other's area, and both can see enough to
diagnose across the boundary.

This is worth setting up properly. The alternative — everybody a `SUPER_ADMIN` — makes the
[audit log](/administration/audit-log/) much less useful, because every entry becomes "someone
who could have done anything did this one".

## Permissions are enforced server-side

Every endpoint carries its own authorization check. A user without rights gets a clean refusal
**from the API**, not from the interface deciding to hide a button.

That matters because the console, the [CLI](/cli/) and direct API calls all hit the same checks.
There is no route that skips them, and client-side logic cannot drift out of step with what is
actually permitted.

A refusal is recorded in the audit log.

## Creating a user

**Settings → Users & Access**. Give a username, email, full name, role, and any per-domain
roles.

New accounts should be created with **must change password** set, so the person who receives the
credentials is the only one who ends up knowing the password.

## Account lockout

After **5 consecutive failed sign-ins**, an account is locked for **15 minutes**. A successful
sign-in resets the counter.

This is deliberately a lockout with a timer rather than one an administrator must clear — the
latter turns a mistyped password into a support ticket, and people respond to that by choosing
weaker passwords.

Failed sign-ins are recorded in the audit log. A pattern of them against one account, or from
one address, is worth looking at.

## Sessions

Sessions have an idle timeout, set per user. An operator console left open on a shared screen is
a real exposure; the timeout is the cheap mitigation.

## The default account

Deployments carry a built-in `admin` account whose initial password is identical on every
installation. First-boot setup creates your own administrator account and locks it.

> **Warning** — If your deployment still has `admin` enabled, change its password and disable
> it. A published default credential is a published default credential.

## Federated accounts

Accounts from a directory are marked as federated. Their passwords live in the directory, not
here — you manage membership there and roles here. See
[Directory integration](/administration/directory/).

## Host CLI accounts

Separately from console accounts, each host has accounts that can sign in to **its** CLI. They
are managed centrally under a host's **Users** tab and pushed to the host, so they keep working
when the control plane does not — which is precisely when you need them.

See [The VS-HCI shell](/cli/).

## Good practice

- One account per person. Shared accounts destroy the audit trail.
- Scope by domain rather than handing out `SUPER_ADMIN`.
- [Two-factor](/administration/two-factor/) on anything that can change things.
- [API keys](/administration/api-keys/) for automation, not a person's password.
- Review the list periodically. Accounts outlive the people who needed them.
