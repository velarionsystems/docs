---
title: "Certificates"
description: "Replacing the self-signed certificate the cluster generates at first boot."
status: published
---

The deployment serves the console and API over HTTPS. At first boot it generates its own
certificate, which is why the first visit warns about an unknown issuer.

Replace it. Until you do, every operator is trained to click through a certificate warning,
which is the habit that makes the warning useless when it matters.

## What is installed now

**Settings → Certificates** shows the current certificate:

```
subject:        O=Hyperion,CN=vscluster.local
issuer:         O=Hyperion,CN=vscluster.local
selfSigned:     true
notAfter:       2027-09-21T18:09:58Z
daysRemaining:  364
keyAlgorithm:   RSA (4096)
signature:      SHA256withRSA

subjectAlternativeNames:
  vscluster.local
  localhost
  10.2.32.240
  127.0.0.1
```

The generated certificate is RSA 4096 with a one-year life, and its subject alternative names
cover the console name and the virtual IP.

## The console name

The certificate is issued for the **console name** — the DNS name that resolves to the virtual
IP, which you set during first-boot setup.

If you left it blank, the certificate names the VIP only, and browsing by any other name warns
even after you install a trusted certificate. The console name can be changed afterwards, which
reissues the certificate.

Use the console name everywhere: bookmarks, scripts, monitoring. An address that is not on the
certificate is an address that warns.

## Installing your own certificate

1. **Generate a signing request.** The deployment produces a CSR with the right names in it.
2. **Have your CA sign it.**
3. **Install the issued certificate**, with its chain.

The private key never leaves the deployment — a CSR is a request to sign a public key, so
there is nothing to transport and nothing to lose.

> **Note** — Include the full chain. A certificate installed without its intermediates
> validates in some clients and not others, which produces the most confusing class of
> certificate problem.

## Regenerating a self-signed certificate

You can generate a new self-signed certificate — after changing the console name, or to renew
before expiry in a deployment that deliberately stays self-signed.

## Trusted certificates

The deployment keeps its own trust store, for verifying things it connects **to** rather than
what it serves. Certificates can be added and removed by fingerprint.

This is where a certificate goes when the deployment needs to trust an endpoint signed by an
internal CA — an external storage array, a directory server, an SCP backup target.

## The cluster's internal CA

Controllers verify each other against their own authority, separate from the certificate the
console serves. That CA is also fetched by a paired deployment during
[DR pairing](/data-protection/disaster-recovery/), where it is pinned — the far site validates
against that anchor **and nothing else**, so a stranger's certificate is refused however it was
issued.

## Expiry

The console shows days remaining. Certificates expire quietly and then everything fails at once,
so it is worth an external reminder as well as an [alert](/monitoring/alerts/).

If you use short-lived certificates from an internal CA, build the renewal into whatever issues
them; there is no automatic renewal here.

## After replacing a certificate

- Reload the console and confirm no warning.
- Check scripts and monitoring that connect over HTTPS — anything pinning the old certificate
  needs updating.
- Confirm hosts and agents are still connected.
