---
title: "Install Arch Linux on Kimsufi Ovh Eco"
date: 2022-11-13T02:56:56Z
draft: false
author: "Bart Prokop"
description: "Description how easiest way to install Arch Linux on Kimsufi KS1 aka OVH Eco dedicated server"
tags: ["Linux", "OVH", "hosting", "Cloud"]
---

In this post, I will cover my learnings from installing Arch Linux on OVH dedicated server.
I've also shared what I learnt on [Arch Linux on a VPS](https://wiki.archlinux.org/title/Arch_Linux_on_a_VPS).

# Installation procedure

*Prerequisites*: Obviously a dedicated server from Kimsufi (any OVH dedicated server should be fine).
You also must have public SSH key to SSH to the server after installation is completed.

Steps below covers installation of "Cloud Ready" images, that are [available here](https://geo.mirror.pkgbuild.com/images/latest/).

1. Navigate to [https://www.ovh.com/manager/#/dedicated/server Dedicated Servers] section in your OVH management panel, then select server you want to deploy Arch Linux to.
2. Click ... next to "Last operating system (OS) installed by OVHcloud" and choose install
3. Select "Install from custom image"
4. For "Image URL" put https://geo.mirror.pkgbuild.com/images/latest/Arch-Linux-x86_64-cloudimg.qcow2
5. For "Image type" select qcow2
6. For "Checksum type" select sha256
7. For "Image checksum" put fingerprint value from https://geo.mirror.pkgbuild.com/images/latest/Arch-Linux-x86_64-cloudimg.qcow2.SHA256
8. Enable "ConfigDrive" to enter "Server host name" and your public "SSH key" (both are mandatory for Arch Cloud Init install)
9. Click "Install the system"
10. Wait (it takes a while) for email from OVH titled "Installation of your image", it will say "Congratulations! Your dedicated server has just been installed! Connect to your server with ssh key provided during your installation."
11. Use `ssh arch@IP_ADDRESS` of our dedicated box to log-in.
12. You've completed arch installation on our box. Harden the server and customise it to your own liking.

[Offical instructions](https://docs.ovh.com/gb/en/dedicated/bringyourownimage).

# Post install

Just few post install sanity tasks

## Update SSH fingerprint on your jump box

Update local server fingerprint (if you previously SSHed to that IP and you are seeing the below when trying to connect):

```
$ ssh arch@SERVER_IP
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

Use the following to remove old server instance SSH key pinning:

```
ssh-keygen -f ~/.ssh/known_hosts -R "SERVER_IP"
```

## fix hosts file

The `/etc/hosts` file from Cloud Ready Arch Linux template is pretty sparse (I've used `paris` for hostname specified in "ConfigDrive").

```
# Static table lookup for hostnames.
# See hosts(5) for details.
127.0.0.1       paris   paris
```

The above was replaced with this:

```
# The following lines are desirable for IPv4 capable hosts
127.0.0.1       localhost

# The following lines are desirable for IPv6 capable hosts
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters

# 127.0.1.1 is often used for the FQDN of the machine
127.0.1.1       paris.prokop.dev paris
```

# Basic Security hardening

## CPU

Check your server for CPU vulnerabilities:

```
[arch@paris ~]$ grep -r . /sys/devices/system/cpu/vulnerabilities/
/sys/devices/system/cpu/vulnerabilities/spectre_v2:Not affected
/sys/devices/system/cpu/vulnerabilities/itlb_multihit:Not affected
/sys/devices/system/cpu/vulnerabilities/mmio_stale_data:Not affected
/sys/devices/system/cpu/vulnerabilities/mds:Not affected
/sys/devices/system/cpu/vulnerabilities/l1tf:Not affected
/sys/devices/system/cpu/vulnerabilities/spec_store_bypass:Not affected
/sys/devices/system/cpu/vulnerabilities/tsx_async_abort:Not affected
/sys/devices/system/cpu/vulnerabilities/spectre_v1:Not affected
/sys/devices/system/cpu/vulnerabilities/retbleed:Not affected
/sys/devices/system/cpu/vulnerabilities/srbds:Not affected
/sys/devices/system/cpu/vulnerabilities/meltdown:Not affected

[arch@paris ~]$ journalctl -k --grep=microcode
Nov 20 09:23:09 paris kernel: microcode: sig=0x30661, pf=0x8, revision=0x10d
Nov 20 09:23:09 paris kernel: microcode: Microcode Update Driver: v2.2.
```

Do not forget to install micro-code updates:

```
sudo pacman -S intel-ucode
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## Network

Server listens on the following ports:

```
sudo ss -tunlp
Netid       State         Recv-Q        Send-Q                  Local Address:Port               Peer Address:Port       Process
udp         UNCONN        0             0                             0.0.0.0:5355                    0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=11))
udp         UNCONN        0             0                          127.0.0.54:53                      0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=19))
udp         UNCONN        0             0                       127.0.0.53%lo:53                      0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=17))
udp         UNCONN        0             0                    5.196.72.20%eth0:68                      0.0.0.0:*           users:(("systemd-network",pid=351,fd=18))
udp         UNCONN        0             0                                [::]:5355                       [::]:*           users:(("systemd-resolve",pid=315,fd=13))
tcp         LISTEN        0             128                           0.0.0.0:22                      0.0.0.0:*           users:(("sshd",pid=399,fd=3))
tcp         LISTEN        0             4096                          0.0.0.0:5355                    0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=12))
tcp         LISTEN        0             4096                       127.0.0.54:53                      0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=20))
tcp         LISTEN        0             4096                    127.0.0.53%lo:53                      0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=18))
tcp         LISTEN        0             128                              [::]:22                         [::]:*           users:(("sshd",pid=399,fd=4))
tcp         LISTEN        0             4096                             [::]:5355                       [::]:*           users:(("systemd-resolve",pid=315,fd=14))
```

### SSHd

Ensure 

# Tailoring your install

## Upgrade all

First step is always to perform full system upgrade.

```bash
sudo pacman -Syu
sudo sync
sudo reboot
```

## Add your own user, shell and SSH key

First let's have zsh installed

```bash
sudo pacman -S zsh
cat /etc/shells
```

Add new local user

```bash
sudo useradd -m -s /usr/bin/zsh bart
sudo pacman -S git # required for OMZ installation
sudo su - bart
```

Perform basic initialisation as new local user:

- install oh-my-zsh
- generate SSH key
- import public key to let you log-in remotely

```zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
rm .zshrc.pre-oh-my-zsh
cp .zshrc .zshrc.backup-2022-11-13
ssh-keygen -t ed25519
cat .ssh/id_ed25519.pub
curl https://raw.githubusercontent.com/bartprokop/ssh-keys/main/bart-gcs.pub >> ~/.ssh/authorized_keys
```
