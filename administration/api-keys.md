---
title: "API keys"
description: "Credentials for automation — scoped, expiring and revocable."
status: published
---

An API key authenticates automation against the [REST API](/reference/api/) without embedding a
person's password in a script.

## Creating one

**Profile → API keys → Create**. Give it a name describing what will use it, optional
permissions, and an expiry.

The key is shown **once**, at creation. The deployment stores only a hash and a short prefix, so
it cannot show it to you again — and cannot leak it if the database is read.

If you lose it, revoke it and create another.

## What a key carries

| Field | Notes |
|---|---|
| Name | What uses it. Be specific |
| Prefix | A few visible characters, for identifying it in the list |
| Permissions | Optional scoping |
| Expiry | Optional — but set one |
| Last used | When it was last seen |
| Enabled | Keys can be disabled without deletion |

## Using a key

Present it as a bearer credential on API requests. See [REST API](/reference/api/).

A key acts with the permissions it was given, bounded by the permissions of the user who owns
it. A key created by a `VIEWER` cannot be given the ability to delete a VM.

## Expiry

Set one. A key with no expiry is a credential that outlives the integration it was made for,
the person who made it, and usually the documentation that explained what it was.

A year is a reasonable default for infrastructure automation; shorter for anything touching a
CI system, where rotation is cheap.

## Last used

The list shows when each key was last seen. Two things are worth acting on:

- A key **never used** — the integration is not working, or was never finished.
- A key not used **for months** — the integration is gone and the key should be too.

## Revoking

Revoke immediately when:

- a key may have been exposed — a repository, a log, a screenshot,
- the integration is retired,
- the person who created it has left.

Revocation is immediate. Creation and revocation are both recorded in the
[audit log](/administration/audit-log/).

## Good practice

- **One key per integration.** A shared key cannot be revoked without breaking everything using
  it, and the audit log cannot tell you which system acted.
- **Name it for its consumer** — "backup-verification", not "key2".
- **Least privilege.** Create keys under an account with the rights the job needs, and no more.
- **Store them properly.** A secret manager, not a repository and not an environment file
  committed by accident.
- **Rotate.** Create the replacement, move the integration, then revoke the old one — in that
  order, so nothing breaks in between.

## Keys and audit

Actions taken with a key are attributed to it in the audit log, so you can tell automation from
a person. That is most of why automation should not use a human account: an audit trail that
cannot distinguish a nightly job from an administrator at a keyboard is much less useful during
an investigation.
