---
title: "Command reference"
description: "The command tree, grouped by what each branch acts on."
status: published
---

`?` or `help` at any depth lists what may follow. Tab completes, including live VM and host
names.

## Scope

```
scope orchestrator            act on the cluster
scope host <name>             act on one host
show scope                    what am I acting on
```

## show — operational state

### System and cluster

```
show version                  controller version
show system status            hosts, agents, VMs, cluster load, recent tasks
show system errors            faults the platform recorded in itself
show dashboard                the dashboard summary
show cluster                  cluster overview
show cluster status           members, roles, priorities, sync state
show cluster nodes [pending]  members, or those awaiting approval
show cluster join-token       the token joining nodes need
```

### Hosts

```
show host                             all hosts
show host <name>                      one host
show host <name> metrics              its metrics
show host <name> sensors              temperatures and hardware sensors
show host <name> disks                its disks
show host pending-approval            hosts awaiting admission
show host join-token
```

### Virtual machines

```
show vm                       all VMs
show vm <name>                one VM
show vm <name> xml            its domain definition
show vm <name> snapshots
show vm <name> disks
show vm <name> nics
```

### Networking

```
show network interfaces       host interfaces
show network physical         physical NICs
show network routes
show network vswitch
show network bridge
show network port-group
show network netplan
```

### Storage

```
show storage pool             storage pools
show storage zfs pool
show storage zfs dataset
show storage mirror           software RAID
show storage iso              the ISO library
```

### Everything else

```
show task [<id>]              tasks, or one task
show alarm [rule]             alert events, or the rules
show audit                    the audit log
show user                     console accounts
show datacenter
show container
show diagnostics              the diagnostic catalogue
show rootshell sessions       active elevated sessions
```

## Acting

### Virtual machines

```
vm start <name>
vm stop <name>
vm pause <name>
vm resume <name>
vm reboot <name>
vm migrate <name> to <host>
```

### Hosts

```
host reboot <name>
host approve <name>
host maintenance <name> on|off
```

### Cluster

```
cluster node approve <id|ip>
cluster node suspend <id|ip>
cluster node resume <id|ip>
```

### Tasks and alerts

```
task cancel <id>
task retry <id>
alarm acknowledge <id>
```

### Storage

```
storage iscsi discover <portal>
```

### Diagnostics

```
run diagnostic <key>          run one read-only check
```

See [Diagnostics](/monitoring/diagnostics/) for the catalogue.

### System operations

```
request upgrade controller <version>
```

See [Upgrading the fleet](/updating/fleet-upgrade/).

### Root shell

```
rootshell                     request authorized OS access
rootshell status              active elevated sessions and time remaining
rootshell revoke <session>    end one
```

See [Root shell access](/cli/root-shell/).

## Worked examples

**Is the cluster healthy?**

```
Hyperion[vsnode1]> show cluster status
```

One `ACTIVE`, everything `ALIVE`, everything `SYNC OK`.

**Empty a host for maintenance**

```
Hyperion[vsnode1]> host maintenance vsnode2 on
Hyperion[vsnode1]> show vm
```

**Why did that fail?**

```
Hyperion[vsnode1]> show task
Hyperion[vsnode1]> show task <id>
Hyperion[vsnode1]> task retry <id>
```

**Is the overlay working?**

```
Hyperion[vsnode1]> run diagnostic net.geneve
```

Silence on an overlay fabric means it is broken.

**Admit a new node**

```
Hyperion[vsnode1]> show host pending-approval
Hyperion[vsnode1]> host approve vsnode4
Hyperion[vsnode1]> show cluster nodes pending
Hyperion[vsnode1]> cluster node approve 10.2.32.104
```

## Scripting

The shell is interactive; `ssh host <command>` is refused. For automation, use the
[REST API](/reference/api/) with an [API key](/administration/api-keys/).

Inside a root shell, `hyshell <command>` runs a single command from the Linux prompt, which is
the supported way to script against the CLI on a node.
