---
title: "Console and terminal"
description: "Reaching a guest's screen and shell from the browser."
status: published
---

Every VM has a graphical console in the console, proxied over a WebSocket. You do not need
network access to the guest, or even a working guest network — the console attaches to the
virtual display the same way a monitor would.

This is how you install an operating system, fix a guest that has lost its network, and see a
boot that is failing.

## Opening the console

Open the VM and choose **Console**. The guest's screen appears. Keyboard and mouse go through.

The console works for any VM that is running, regardless of its network configuration. That is
the point: a guest whose firewall you have just broken is still reachable here.

## How access is authorized

The browser cannot hold your session credential on a raw WebSocket, so the console asks the API
for a short-lived, single-use **ticket** and connects with that. The ticket is scoped to one VM
and expires quickly.

Practically, this means console sessions cannot be shared by copying a URL, and a console left
open does not become a standing door into the guest.

Opening a console is recorded in the [audit log](/administration/audit-log/).

## Attaching installation media

From the console screen you can attach an ISO from the [ISO library](/storage/iso-library/) and
change it without stopping the VM — which is what a multi-disc installation or a driver disk
needs.

You can also mount the **guest tools** ISO here, and eject it when the install is done.

## Virtual USB

Files can be presented to a running guest as a USB mass-storage device — useful for getting a
driver, a licence file or a configuration bundle into a VM that has no network yet. See
[Virtual USB](/vms/virtual-usb/).

## The host terminal

Separately from a guest's console, each **host** has a terminal on its detail screen in
[Infrastructure](/console/infrastructure/). That one opens the VS-HCI CLI on the server — not a
Linux shell, and not the guest.

It is the same restriction SSH has, for the same reason: a browser tab that opened a root shell
would walk straight past the authorization that [root shell access](/cli/root-shell/) exists to
enforce. Sessions are audited when they open and when they close.

## When the console does not open

| Symptom | Likely cause |
|---|---|
| Console blank, VM `RUNNING` | The guest is running but not drawing — check it actually booted |
| Console will not connect | The VM's host is unreachable; check it in Infrastructure |
| Connects then drops | The ticket expired before the socket established — retry |
| No console option | The VM is not running |

If the VM is on a host that is `OFFLINE`, the console has nothing to attach to. Fix the host
first.
