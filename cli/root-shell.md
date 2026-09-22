---
title: "Root shell access"
description: "Getting to the operating system underneath — a signed, time-limited, audited escalation."
status: published
---

The VS-HCI shell does not reach the operating system. When you genuinely need it — a hardware
problem, a recovery, something the [diagnostics](/monitoring/diagnostics/) do not cover —
`rootshell` is the way, and it requires an authorization you cannot produce on the appliance
itself.

## How it works

```
Hyperion[vsnode1]> rootshell

Challenge:

  HYP1.eyJ2IjoxLCJpZCI6...

  Fingerprint:  A1B2-9C3F-118E-44DA
  Host:         vsnode1 (a1b2c3...)
  Requested by: vsadmin / vs:admin
  Expires:      2026-09-22T07:05:00Z

Enter signed response:
```

The node prints a **challenge**. The holder of your organisation's signing key signs it — **on
their own machine, never on the appliance** — and returns a response token you paste back.

```bash
hyperion-sign 'HYP1.eyJ2...' --key vshci-rootshell-signing-key.pem
```

The signing tool decodes and displays the challenge before signing, so the key holder can read
what they are authorizing. It returns a `HYPSIG1.` token.

Paste it, and you are root for a bounded session:

```
Authorization verified.

Entering Linux root shell...

  ── elevated session ──────────────────────────────────────────────
   You are root on the underlying operating system. This session is logged.

     rootexit           return to the VS-HCI CLI
     hyshell <command>  run a VS-HCI CLI command from here

   This session ends automatically when its authorization expires.
  ──────────────────────────────────────────────────────────────────

  Authorization expires in 60 minutes (session odkqKq6y87okDt_6Gw2qTg).

root@vsnode1:/home/vsadmin#
```

## What is verified

| Check | Why |
|---|---|
| Signature against the key the build trusts | Only the offline key grants root |
| Challenge not expired — **10 minutes** | A shoulder-surfed challenge goes stale |
| Nonce not already used | Single use; survives a reboot |
| Bound to this machine | A challenge from one node is inert on another |
| Bound to the requesting user | Recorded for audit |

The signature covers the **whole** token, so a signature cannot be moved to a challenge naming a
different host.

## Where the verification happens

Verification runs **as root, in a privileged helper** — not in the shell you are typing into.

The shell runs unprivileged, so anything it decided would be advisory: a user who can run the
shell can invoke the helper directly. The helper therefore trusts nothing from its caller — it
mints its own challenge, reads the response from its own input, and verifies against its own
trusted key. Your real identity comes from something the caller cannot forge.

This is the same reason [diagnostics](/monitoring/diagnostics/) are a fixed catalogue
re-validated on the host: a check the caller could reach is not a check.

## Who may ask

`rootshell` requires the **SUPER_ADMIN** role.

`rootshell status` and `rootshell revoke` do not. Seeing and killing an elevated session is not
an escalation, and gating them would stop an operator shutting one down in a hurry.

That role check is **policy, not a boundary** — it is enforced in the shell rather than in the
helper on purpose, because the helper would have to ask the control plane who you are, and root
shell access must work when the control plane is down. A caller who skips the check still gets
nowhere without a signature.

## Sessions

```
Hyperion[vsnode1]> rootshell status
Hyperion[vsnode1]> rootshell revoke <session>
```

Sessions expire after **60 minutes** by default. That is a hard stop, not an idle timeout — a
session that is still open when its authorization runs out is killed.

Inside the session, `rootexit` returns to the VS-HCI shell and `hyshell <command>` runs a
platform command without leaving the Linux prompt.

## Audit

Every request, grant, denial, expiry and revocation is written **to the host** and to the system
log — not only to the control plane. A node that has lost contact with the control plane still
keeps a complete trail, which is important because that is exactly when root access gets used.

## Key management

There is one signing key for the fleet, held by whoever your organisation designates. Its public
half is compiled into the platform build.

Replacing the trust anchor requires a new build and a redeploy — deliberately, so that an
attacker who can rewrite a configuration file cannot install their own.

> **Warning** — **Elevated access to any node still running an older build requires the older
> key.** Roll the new build out across the fleet before retiring the old key, or you will lock
> yourself out of whatever you did not upgrade.

## Operating this well

- **Keep the key offline**, held by a small number of people, and never on an appliance.
- **Read the challenge before signing.** It names the host and the requester; that is what the
  key holder is confirming.
- **Have more than one key holder.** A single holder on leave is an outage you cannot fix.
- **Use [diagnostics](/monitoring/diagnostics/) first.** Most questions are answered by a
  read-only check that needs no escalation at all.
- **Expect it to be rare.** Frequent root shell use usually means something is missing from the
  platform — tell support rather than working around it.

## What the appliance closes off

Root shell access is only meaningful if the other routes to a shell are closed. They are:

| Route | Closed by |
|---|---|
| `ssh host <command>`, `scp`, `rsync` | Non-interactive invocation is refused outright |
| `sudo bash` | Operator accounts are removed from the `sudo` group |
| Shelling out from an editor | There is no editor and no shell-out — every command is a node in the tree |
| Killing the shell to fall back to a parent | The shell replaces its own process; there is no parent |
| The browser terminal | It opens the VS-HCI shell, and is audited on open and close |
| The installer's own account | Setup converts **every** interactive `sudo` account, not just the one you named |

> **Warning** — The `sudo` group is the one that silently defeats everything else. An account
> left in it runs `sudo bash` and never sees a challenge.
