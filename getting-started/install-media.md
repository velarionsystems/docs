---
title: "Creating install media"
description: "Writing the VSOS-HCI image to a USB stick, or presenting it over a BMC."
status: published
---

VS-HCI ships as a single bootable image that contains the operating system, the platform and
every package a node needs. Nothing is fetched from the internet during installation, so the
same medium works at a connected site and an air-gapped one.

The file is named for the version and architecture it carries:

```
vsos-hci-4.0.1-amd64.iso
```

## Writing it to a USB stick

The image is a hybrid ISO — write it to the raw device, do not copy it onto a filesystem and do
not "burn" it as a data disc.

**Linux**

```bash
sudo dd if=vsos-hci-4.0.1-amd64.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

Replace `/dev/sdX` with the whole device, not a partition (`/dev/sdb`, not `/dev/sdb1`). Check
it with `lsblk` first — `dd` will not ask twice.

**macOS**

```bash
diskutil list                       # identify the disk
diskutil unmountDisk /dev/diskN
sudo dd if=vsos-hci-4.0.1-amd64.iso of=/dev/rdiskN bs=4m
```

**Windows**

Use a tool that writes images in raw/DD mode — Rufus in "DD Image" mode, or balenaEtcher.

> **Warning** — Rufus defaults to ISO mode, which rewrites the boot structure and produces a
> stick that boots to a manual installer instead of the VS-HCI wizard. Choose **DD Image** when
> it asks.

## Presenting it over a BMC

On a server with out-of-band management — iDRAC, iLO, XClarity, a generic IPMI implementation —
attach the ISO as virtual media and set the server to boot from it once. This is the better
option for a rack: no physical stick, and the install can be driven from the same place you
watch it.

Virtual media is slower than a USB stick. Expect the install to take longer, and keep the
management session open for its duration.

## Booting

Boot the server from the medium. The installer is automated and unattended up to the point
where the platform is on disk — it partitions the system disk, installs the base system and
reboots.

> **Warning** — The installer **erases the system disk** and repartitions it without
> confirming. Make sure the server has nothing on it you want to keep, and that the disk you
> intend as the system disk is the one the server boots from.

After that reboot, the server comes up into first-boot setup. That is the part you drive —
see [Installing the first server](/getting-started/first-server/).

## Unattended rollout

For a large deployment you can skip the setup screens entirely by placing an answer file at
`/etc/hyperion/wizard-seed.conf` before first boot, typically via a PXE or kickstart
environment. When that file exists the wizard reads it and configures the node without
prompting.

The file is shell-style `KEY=value` lines. These must be present:

| Key | Meaning |
|---|---|
| `ADMIN_USER` | Administrator account name — not `hyperion`, which is reserved |
| `ADMIN_PASS` | Its password; minimum 8 characters, used for both SSH and the console |
| `MGMT_IF` | Management interface, e.g. `eno1` |
| `MODE` | `standalone` or `cluster` |
| `MGMT_CIDR`, `GATEWAY` | Required when addressing is static, which is the default |
| `VIP` | Required for `cluster`, whether creating or joining |
| `JOIN_TOKEN` | Required when joining as a controller |
| `HOST_TOKEN_IN` | Required when joining as a host |

Everything else has a default: `MGMT_VLAN=1` untagged, `NTP=pool.ntp.org`,
`FQDN=<hostname>.local`, `MGMT_ADDRESSING=static`, and `APPLIANCE=true`.

A seed that is missing a required key is rejected and the node is left unconfigured rather than
half-built.
