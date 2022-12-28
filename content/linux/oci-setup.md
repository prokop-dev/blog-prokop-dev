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

Start with configuring "Virtual Cloud Networks" (under "Networking" section).
We will use wizard, as normal setup is too convoluted (e.g. CIDR ranges needs to be defined via options busied in control panel).
Click "Start VCN Wizard" and chose "Create VCN with Internet Connectivity".
Then set the following values:

- VCN Name: `paris`, e.g. I used part of region name from region, eg. `uk-paris-1`.
- Compartment: use root compartment
- VCN CIDR Block: I highly recommend to use something smaller than `10.0/16`. I went for Private C range `192.168.123.0/24`.
- Public Subnet CIDR Block: For "always free" tenancy, 30 usable IP addresses should be OK, so go with subclass: `192.168.123.0/27`.
- Private Subnet CIDR Block: Also assign space of 32 IP addresses for not reachable from Internet hosts: `192.168.123.32/27`.

Click review / create and enjoy your VCN created. We will use it later to connect server instances to it.
Explore if the following were created for you:

- Two subnets
- Two routing tables
- One internet gateway
- Two security lists
- One DHCP option

# Setup Vault

This will allow to use full disc encryption for VMs.
Navigate to "Vault" under "Security" section.

# Create Virtual Servers

Oracle CLoud does not offer Arch Linux.
The procedure to install Arch on Oracle free tier is a bit of convoluted, although there are some gists.
The most "pure" and "not-enterpisey" OS available is Ubuntu.
As Debian derivative, it is closest to what I like work with.

The generous Oracle offering will let to create:

- Two x86 64bit 1MB Ram instances.
- 

## Setting up the AMD

Go to "Instances" under "Compute" section.
Click "Create instance" (obviously). The provide:

- Name: I use to name VPSes after location, so `paris1`, `paris2`, etc works for me.
- Create in compartment: Use root
- Placement: If you see `Always Free-eligible`, do not touch it.
- Image and shape: Click "Edit", then "Change image" and choose latest Ubuntu available. It is advisable also to select "Minimal" version. Confirm with "Select image" button.
- Networking: Click "Edit" and select VCN created previously. Chose existing public or private subnet, according to your liking.
- Add SSH keys: Use "Paste public keys" and paste SSH public key created previously (the one for Cloud Shell)
-  

