---
title: "Fix Broken Broadband"
date: 2022-06-25T10:43:47+01:00
draft: false
author: "Bart Prokop"
description: "EdgeRouter OS to the rescue of broken UK broadband"
tags: ["EdgeRouter", "Internet", "setup"]
---

# Substandard service

So it is mid-2022 and my broadband is:

- PPPoE
- No IPv6
- MTU at 1492
- locked down router based on OpenWRT that provider thinks is awesome
- no static IP

Time to fix as many things as can be fixed.

## Prereqs

There are two:

1. Call ISP and get your PPPoE username and password. Quite oftern you need make few calls/online chats.
2. Bin the provided "router" and connect something decent to ONT. I will use EdgeRouter.

# Standard setup

## Basic Wizard

TBD

## Fix MTU

Change eth0 MTU to 1508 and pppoe0 to 1500. Then test if Internet still works.

This can be tested from Windows Command Prompt:

```
# 1472(payload) + 8(ICMP) + 20(IP) = 1500 MTU

C:\Users\bart>ping www.yahoo.com -f -l 1472

Pinging new-fp-shed.wg1.b.yahoo.com [87.248.100.215] with 1472 bytes of data:
Reply from 87.248.100.215: bytes=1472 time=26ms TTL=56
Reply from 87.248.100.215: bytes=1472 time=25ms TTL=56
Reply from 87.248.100.215: bytes=1472 time=25ms TTL=56
Reply from 87.248.100.215: bytes=1472 time=27ms TTL=56

Ping statistics for 87.248.100.215:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 25ms, Maximum = 27ms, Average = 25ms
```

## Give description to pppe0 interface

This will make EdgeOS Dashboard a little more readable.

```bash
set interfaces ethernet eth0 pppoe 0 description "Vodafone Broadband"
```

## Some system settings

Not essential, but were desired for my outer model:

```bash
configure

set system domain-name mydomain.tld
set system host-name mycity
commit

delete service gui older-ciphers
delete system offload hwnat
commit

save
exit
```

# Firewall fixes

## Enable IGMP

It will allow to ping router externally.

```bash
set firewall name WAN_LOCAL rule 30 action accept
set firewall name WAN_LOCAL rule 30 description "Allow ICMP"
set firewall name WAN_LOCAL rule 30 log disable
set firewall name WAN_LOCAL rule 30 protocol icmp
```

## Remove generic MSS clamping

The wizard creates pretty annoying entry in firewall options setting clamping MSS at 1412 octets. If you run [Speed Guide TCP Analyzer](https://www.speedguide.net/analyzer.php), you will fins the following, showing that original increase of MTU to 1500 on broadband interface was not effective.

```
MTU = 1452
MTU is somewhat optimized, used with PPoE DSL broadband, SonicWall firewalls and some VPNs. If not, consider raising MTU to 1500 for optimal throughput.

MSS = 1412
Maximum useful data in each packet = 1412, which equals MSS.
```

It seems that removing firewall options completely is effective solution here.

```bash
delete firewall options
```

YMMV, but it worked for me:

```
MTU = 1500
MTU is fully optimized for broadband.
MSS = 1460
Maximum useful data in each packet = 1460, which equals MSS.
```

# Dynamic DNS with Google Domains

I use Google Domains and it seems that EdgeOS bundles version of ddclient that supports googledomains protocol:

```
/usr/sbin/ddclient --version

o 'googledomains'

The 'googledomains' protocol is used by DNS service offered by www.google.com/domains.

Configuration variables applicable to the 'googledomains' protocol are:
  protocol=googledomains       ##
  login=service-login          ## the user name provided by the admin interface
  password=service-password    ## the password provided by the admin interface
  fully.qualified.host         ## the host registered with the service.

Example ddclient.conf file entries:
  ## single host update
  protocol=googledomains,                                      \
  login=my-generated-user-name,                                \
  password=my-genereated-password                              \
  myhost.com

  ## multiple host update to the custom DNS service
  protocol=googledomains,                                      \
  login=my-generated-user-name,                                \
  password=my-genereated-password                              \
  my-toplevel-domain.com,my-other-domain.com
```

So it will be matter of wee CLI magic:

```bash
set service dns dynamic interface pppoe0 service custom-google protocol googledomains
set service dns dynamic interface pppoe0 service custom-google host-name dynhost.domain.tld
set service dns dynamic interface pppoe0 service custom-google login generated-user-name
set service dns dynamic interface pppoe0 service custom-google password genereated-password
```

