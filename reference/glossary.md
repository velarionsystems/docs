---
title: "Glossary"
description: "The terms this documentation uses, and what they mean here."
status: published
---

**Active controller** — The controller currently serving the console and API and owning the
[virtual IP](#virtual-ip-vip). Exactly one at a time. See
[High availability](/high-availability/).

**Agent** — The process on each host that the control plane instructs. Its version is tracked
separately from the control plane's.

**Anti-affinity group** — A label keeping VMs in the same group on different hosts, so a single
host failure cannot take all of them.

**Appliance mode** — A node locked down so SSH lands in the [VS-HCI shell](/cli/) rather than a
Linux prompt, with operator accounts removed from the `sudo` group. The default.

**Availability zone** — A group of hosts sharing a failure domain, and the unit within which
workloads are placed and restarted. See
[Availability zones](/high-availability/availability-zones/).

**Chassis** — A host as the virtual networking layer knows it.

**Console** — The web interface. Also, confusingly but conventionally, a VM's screen. Context
distinguishes them.

**Console name** — The DNS name resolving to the virtual IP, and the name on the deployment's
TLS certificate.

**Controller** — A node running the control plane. A controller is usually also a host.

**DR placeholder** — The replica of a protected VM at the paired site. A real VM with real disks
and reserved memory that **cannot be started there**, because its disks are the receiving end of
a live mirror.

**Evictable** — A VM that may be powered off to make room for a more important one during a
failover.

**Fabric** — An isolated routed network domain, a VRF. Either VLAN-based or overlay. See
[Fabrics](/networking/fabrics/).

**Fabric MTU** — The MTU available inside an overlay network: the underlay MTU minus 58 bytes of
encapsulation.

**Fencing** — Cutting a lost host off shared storage before restarting its workloads elsewhere.
If it fails, nothing is restarted. See [Failover and recovery](/high-availability/failover/).

**GDS — Global Distributed Switch** — A switch providing networks that exist identically on
every host in its uplink. Five kinds: L2, L3, management, storage and replication. See
[Distributed switches](/networking/distributed-switches/).

**Geneve** — The encapsulation carrying overlay traffic between hosts. Costs 58 bytes, and runs
over UDP 6081.

**Host** — A server running workloads. A host may also be a controller.

**HyShell** — The internal name for the [VS-HCI shell](/cli/).

**Hyperconverged** — A host contributing both compute and storage. The normal role.

**Hyperion** — The platform's internal engineering name. It appears in the CLI banner and
prompt, in service names and in filesystem paths. The product is VS-HCI.

**Join token** — The short token a node presents to enrol. It buys a place in the
[Approvals](/console/approvals/) queue, not membership.

**L3-out** — An overlay fabric's edge: two hosts through which it reaches the physical network.
See [Routing and NAT](/networking/routing-and-nat/).

**LUN** — A block device exported over iSCSI. One VM disk is one LUN.

**Maintenance mode** — A host excluded from scheduling so its workloads can be moved off before
planned work.

**Overlay / underlay** — The overlay is the virtual network guests see. The underlay is the
physical network carrying the tunnels.

**PENDING (VM)** — The platform is not going to start this VM, and says why. Distinct from
`STOPPED`, which someone chose. Nothing restarts a `PENDING` VM automatically.

**Preflight** — A probe checking hardware is fit for a storage mode before building it. See
[Replicated storage](/storage/replicated/).

**Pool** — Storage on one host that VM disks live in.

**Priority (controller)** — The election ranking. **Lower is more preferred.**

**Root shell** — Authorized access to the operating system underneath, granted by a signed
challenge and time-limited. See [Root shell access](/cli/root-shell/).

**Scope** — What CLI commands currently act on: the cluster, or one host.

**Standby controller** — A controller replicating the active one's database, ready to take over.
Answers reads; returns 409 to writes.

**Storage VIP** — An address following whichever node is active in a replicated storage pair.

**Task** — A unit of asynchronous work. See [Tasks](/console/tasks/).

**Term** — A counter incremented at each controller election. A climbing term means repeated
elections.

**Transit network** — The one network in an overlay fabric with a port onto the physical VLANs,
used by the L3-out to reach your router.

**Uplink** — A bond of physical NICs behind a bridge, with a member on each participating host.
The foundation of every distributed switch. See
[Uplinks and bonds](/networking/uplinks-and-bonds/).

<span id="virtual-ip-vip"></span>**Virtual IP (VIP)** — The address of the deployment. It follows the active controller, which
is why it is the address to use for everything.

**VSOS-HCI** — The operating system image every node runs.

**VS-Hypervisor** — A single-node deployment. Same product, no peers.

**VS-HCI** — The clustered product this documentation describes.

**VTEP** — A host's tunnel endpoint address on the underlay, created when an overlay switch is
built.
