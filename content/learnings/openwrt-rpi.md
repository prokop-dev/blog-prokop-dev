---
title: "OpenWRT on Raspberry Pi"
date: 2025-02-13T17:17:17Z
draft: false
author: "Bart Prokop"
description: "My love for Open WRT"
tags: ["Linux", "openwrt"]
---

## Weekend migration from EdgeRouter Lite to Raspberry Pi 5

### Clean (re)install

```
cd /tmp/
wget https://downloads.openwrt.org/releases/24.10.0/targets/bcm27xx/bcm2712/openwrt-24.10.0-bcm27xx-bcm2712-rpi-5-squashfs-sysupgrade.img.gz
sysupgrade -v -n openwrt-24.10.0-bcm27xx-bcm2712-rpi-5-squashfs-sysupgrade.img.gz

+ add factory reset option
```

### First connect

The eth0 interface is configured that way it has static `192.168.1.1` IP address.

```
ssh-keygen -R 192.168.1.1
ssh root@192.168.1.1
passwd
```

Then update lan to use DHCP:

```
vi /etc/config/network

config interface 'lan'
        option device 'br-lan'
        option proto 'dhcp'
```

### Configure USB Ethernet adapter as WAN

```
opkg update
opkg install kmod-usb-net-rtl8152
```

Then add the following to `/etc/config/network`. This is configuration works out of the box with Sky UK:

```
config interface 'wan'
        option device   'eth1'
        option proto    'dhcp'
        option clientid '613162326333653466353036406e6f7774767c6131623263336534'

config interface 'wan6'
        option device   'eth1'
        option proto    'dhcpv6'
```

### Fix DNS server to allow my local domains

I use private IP addresses with my domains. This is to ensure things like UniFi controller hosted internally will resolve.
Add the following to `/etc/config/dhcp`

```
       list rebind_domain '/prokop.uk/'
       list rebind_domain '/prokop.dev/'
       list rebind_domain '/prokop.ovh/'
```

### Get Zero Tier working

After installing package `zerotier`, the following was the necessary configuration, note with 25th edition of OpenWrt, we got new file format and 14.1 version of client.

```
config zerotier 'global'
        option enabled '1'
        option secret ''

config network 'bart_zt_net'
        option id '1234567890abcdef'
```

Additionally, I have added all `zt` interfaces to `lan` firewall config zone.
Note, I run ZeroTier as a trusted Site-To-Site backbone for my network. Your use case might vary.

```
config zone
	option name		lan
	list   network		'lan'
	list   device		'zt+'
	option input		ACCEPT
	option output		ACCEPT
	option forward		ACCEPT
```

## Static Leases with OpenWrt

I have finally got PoE managed switch in Aguas Verdes.
It is now imperative to assign to it deterministic IP address, so I can manage it remotely.

```
vi /etc/config/dhcp

config host
        option ip       '192.168.1.2'
        option mac      '00:aa:11:bb:22:cc'
        option name     'sw-ave1'
        option dns      '1'
```

## What to install

Curated list of all packages that I installed.

```
opkg install owut

# opkg install kmod-usb-net-rtl8152 zerotier
```
