---
title: "OCI Setup"
date: 2022-12-27T00:01:06Z
draft: false
author: "Bart Prokop"
description: "My Oracle Cloud Infrastructure Setup"
tags: ["personal", "blog"]
ShowToc: true
---

Oracle provides generous "free-tier" for its CLoud. This article describes the basic setup to maximize OCI "always free" tier.

# Prerequisites

Run Oracle Cloud Shell and generate SSH keypair, that you will use with OCI instances.
Here come surprise... My preferred key type is ed25519, it is the most recommended public-key algorithm available today! But if I try to create it, the following error is presented:

```
ED25519 keys are not allowed in FIPS mode
```

Apparently, Oracle is part of FIPS / NIST security theater (or NSA collaborator) and follows non-sense compliance rules by letter and spirit, ignoring this [https://csrc.nist.gov/publications/detail/fips/186/5/draft](NIST draft). OK, so let's go with ecdsa P-384 key type.

```bash
ssh-keygen -t ecdsa -b 384
```

Similar situation is with generating GPG key, no Edwards curve available.
So old (not-so-good) RSA is the only reasonable option.
No need for `--expert` option, just go with defaults (RSA, RSA, 2048), when only security theater gets some modernization, remember about rotating the keys.
Moreover, configuring `git` requires extra work and it can not even ask for passphrase until `export GPG_TTY=$(tty)` is not executed (or added to `.bashrc`).

```bash
gpg --gen-key
git config --global user.email "your@email.com"
git config --global user.name "Your Name"
git config --global commit.gpgsign true
git config --global user.signingkey XXXXXXX # use key grip: gpg --list-secret-keys --keyid-format LONG
```

# Setup Network

We decided that tenancy for each family member will be created in different region and have unique IP subnet, so we will try to have some cross-site VPN fun.
We use specific IP ranges, described here.

Start with configuring "Virtual Cloud Networking".
We will not use wizard, rather we will control every step manually.
Click "Create VCN" and set the following values:

- Name: `paris`, e.g. I used part of region name from 