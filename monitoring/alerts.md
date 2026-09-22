---
title: "Alerts"
description: "Rules on any metric, the duration that stops them flapping, and where they notify."
status: published
---

An alert rule watches a metric and raises an event when it crosses a threshold for long enough.

A deployment with no rules is one you find out about from a user.

## A rule

| Field | Notes |
|---|---|
| Name, description | Yours |
| Metric | What to watch |
| Condition | Greater than, less than, equal, not equal, greater or equal, less or equal |
| Threshold | The value to compare against |
| Duration | How long it must hold before firing. Default **300 seconds** |
| Severity | Informational, warning, error, critical, emergency |
| Host | Optional — restrict the rule to one host |
| Notify | Email, webhook, Slack |
| Enabled | Rules can be paused without deleting them |

Rules are evaluated every **60 seconds**.

## Duration is what makes alerts usable

The duration is the most important field and the one most often left at its default without
thinking.

CPU touching 95% for a single sample is not an incident; it is a backup starting. CPU at 95%
for ten minutes is. Without a duration you get an alert for every spike, people learn to ignore
the channel, and the real alert arrives into an audience that has stopped reading.

Set the duration to how long you would need to see the condition before *you* would act.

## Severity

Severity should map to what you want to happen:

| Severity | Meaning |
|---|---|
| `EMERGENCY`, `CRITICAL` | Wake someone |
| `ERROR` | Deal with it today |
| `WARNING` | Deal with it this week |
| `INFO` | Recorded, not acted on |

A rule set where everything is critical is a rule set with no severity at all.

## Notification

| Channel | Use |
|---|---|
| **Email** | Anything non-urgent |
| **Webhook** | Feeding a ticketing or on-call system |
| **Slack** | Team visibility |

Send critical alerts somewhere that wakes people, and warnings somewhere that does not. A
webhook into your existing on-call rotation is better than email for anything urgent — email has
no escalation.

## Events

A rule that fires creates an **event**, which appears on the Dashboard and the Alerts screen
with its severity.

Events are **acknowledged**, not deleted. Acknowledging records that a person has seen it and
clears it from the active count.

```
Hyperion[vsnode1]> show alarm
Hyperion[vsnode1]> alarm acknowledge <id>
```

## Rules worth having

A starting set for a new deployment:

| Watch | Condition | Duration | Severity |
|---|---|---|---|
| Host offline | — | Immediate | Critical |
| Host memory | > 90% | 10 min | Warning |
| Host CPU | > 90% | 15 min | Warning |
| Pool capacity | > 85% | 5 min | Warning |
| Pool capacity | > 95% | 1 min | Critical |
| Storage cluster health | Not healthy | 5 min | Critical |
| Host temperature | > 75 °C | 5 min | Warning |
| Replication lag (with DR) | > your recovery point | 15 min | Error |

Capacity rules earn their keep more than CPU rules. A full storage pool fails writes under every
guest on it; a busy CPU makes things slow.

## What alerts do not cover

Alert rules watch **metrics**. Two other things are worth checking separately, because nothing
fires when they go wrong:

- **Failed [tasks](/console/tasks/)** — a repeating failure from work nobody started means
  something the platform expects to be true is not.
- **System errors** — the platform's record of faults in itself, under Settings. Each carries
  an ID worth quoting to support.

## Testing a rule

Set the threshold temporarily to something the current value already crosses, confirm the event
arrives where you expect, then put it back.

An alert rule whose notification path has never delivered a message is not a rule; it is an
intention.
