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
Then click "Create Vault". Use following options:

- Create in Compartment: use your root compartment.
- Name: put something that will allow you identify Vault later.
- Make it a virtual private vault: leave it unchecked (unless you really want make "vault backups").

The reason why I do not want to have option to "backup Vault" is that it should force me to design all my cloud deployment that way that I can forfeit all data - I should have proper independent Backup and DR procedure to rebuild everything from scratch. It is in the end "someone's else computer" as Larry once said.

After Vault is created, click on it, go to "Master Encryption Keys" and click "Crete Key" button:

- Create in Compartment: root
- Protection Mode: HSM
- Name: `vaultname-fde`, I use vault name, hyphen and broad function, here Full Disk Encryption.
- Key Shape: Algorithm: use AES
- Key Shape: Length: use 256
- Import External key: should be unchecked. We want to trust that HSM will create cryptographically strong random key.

Click "Create Key". Copy master key OCID, as you will need it later to grant `blockstorage` service access to this key. 

Now you need to allow new key to be used by block storage, so new policy to control Vault access needs to be added.

Go to "Identity" then "Policies".
Click "Create Policy". Use:

- Name: vaultname-policy
- Description: Access policy to vaultname Vault
- Compartment: root
- In "Policy Builder" click Show manual editor and enter `Allow service blockstorage to use keys in tenancy where target.key.id = '<key_OCID>'`

# Create Virtual Servers

Oracle CLoud does not offer Arch Linux.
The procedure to install Arch on Oracle free tier is a bit of convoluted, although there are some gists.
The most "pure" and "not-enterpisey" OS available is Ubuntu.
As Debian derivative, it is closest to what I like work with.

The generous Oracle offering will let to create:

- Two AMD 64bit 1GB Ram instances.
- One ARM 4core 24GB RAM instance.

## Setting up the AMD

Go to "Instances" under "Compute" section.
Click "Create instance" (obviously). The provide:

- Name: I use to name VPSes after location, so `paris1`, `paris2`, etc works for me.
- Create in compartment: Use root
- Placement: If you see `Always Free-eligible`, do not touch it.
- Image and shape: Click "Edit", then "Change image" and choose latest Ubuntu available. It is advisable also to select "Minimal" version. Confirm with "Select image" button.
- Networking: Click "Edit" and select VCN created previously. Chose existing public or private subnet, according to your liking. If using public range, ensure "Assign a public IPv4 address" is selected.
- Add SSH keys: Use "Paste public keys" and paste SSH public key created previously (the one for Cloud Shell)
-  Under "Boot volume", ensure that both "Use in-transit encryption" and "Encrypt this volume with a key that you manage" are checked. Use master FDE key, you created previously.

Click "Create" and wait.

Repeat this procedure to create second "always free" VPS.

## Meaty 4xCPU 24GB RAM 100GB SSD

As we used 50% of 200GB block storage for AMD VMs (i.e. minimum volume size per VM is 50GB), we really have two possibilities for our remaining free tier:

- 1x biggest ARM machine: 4xCPU 24GB RAM 100GB SSD
- 2x medium ARM machines: 2xCPU 12GB RAM 50GB SSD

Despite there is [https://arnoldgalovics.com/free-kubernetes-oracle-cloud/](Kubernetes way of running 4 ARM instances on OCI) for free, I decided for one biggest possible machine for my setup.
I would probably not have enough workloads for it now, but it is definitely something I will explore in future.

Here are things to do differently to the above AMD recipe.

- Shape: Please click "Change shape" and select `Ampere`, the expand the accordion and move CPU and RAM sliders to the far right. Confirm with "Select shape" button.
- Image: Chose Ubuntu, latest version in `aarch64` architecture.
- Specify a custom boot volume size: Check this and increase size from 50 GB to 100 GB.

## Remaining tasks

### DNS

Create A records for newly created servers, e.g.

```
name1.domain.com    A   2.2.2.1
name2.domain.com    A   2.2.2.2
name3.domain.com    A   2.2.2.3
```
