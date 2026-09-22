---
title: "REST API"
description: "Authenticating, conventions, errors, and the interactive specification."
status: published
---

Everything the console does, it does through this API. There is nothing the console can do that
the API cannot.

## The specification

The deployment serves its own OpenAPI specification, which is always accurate for the version
you are running:

```
https://<vip>/v3/api-docs        the specification
https://<vip>/swagger-ui.html    interactive browser
```

That is the exhaustive reference — every endpoint, every parameter. This page covers the things
you need to know that a specification does not tell you.

## Base URL

Always the **virtual IP** or the console name:

```
https://vs.example.com/api/...
```

> **Warning** — A standby controller answers reads but returns **HTTP 409** to a write, naming
> the active controller. Client code should either always use the VIP, or handle a 409 by
> following the address it returns. Pointing a script at a node's own address works until that
> node stops being active.

## Authentication

Two ways in.

### Sign in

```bash
curl -X POST https://vs.example.com/api/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"operator","password":"..."}'
```

```json
{
  "access_token": "...",
  "refresh_token": "...",
  "token_type": "Bearer",
  "expires_in": 86400,
  "user": { "username": "operator", "role": "SUPER_ADMIN", "domains": { ... } }
}
```

Present the access token as a bearer credential:

```bash
curl https://vs.example.com/api/vms \
  -H "Authorization: Bearer $TOKEN"
```

Access tokens last **24 hours**, refresh tokens **7 days** by default.

Where the account has two-factor enabled, the sign-in flow asks for a code.

### API keys

For automation, use an [API key](/administration/api-keys/) rather than a person's password.
Keys are scoped, expiring and revocable, and actions taken with one are attributed to it in the
[audit log](/administration/audit-log/).

## Errors

Errors are RFC 7807 problem documents:

```json
{
  "type": "about:blank",
  "title": "Not Found",
  "status": 404,
  "detail": "No endpoint GET /api/nope.",
  "instance": "/api/nope",
  "timestamp": "2026-09-22T07:36:56.570860295Z"
}
```

`detail` is written to be read. Log it.

| Status | Meaning |
|---|---|
| 400 | Malformed request |
| 401 | Not signed in, or the session expired |
| 403 | Signed in, not permitted — recorded in the audit log |
| 404 | No such object or endpoint |
| 409 | Conflict — including **a write sent to a standby** |
| 429 | Rate limited |
| 500 | Server error — carries an `errorId` to quote to support |

## Paging

Collections that can grow are paged:

```
GET /api/audit?page=0&size=50
```

```json
{
  "content": [ ... ],
  "page": 0,
  "totalPages": 491,
  "totalElements": 981
}
```

Smaller collections — hosts, uplinks, templates — return a plain array.

## Rate limits

| | |
|---|---|
| Sign-in | 10 attempts per minute |
| API | 1000 requests per minute |

Exceeding either returns 429. Account lockout is separate: 5 consecutive failed sign-ins lock an
account for 15 minutes.

## Asynchronous work

Anything slow returns a **task** rather than blocking. Poll it, or subscribe over the WebSocket:

```
POST /api/vms                 → task created
GET  /api/tasks/{id}          → status and progress
```

A 200 from a create call means *accepted*, not *finished*. See [Tasks](/console/tasks/).

## Authorization

Every endpoint is authorized server-side against your role **and the domain it belongs to** —
virtualization, networking, storage, hosts or the control plane itself. A key or account scoped
to storage is refused on networking endpoints.

See [Users and roles](/administration/users-and-roles/).

## Shapes worth knowing

**Identifiers are UUIDs.** Names are for people; do not key automation on them.

**Per-host resources are nested.** Networking and storage configuration are per host:

```
/api/hosts/{hostId}/networks/interfaces
/api/hosts/{hostId}/storage/pools
```

**Secrets are write-only.** Passwords, CHAP secrets, BGP passwords and keys can be set and never
read back. An integration that needs a secret must hold it itself.

## WebSockets

Live updates — task progress, metrics, events — arrive over a WebSocket. A browser cannot put a
credential on a raw socket, so the client asks the API for a short-lived, single-use **ticket**
and connects with that. VM consoles work the same way.

## A worked example

```bash
BASE=https://vs.example.com
TOKEN=$(curl -sk -X POST $BASE/api/auth/login \
  -H 'Content-Type: application/json' \
  -d '{"username":"automation","password":"..."}' \
  | python3 -c 'import json,sys; print(json.load(sys.stdin)["access_token"])')

# Fleet state
curl -sk $BASE/api/hosts   -H "Authorization: Bearer $TOKEN"
curl -sk $BASE/api/vms     -H "Authorization: Bearer $TOKEN"

# Start a VM, then follow the task
curl -sk -X POST $BASE/api/vms/$VM_ID/start -H "Authorization: Bearer $TOKEN"
curl -sk $BASE/api/tasks -H "Authorization: Bearer $TOKEN"
```

## Versioning

The API belongs to the version of the platform serving it. Read the
[release notes](/support/release-notes/) before upgrading anything that automates against it,
and re-read the specification from the deployment afterwards rather than a cached copy.
