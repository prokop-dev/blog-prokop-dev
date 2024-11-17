---
title: "Sunday, 17th November 2024"
date: 2024-10-17T17:17:17Z
draft: false
author: "Bart Prokop"
description: "Post Holiday Fixes"
tags: ["Linux", "openwrt"]
---

## Few nice command lines

### Post sysupgrade OpenWrt package fix

I've upgraded OpenWRT to latest version, but despite configuration files being preserved, I need to reinstall packages.

Here is my latest customization command for my own easy reference.

```bash
# opkg update
# opkg install zerotier msmtp-mta wireguard-tools luci-proto-wireguard \
               prometheus-node-exporter-lua
```

### Undo rebind protection on EdgeRouter X

As I still not migrated to my new Pi5 router at my Belfast home, I needed to waste time to add my domain to whitelist private IP addresses.

```
C:\Users\bart>ssh bart@192.168.xxx.1

bart@belfast:~$ configure
[edit]

bart@belfast# set service dns forwarding options
Possible completions:
  <text>        Additional options for dns forwarding. You must
                use the syntax of dnsmasq.conf in this text-field. Using this
                without proper knowledge may result in a crashed dnsmasq daemon.
                Check system log to look for errors.


bart@belfast# set service dns forwarding options rebind-domain-ok=/prokop.dev/

bart@belfast# show
…
 service {
…
     dns {
…
         forwarding {
             cache-size 10000
             force-public-dns-boost
             listen-on eth1
             listen-on eth2
+            options rebind-domain-ok=/prokop.dev/
         }
     }
…


bart@belfast# commit

bart@belfast# save

bart@belfast:~$ reboot
```

See https://thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html for exact syntax you can put here...

If you experience problems related to "Public DNS boost option", disable it:

```
delete service dns forwarding force-public-dns-boost
set service dns forwarding name-server 1.1.1.1
set service dns forwarding name-server 1.0.0.1
set system name-server 8.8.8.8
set system name-server 8.8.4.4
set interfaces ethernet eth0 dhcp-options name-server no-update
```

I need to migrate off EdgeRouter Lite to RPi with OpenWRT ASAP.

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
