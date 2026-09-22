---
title: "Licensing"
description: "The evaluation period, requesting a licence, and what the core count is measured against."
status: published
---

VS-HCI is licensed by **processor core** across the fleet.

**Settings → Licensing** shows the current state:

```
state:             DEMO
reason:            Evaluation period — 89 days remaining, ending 2026-12-20.
deploymentMode:    CLUSTER
demoDaysRemaining: 89
hostsEnrolled:     3
enrolledCores:     30
licensedCores:     0
installUuid:       c8365585-5e21-4ab5-a3c0-16dc6ccd3deb
```

## Evaluation

A new deployment starts in **DEMO** with a **90-day** evaluation period, fully functional.

The screen counts down, and the state becomes alarming as the end approaches. Request a licence
well before it runs out — the request goes through a person at Velarion, not a vending machine.

## States

| State | Meaning |
|---|---|
| `DEMO` | Inside the evaluation period, no licence applied |
| `LICENSED` | Signed, bound to this deployment, in date, within its core count |
| `EXPIRING` | Valid, but close enough to expiry to act on |
| `OVER_LIMIT` | Valid, but the fleet has more cores than the licence covers |
| `EXPIRED` | The expiry date has passed |
| `INVALID` | Present but not usable here — wrong signature, or issued for a different deployment |

`OVER_LIMIT` is the one that arrives unannounced: it happens when you add hardware, not when you
change anything about the licence. Adding a host adds its cores to the count.

## Cores, not sockets

The count is **enrolled cores** — the sum of the physical cores across every host in the
deployment. Three hosts with ten cores each is thirty cores.

Check the core count of hardware **before** you buy it. A licence sized for your current fleet
does not cover a refresh onto denser servers.

Some licences are issued with an unlimited core count, which is distinct from a count of zero —
zero cores is a real and very different answer.

## Tiers

Licences are issued in tiers — `SMALL`, `MEDIUM` and `LARGE` — which is what you request
against.

## Requesting a licence

1. **Settings → Licensing → Request a licence.** The deployment produces a request containing
   its installation identity and a fingerprint.
2. Send it to Velarion.
3. **Apply the licence** you receive back.

The request is bound to **this installation**. A licence issued against it will not validate on
a different deployment — which is the point, and also why a rebuild needs a new request rather
than a copied file.

Past requests are listed, so you can see what was asked for and when.

## Applying a licence

Import it from the same screen. It is validated on import: the signature must check out against
the trust anchor this build carries, and it must be bound to this installation.

An `INVALID` result means one of those two failed — usually a licence for a different
deployment.

## Removing a licence

A licence can be removed, which is what you do when moving one between deployments. Removing it
returns the deployment to whatever state it would otherwise be in — which, if the evaluation
period has passed, is `EXPIRED`.

## What expiry does

An expired or over-limit licence is an alarming state shown on the Dashboard and in Settings. It
is a commercial condition; check your agreement for what it means for entitlement and support.

Do not let it arrive as a surprise: set an [alert](/monitoring/alerts/), and treat `EXPIRING` as
the reminder it is.

## Installation identity

The `installUuid` identifies this deployment and is what a licence is bound to. It survives
normal operation, including controller failover and upgrades.

Quote it when talking to Velarion about licensing — it identifies the deployment unambiguously.
