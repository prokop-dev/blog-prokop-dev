---
title: "Saturday, 10th October 2024"
date: 2024-10-05T14:56:21Z
draft: false
author: "Bart Prokop"
description: "Fixing things in my Polish home"
tags: ["network", "openwrt", "security"]
---

## Local IP addresses in public DNS and OpenWRT rebind protection, fix address resolution

First of all, I have noticed that I cannot resolve private IP addresses (RFC1918) defined in my CloudFlare DNS zone.
While on the network served by OpenWRT router, I got this error:

```
$ nslookup ****.prokop.dev
*** No internal type for both IPv4 and IPv6 Addresses (A+AAAA) records available for zt33.prokop.dev
Server:  OpenWrt.lan
Address:  fd**:****:****::1
```

And it of course works when using network in my Belfast's home office, which is currently served by EdgeRouter 4 (to be soon replaced by custome OpenWRT built on RPi5).
It also works when using public CloudFlare resolver:

```
$ nslookup ****.prokop.dev 1.1.1.1
Non-authoritative answer:
Server:  one.one.one.one
Address:  1.1.1.1

Name:    ****.prokop.dev
Addresses:  fd**:****:****:****:****:****:****:****
          192.168.***.***
```

This is happening as OpenWRT protects against [DNS Rebind attack](https://en.wikipedia.org/wiki/DNS_rebinding).
The following section in `/etc/config/dhcp` is responsible for this:

```
config dnsmasq
        option rebind_protection '1'
        option rebind_localhost '1'
```

So, I want my own domain to resolve 192.168.y.x addresses.
The fix is to add the following to `/etc/config/dhcp`:

```
list rebind_domain '/example.com/'
```

The resulting config:

```
config dnsmasq
        ...
        option rebind_protection '1'
        option rebind_localhost '1'
        list rebind_domain '/prokop.uk/'
        list rebind_domain '/prokop.dev/'
        list rebind_domain '/prokop.ovh/'
        ...
```

And after restart local addresses are resolved by dnsmasq:

```
$ nslookup ****.prokop.dev 192.168.***.1
Non-authoritative answer:
Server:  OpenWrt.lan
Address:  192.168.***.1

Name:    ****.prokop.dev
Addresses:  fd**:****:****:****:****:****:****:****
          192.168.***.***
```

### Note to myself on stability...

Checking uptime is the best way to check when I visited my parents last time...

```
root@OpenWrt:/etc/config# uptime
 14:17:43 up 44 days, 20:25,  load average: 0.00, 0.00, 0.00
```

Usually on each visit, I reconfigure something on the router.

## Upgrading OpenWRT to latest

This was tested on EdgeRouter-X.

```
root@OpenWrt:~# cd /tmp/

root@OpenWrt:/tmp# wget https://downloads.openwrt.org/releases/23.05.5/targets/ramips/mt7621/openwrt-23.05.5-ramips-mt7621-ubnt_edgerouter-x-squashfs-sysupgrade.bin
Downloading 'https://downloads.openwrt.org/releases/23.05.5/targets/ramips/mt7621/openwrt-23.05.5-ramips-mt7621-ubnt_edgerouter-x-squashfs-sysupgrade.bin'
Connecting to 146.75.122.132:443
Writing to 'openwrt-23.05.5-ramips-mt7621-ubnt_edgerouter-x-squashfs-sysupgrade.bin'
openwrt-23.05.5-rami 100% |*******************************|  5400k  0:00:00 ETA
Download completed (5530254 bytes)

root@OpenWrt:/tmp# sha256sum openwrt-23.05.5-ramips-mt7621-ubnt_edgerouter-x-squashfs-sysupgrade.bin
72e6aab18e5ad70cf2e8184d52450dcea20c65ced829790c928bd48ba4bb7724  openwrt-23.05.5-ramips-mt7621-ubnt_edgerouter-x-squashfs-sysupgrade.bin

root@OpenWrt:/tmp# sysupgrade -v openwrt-23.05.5-ramips-mt7621-ubnt_edgerouter-x-squashfs-sysupgrade.bin
Sat Oct  5 16:14:11 UTC 2024 upgrade: Saving config files...
etc/config/dhcp
etc/config/dropbear
etc/config/firewall
etc/config/luci
etc/config/network
etc/config/rpcd
etc/config/system
etc/config/ucitrack
etc/config/uhttpd
etc/config/zerotier
etc/dropbear/authorized_keys
etc/dropbear/dropbear_ed25519_host_key
etc/dropbear/dropbear_rsa_host_key
etc/group
etc/hosts
etc/inittab
etc/luci-uploads/.placeholder
etc/nftables.d/10-custom-filter-chains.nft
etc/nftables.d/README
etc/opkg/keys/b5043e70f9a75cde
etc/passwd
etc/profile
etc/rc.local
etc/shadow
etc/shells
etc/shinit
etc/sysctl.conf
etc/uhttpd.crt
etc/uhttpd.key
Sat Oct  5 16:14:11 UTC 2024 upgrade: Commencing upgrade. Closing all shell sessions.
```

And after few minutes, let's try to ssh again:

```
$ ssh root@192.168.33.1
Enter passphrase for key '/c/Users/proko/.ssh/id_ed25519':


BusyBox v1.36.1 (2024-09-23 12:34:46 UTC) built-in shell (ash)

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
 OpenWrt 23.05.5, r24106-10cc5fcd00
 -----------------------------------------------------
root@OpenWrt:~#
```

Now just need to reinstall packages that were previously installed manually (sysupgrade cannot preserve installed packages).

```
...
```