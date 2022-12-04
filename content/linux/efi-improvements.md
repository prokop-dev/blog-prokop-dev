---
title: "EFI Improvements"
date: 2022-12-04T17:58:42Z
draft: false
author: "Bart Prokop"
description: "Starting new, starting fresh"
tags: ["Linux", "EFI", "boot"]
ShowToc: false
---

I use [rEFInd](https://www.rodsbooks.com/refind/) as my EFI Boot Manager.
This post is about taking some extreme measures to achieve best possible boot experience across all my machines - both servers, PCs and laptops.
Why bother about boot?
For me it is important to have the following features available:

1. Some pre-boot environment that might be useful, if I brick my main OS.
2. Ability to boot alternative syste.

# EFI partition

I decided to always reserve 1 GB for EFI partition. Please note those recommendations:

- sss
- yyy

I keep on my EFI partition also firmware files and 

I always mount my EFI partition as `/efi` using the following entry in `/etc/fstab`:

```
# /dev/sda1 UUID=C043-FAA5
LABEL=EFI               /efi            vfat            rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro   0 2
```

# Tools

This section discusses tools, I have on my EFI partition.

## Memtest86+

Why I need it? Just to test memory after adding / removing / changing / upgrading memory modules.
Memtest is actually pretty good software to check that computer is properly operating.
Installation - memtest EFI binary should end up in `/EFI/tools` directory on EF partition:

```bash
pacman -S memtest86+-efi
cp /boot/memtest86+/memtest.efi /efi/EFI/tools/memtest86.efi
```

Note that the destination file MUST be named exactly as `memtest86.efi`, otherwise rEFInd will not find it.

## GPT fdisk

EFI version of gdisk can be downloaded from [SourceForge](https://sourceforge.net/projects/gptfdisk/files/gptfdisk/1.0.4/gdisk-binaries/gdisk-efi-1.0.4.zip/download).

Installation - extract `gdisk_x64.efi` and copy it to EFI partition:

```
unzip gdisk-efi-1.0.4.zip
cp gdisk-efi/gdisk_x64.efi /efi/EFI/tools/
```

rEFInd will automatically recognize presence of `fdisk`.

## GRUB2

TBD... (I want to load ISO files, and GRUB allows for this)

## EFI Shell

For advanced troubleshooting / admin tasks.

## PXE

To boot various OSes from my OpenWrt NAS.

# References

- https://wiki.archlinux.org/title/REFInd
