---
title: "Two-factor authentication"
description: "TOTP enrolment for console accounts and host CLI accounts."
status: published
---

The deployment supports time-based one-time passwords (TOTP) — the six-digit codes produced by
an authenticator app.

## Enrolling

**Profile → Two-factor authentication → Set up**.

1. The console shows a QR code. Scan it with your authenticator app.
2. Enter a code from the app to confirm the clock and the secret agree.
3. Two-factor is enabled on your account.

Step 2 is what stops you locking yourself out with a secret that was never actually stored
anywhere.

## Signing in

After your password, you are asked for a code.

## Disabling

From the same screen, with a current code. You cannot disable someone else's two-factor from
your own profile — an administrator does that from the user's account, and it is recorded in the
[audit log](/administration/audit-log/).

## Where it applies

TOTP protects **console and API sign-in**. It also applies to the
[VS-HCI shell](/cli/) on a host, where an account with TOTP enrolled is asked for a code after
its password — including when the host is authenticating locally because the control plane is
unreachable.

The parameters are the same in both cases, so a code accepted online is accepted offline.

## Clock skew

TOTP depends on both ends agreeing on the time. If codes are rejected, check the clock on the
device running the authenticator.

The servers get their time from the NTP server configured during setup, which is one of several
reasons that setting matters. See [Requirements](/getting-started/requirements/).

## Recovery

> **Warning** — Losing the enrolled device locks you out of that account. Another
> `SUPER_ADMIN` can clear two-factor on it.
>
> Keep **more than one** administrator with two-factor enrolled on different devices. A
> deployment with one administrator and one phone is one lost phone away from a support call.

## What to enrol

At minimum, every account that can change things: `SUPER_ADMIN`, `ADMIN` and `OPERATOR`.

`VIEWER` accounts are lower risk but still see the whole deployment, which is reconnaissance if
nothing else.

Automation should use [API keys](/administration/api-keys/) rather than an account with TOTP —
a key is scoped, expiring and revocable, which is a better fit than a shared secret in a script.
