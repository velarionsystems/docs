---
title: "Audit log"
description: "Who asked for what, from where — and how it differs from the task log."
status: published
---

The audit log records **who asked**. [Tasks](/console/tasks/) record what the platform then did.

Both matter: a task tells you a VM was deleted; the audit log tells you which account asked for
it, from which address, with which client.

**Settings → Audit Logs**.

## What is recorded

Every entry carries the action, the object it acted on, the user, the source address, the user
agent and the time.

| Group | Actions |
|---|---|
| Lifecycle | `CREATE`, `UPDATE`, `DELETE` |
| Sessions | `LOGIN`, `LOGOUT`, `LOGIN_FAILED` |
| Workloads | `START`, `STOP`, `PAUSE`, `RESUME`, `REBOOT`, `MIGRATE`, `RESIZE`, `CLONE`, `SNAPSHOT` |
| Platform | `BOOTSTRAP`, `BACKUP`, `RESTORE`, `ASSIGN`, `UNASSIGN` |
| Security | `ENABLE_2FA`, `DISABLE_2FA`, `GENERATE_API_KEY`, `REVOKE_API_KEY`, `PERMISSION_DENIED` |
| Licensing | `LICENSE_REQUEST`, `LICENSE_IMPORT`, `LICENSE_REMOVE`, `LICENSE_ASSIGN`, `LICENSE_UNASSIGN` |
| Diagnostics | `DIAGNOSTIC` |

## The entries worth watching

**`PERMISSION_DENIED`.** Someone tried to do something they are not allowed to. Occasionally a
role that is too narrow; occasionally an account being probed.

**`LOGIN_FAILED`.** A few are typos. A pattern against one account, or from one address, is
not.

**`DISABLE_2FA`.** Rare and deliberate. Every occurrence should have a reason you know about.

**`GENERATE_API_KEY`.** A new credential exists. It should correspond to an integration
somebody asked for.

**`RESTORE`.** The control-plane database was replaced. This should never be a surprise.

**`DIAGNOSTIC`.** Read-only, but it records who was looking at what — useful context during an
incident.

## What is not in it

Actions taken on the underlying operating system are **not** here. They have their own trail:
root shell requests, grants, denials, expiries and revocations are logged on the host itself and
to the system log, so a host that has lost contact with the control plane still keeps a record.

See [Root shell access](/cli/root-shell/).

Host CLI sign-ins that fall back to local authentication are likewise recorded on the host.

## Attribution

Entries name the account, and an action taken with an [API key](/administration/api-keys/) is
attributed to that key. This is the practical reason to give each person their own account and
each integration its own key: shared credentials produce a log that records what happened and
not who did it.

## Retention and export

The log is held in the control plane's database and is included in
[backups](/data-protection/backups/).

For long-term retention or correlation with other systems, export it. A deployment whose audit
history only exists on the deployment has a gap exactly where an investigation into that
deployment would need it.

## From the CLI

```
Hyperion[vsnode1]> show audit
```

## System errors are separate

**Settings → System Errors** records faults the platform found in *itself*, as opposed to
actions users took. A recurring entry there is a defect or a misconfiguration, and each carries
an ID worth quoting to support.
