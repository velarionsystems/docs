---
title: "Tasks"
description: "How asynchronous work is tracked, and what to do when a task fails."
status: published
---

Anything that takes longer than an instant becomes a **task**. Creating a VM, migrating one,
building a storage cluster, upgrading a node, exporting a LUN — the API accepts the request,
creates a task, and returns immediately.

This is why the console never blocks on a slow operation, and why closing the browser does not
cancel anything.

## Task states

| Status | Meaning |
|---|---|
| `PENDING` | Accepted, not started |
| `RUNNING` | In progress; progress is a percentage |
| `SUCCESS` | Completed |
| `FAILED` | Did not complete — the task carries the reason |
| `CANCELLED` | Stopped by an operator |
| `TIMED_OUT` | Exceeded its time limit |

## The Tasks screen

Every task across the fleet, newest first, with its title, status, progress and start time.
Progress streams live over a WebSocket, so a running task's bar moves without reloading.

Opening a task shows its detail, including the failure reason when there is one, and which host
it ran against.

```
Hyperion[vsnode1]> show task
Hyperion[vsnode1]> show task <id>
```

## When a task fails

A failed task is usually the real explanation for whatever else looks wrong — a VM that will
not start, a host stuck in `BOOTSTRAPPING`, a pool that never appeared.

1. **Read the reason.** The task carries the error, not just the fact of failure.
2. **Check the host it ran on.** A task that failed because its host was unreachable is a host
   problem, not a task problem.
3. **Retry, once you have fixed the cause.** Tasks are retryable:

   ```
   Hyperion[vsnode1]> task retry <id>
   ```

4. **Cancel it** if it is stuck and you do not want it retried:

   ```
   Hyperion[vsnode1]> task cancel <id>
   ```

> **Note** — Retrying a task that failed for an unfixed reason will fail again the same way.
> Tasks are not a queue that drains if you wait.

## Tasks you did not start

Not every task comes from a person. The platform creates tasks for its own scheduled work —
snapshot policies, metric retention, health checks, reconciliation after a host comes back.
Seeing tasks appear on an idle cluster is normal.

A **repeating** failure from a task nobody started is worth chasing: something the platform
expects to be true is not.

## Where tasks show up

- **Tasks** — all of them.
- **Dashboard** — the most recent, as a health signal.
- **The object itself** — a VM's screen shows the tasks that acted on it.

Anything that changed state also lands in the [audit log](/administration/audit-log/), which
records who asked. Tasks record what happened; the audit log records who asked for it.
