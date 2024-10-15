---
title: "Saturday, 19th October 2024"
date: 2024-10-19T19:19:19Z
draft: false
author: "Bart Prokop"
description: "Break in the rain"
tags: ["network", "openwrt", "docker"]
---

## Aligning IPv4 and IPv6 numbering

I use following Class C for three network location:

- 192.168.16.0/20 - location 1
- 192.168.32.0/20 - location 2
- 192.168.48.0/20 - location 3

That gives 16 subnets per location, e.g. main network, guest network, IoT, DMZ, etc.
The VLAN id is added to third octet, so if I use VLAN 1 and VLAN 5, it will give:

- 192.168.17.0/24 - main network in location 1, VLAN 1.
- 192.168.21.0/24 - guest netwoek in location 1, VLAN 5.
- 192.168.49.0/24 - main network in location 3, VLAN 1.
- 192.168.53.0/24 - guest netwoek in location 3, VLAN 5.

You, get it simple schema.

By adding `ip6assign` and `ip6hint` to `/etc/config/network`, I can achieve aligment at 8th octet of IPv6 addresses.
Aligment with VLAN and IPv4 scheme.

E.g. the following:

```
config interface 'vlan1'
        option device 'br-lan.1'
        option proto 'static'
        option ipaddr '192.168.33.1'
        option netmask '255.255.255.0'
        option ip6assign '64'
        option ip6hint '11'

config interface 'vlan5'
        option device 'br-lan.5'
        option proto 'static'
        option ipaddr '192.168.37.1'
        option netmask '255.255.255.0'
        option ip6assign '64'
        option ip6hint '15'
```

produces that network numbering:

```
root@OpenWrt:~# ifconfig
br-lan.1  Link encap:Ethernet  HWaddr 78:8A:20:09:77:2E
          inet addr:192.168.33.1  Bcast:192.168.33.255  Mask:255.255.255.0
          inet6 addr: fd15:329d:90c0:11::1/64 Scope:Global
          inet6 addr: fe80::7a8a:20ff:fe09:772e/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:10232 errors:0 dropped:3 overruns:0 frame:0
          TX packets:9232 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:3260521 (3.1 MiB)  TX bytes:3099369 (2.9 MiB)

br-lan.5  Link encap:Ethernet  HWaddr 78:8A:20:09:77:2E
          inet addr:192.168.37.1  Bcast:192.168.37.255  Mask:255.255.255.0
          inet6 addr: fe80::7a8a:20ff:fe09:772e/64 Scope:Link
          inet6 addr: fd15:329d:90c0:15::1/64 Scope:Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:92 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:4440 (4.3 KiB)
```

Pretty good result - look for ULA addresses above, i.e. those starting with `fd15:329d:90c0:` prefix.
21hex == 33dec, 25hex = 27dec, etc...
