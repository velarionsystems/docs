---
title: "Settings"
description: "The administrative screens — access, certificates, licensing, HA, DR and backups."
status: published
---

**Settings** holds everything that configures the deployment rather than running a workload on
it. It is organised as tabs.

## Users & Access

Console accounts, their roles, and the domains those roles apply to. Also API keys, directory
integration, and the accounts that may sign in to each host's CLI.

See [Users and roles](/administration/users-and-roles/),
[Directory integration](/administration/directory/) and [API keys](/administration/api-keys/).

## Audit Logs

Every create, update, delete and sign-in, with the user, source address and user agent. This is
the record of *who asked*; [Tasks](/console/tasks/) is the record of what the platform then did.

See [Audit log](/administration/audit-log/).

## System Errors

Errors the platform has recorded about itself, as distinct from tasks that failed. A recurring
entry here is a defect or a misconfiguration rather than a one-off.

Each error carries an ID. Quote it when contacting support — it ties directly to the event.

## Appearance

Theme and accent colour. Saved per browser, for you — it is not a deployment-wide setting and
changing it affects nobody else.

## Certificates

The TLS certificate the console and API serve. Generate a signing request for your own
authority, install the issued certificate, and review what is currently installed.

See [Certificates](/administration/certificates/).

## Licensing

The applied licence, what it entitles, and what this installation is locked to. Also where you
request a licence and apply one.

See [Licensing](/administration/licensing/).

## Orchestrator Cluster

The control plane's own HA: which node is active, each member's priority and sync state, the
virtual IP and the interface it sits on.

This is where you adjust priorities, suspend a controller for maintenance and resume it.

See [High availability](/high-availability/).

## DR Site

Pairing with a second deployment for disaster recovery, and the replication between them.

See [Disaster recovery](/data-protection/disaster-recovery/).

## Backup & Restore

Where the control-plane database is backed up to, on what schedule, and what has been taken.
Also inspection and restore of an individual backup.

See [Backup and restore](/data-protection/backups/).

> **Warning** — Backups here cover the **control plane's database** — every definition in the
> deployment. They are not backups of your virtual machines' disks. For those, see
> [Snapshot policies](/data-protection/snapshot-policies/) and
> [Disaster recovery](/data-protection/disaster-recovery/).

## What is not here

Your own password, two-factor enrolment and personal API keys are on **Profile**, not Settings.
Settings is for things that affect other people.
