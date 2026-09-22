---
title: "Installing the first server"
description: "First-boot setup: identity, management network, and whether this node starts a cluster."
status: published
---

When a freshly installed server boots, it comes up into **VS-HCI Setup**, a text wizard on the
console. Nothing is applied until you reach the review screen and choose **Apply**, so you can
move back and forth between sections freely.

You need physical console access, or a BMC console. SSH is not available yet.

## Section 1 — Server identity

| Field | Notes |
|---|---|
| Hostname | The server's name. It becomes the host's name in the console too |
| Administrator username | Defaults to `vsadmin`. This is your account |
| Password | Minimum 8 characters |

The administrator account is created in **two** places with the same credentials: a Linux
account you use for SSH, and a console account with the **SUPER_ADMIN** role. One password,
both doors.

> **Note** — The name `hyperion` is reserved for the control plane's own identity on the
> server and the wizard refuses it. Pick anything else.

## Section 2 — Management network

This is the only network the installer configures. Everything else is created later from the
console.

**Interface.** Pick the NIC carrying the management network.

**VLAN.** An ID from 1 to 4094, then how the port presents it:

- **Untagged** — the switch port's native or access VLAN. This is the common case.
- **Tagged** — the port is a trunk carrying 802.1Q. The switch port must already be configured
  as a trunk for that VLAN.

**Addressing.** Static or DHCP. Static is strongly preferred for any node running the control
plane; the virtual IP has to live in the same subnet and a controller that moves is a problem
you do not want.

**FQDN and NTP.** The server's fully qualified name, and an NTP server. Time matters — the
control plane's clustering depends on it.

On an untagged network the wizard brings the interface up with the values you gave and tests
them immediately:

```
Connectivity check:

Gateway  OK
DNS      OK
NTP      OK
```

A failure does not block you — an air-gapped site legitimately has no DNS or NTP — but a failed
gateway check on a site that should have one means the answer is wrong.

> **Note** — On a **tagged** VLAN, no connectivity test runs here. The tag does not exist
> until the management bridge is built, so a test at this point would run on the port's native
> VLAN and tell you nothing true. Verification happens at apply time instead, and the installer
> falls back to a plain untagged interface if the tagged layout cannot reach the gateway.

## Section 3 — Deployment mode

Two choices, and the first one decides the shape of everything after it.

**Standalone** — one server, control plane and hypervisor together, no HA. This produces
VS-Hypervisor. It can be promoted to a cluster later by adding a second node.

**Cluster** — high availability. Then:

### Create a new cluster

This server is the first node. It runs the control plane, runs workloads, and owns the virtual
IP.

| Field | Notes |
|---|---|
| Virtual IP | A free address in the management subnet. The console lives here |
| Console name | DNS name that resolves to the VIP. Goes on the TLS certificate. Blank is valid — the certificate then names the VIP only |
| Node priority | **Lower wins.** 1 is the most preferred. Default 100 |

The wizard generates a **join token** and shows it on the summary screen. One token serves the
whole cluster, for both controllers and hosts. Write it down — you need it on every other
server.

### Join an existing cluster

| Field | Notes |
|---|---|
| Join as | **Controller + host** — runs the control plane and workloads. **Host only** — workloads only |
| Cluster VIP | The virtual IP of the running cluster |
| Join token | From the first node's summary screen |
| Node priority | Controllers only. Default 150 |

> **Warning** — A joining controller must have a **higher** priority number than the running
> master, because lower is more preferred. Leave it at 150 against a master at 100. A joiner
> that outranks the master can take the virtual IP before its database has finished syncing.

## Section 4 — Review and apply

The summary restates everything, including the join token on a first node:

```
  Hostname:       vsnode1
  Admin (SSH):    vsadmin
  FQDN:           vsnode1.example.com (this node)
  Console name:   vs.example.com (certificate)
  NTP server:     pool.ntp.org
  Management:     10.2.32.101/24 on eno1 (gw 10.2.32.1, dns 10.2.32.1)
  Mgmt VLAN:      232 (untagged / access port)
  Mgmt uplink:    bond99 → br-mgmt (address on mgmt-mgmt)
  Role:           Cluster master — control plane + hypervisor, owns VIP 10.2.32.240
  Node priority:  100
  Join token:     <token>
  Appliance mode: true
```

**Apply** installs the platform packages from the medium's offline pool, converts the
management interface to a bridged layout, starts the control plane and registers the server as
a host.

This takes several minutes. When it finishes, the wizard prints the console URL and the
credentials to sign in with.

## Appliance mode

`Appliance mode: true` is the default and means the server is locked down when setup finishes:

- SSH lands in the **VS-HCI shell**, not a Linux prompt.
- Your administrator account is removed from the `sudo` group.
- Reaching the operating system requires [root shell access](/cli/root-shell/), which is
  authorized per session.

This applies to every interactive account on the box, not just the one you named. Setting
`APPLIANCE=false` in a seed leaves accounts on a normal shell — appropriate for a lab, not for
production.

## Next

Bring up the rest of the cluster: [Adding servers](/getting-started/adding-servers/).
