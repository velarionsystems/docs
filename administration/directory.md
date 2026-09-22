---
title: "Directory integration"
description: "Authenticating console users against LDAP or RADIUS."
status: published
---

Rather than creating every account by hand, the deployment can authenticate users against a
directory you already run.

Two provider types are supported:

| Type | Use |
|---|---|
| **LDAP** | Active Directory, OpenLDAP, and anything speaking LDAP |
| **RADIUS** | An existing RADIUS infrastructure, often fronting something else |

## How it appears to users

Each provider has a display name, which is what the login page calls that domain. A user picks
their domain and signs in with their directory credentials.

Accounts authenticated this way are marked **federated**: their passwords live in the directory
and cannot be changed here.

## Adding a provider

**Settings → Users & Access → Authentication**. Configure the provider, then **verify** it
before saving.

Verification is not a formality — it is the difference between discovering a wrong bind DN now
and discovering it when someone cannot sign in.

## Roles still live here

The directory answers *who you are*. The deployment decides *what you may do*.

Directory groups do not automatically become roles. Assign roles and domains to federated users
the same way as local ones — see [Users and roles](/administration/users-and-roles/).

This is more explicit than mapping groups to roles, and the trade is deliberate: a group
membership changed by someone who does not know it grants infrastructure administration is a bad
surprise.

## Keep a local account

> **Warning** — Keep at least one **local** `SUPER_ADMIN` account with a known password, and
> two-factor on it.
>
> If the directory is unreachable — a network partition, an expired service account, a directory
> outage — federated sign-in fails. Without a local account you are locked out of the console at
> the moment you most need it.

The same reasoning applies to host CLI accounts: they are stored on each host precisely so that
they work when the control plane does not. See [The VS-HCI shell](/cli/).

## Two-factor

Where the directory or the RADIUS infrastructure already enforces multi-factor, that happens
during directory authentication and the deployment sees the result.

For local accounts, the deployment has [its own TOTP](/administration/two-factor/).

## Troubleshooting

| Symptom | Check |
|---|---|
| Nobody can sign in to a domain | Verify the provider; check the service account |
| A user signs in but can do nothing | No role assigned — roles are not inherited from the directory |
| Sign-in slow, then fails | Directory reachability and timeouts |
| Works for some users, not others | Search base, and whether those users are inside it |

Failed sign-ins appear in the [audit log](/administration/audit-log/), which distinguishes a
rejected credential from a provider that could not be reached.
