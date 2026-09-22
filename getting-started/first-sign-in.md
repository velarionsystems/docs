---
title: "First sign-in"
description: "Reaching the console for the first time, and the handful of things to do straight away."
status: published
---

## Reaching the console

Browse to the cluster's virtual IP over HTTPS, or to the DNS name you gave as the console name
during setup:

```
https://10.2.32.240/
https://vs.example.com/
```

Always use the **virtual IP or console name**, not an individual server's address. The VIP
follows whichever controller is active; a node address stops working the moment that node does.

> **Note** — Writes must go to the active controller. If you reach a standby directly it
> answers with HTTP 409 and the address of the active node rather than accepting a change it
> cannot commit.

### The certificate warning

The cluster generates its own certificate at first boot, so your browser does not recognise the
issuer. That is expected on a new deployment. Accept it to get in, then replace the certificate
with one from your own authority — see [Certificates](/administration/certificates/).

If you gave a console name during setup, the certificate names it. If you left it blank, the
certificate names the VIP only, and browsing by any other name will warn even after you install
a trusted certificate.

## Signing in

Use the administrator account from first-boot setup — the same username and password you set for
SSH.

The console forces a password change on first sign-in. That is not a suggestion you can skip;
the account is flagged until the password is changed.

> **Warning** — Deployments also carry a built-in `admin` account. Setup locks it, because its
> password is identical on every installation. If your deployment still has `admin` enabled,
> treat that as the first thing to fix: change its password and disable it.

## The first five things to do

### 1. Replace the certificate

Until you do, every operator learns to click through a browser warning, which is exactly the
habit you do not want. **Settings → Certificates** generates a CSR for your own CA.
See [Certificates](/administration/certificates/).

### 2. Create real accounts

Do not run day to day as the account setup created. Give each operator their own login, so the
audit log names a person. Roles are per-domain — someone can administer storage without being
able to touch networking. See [Users and roles](/administration/users-and-roles/).

If you have a directory, connect it instead of creating accounts by hand —
see [Directory integration](/administration/directory/).

### 3. Turn on two-factor authentication

At minimum for accounts that can change things. See
[Two-factor authentication](/administration/two-factor/).

### 4. Set up alerting

A cluster with no alert rules is a cluster you find out about from a user. Start with host
reachability, disk capacity and storage health. See [Alerts](/monitoring/alerts/).

### 5. Configure backups

The control plane's database holds every definition in the deployment. Point it at somewhere off
the cluster and set a schedule. See [Backup and restore](/data-protection/backups/).

## Finding your way around

The console's left-hand navigation groups everything by what it acts on:

| Section | What lives there |
|---|---|
| **Dashboard** | Fleet health at a glance |
| **Infrastructure** | Servers, availability zones, and each host's detail |
| **Approvals** | Nodes waiting to be admitted |
| **VMs** | Virtual machines |
| **Containers**, **Kubernetes** | Other workload types |
| **Networks** | Uplinks, distributed switches, fabrics |
| **Storage** | Pools, volumes, replicated storage, exports |
| **Tasks** | Everything asynchronous the platform is doing |
| **Alerts** | Rules and the events they fired |
| **Backups** | Control-plane backup and restore |
| **Software** | Versions and upgrades |
| **Settings** | Certificates, licensing, authentication, and the rest |

See [A tour of the console](/console/) for what each screen is for.

## If the console does not come up

Sign in over SSH to any node and ask the platform directly:

```
Hyperion[vsnode1]> show cluster status
Hyperion[vsnode1]> show system status
```

`show cluster status` tells you whether a controller is active and which node owns the VIP.
The CLI authenticates against the host itself when the control plane is unreachable, so it keeps
working during exactly the outage you need it for. See [The VS-HCI shell](/cli/) and
[Troubleshooting](/support/troubleshooting/).
