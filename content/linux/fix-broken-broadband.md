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

## End goal

Get as much as possible out of bad "consumer grade" service by leveraging Ubiquiti EdgeRouter.
Target state:

TBD_PICTURE

# Standard setup

This describes general procedure just to get connectivity in place plus some tweaks.

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

set service gui older-ciphers disable
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

# IPv6 with Hurricane Electric Tunnel Broker

First request [new tunnel here](https://tunnelbroker.net/new_tunnel.php).

## Automate local IPv4 endpoint updates

As we use dynamic IP, it is important to keep our end up-to-date. We need to set-up another ddclient instance.
The example `TUNNELID` and an `UPDATEKEY` can be found in the Advanced tab of the Tunnel Details page.

```bash
set service dns dynamic interface pppoe0 service custom-he protocol dyndns2
set service dns dynamic interface pppoe0 service custom-he server ipv4.tunnelbroker.net
set service dns dynamic interface pppoe0 service custom-he host-name TUNNELID
set service dns dynamic interface pppoe0 service custom-he login USERNAME
set service dns dynamic interface pppoe0 service custom-he password UPDATEKEY
```

This introduces the following change to configuration:

```
+                service custom-he {
+                    host-name TUNNELID
+                    login USERNAME
+                    password UPDATEKEY
+                    protocol dyndns2
+                    server ipv4.tunnelbroker.net
+                }
```

## Create Tunnel Interface

Now we can create tunnel interface. Values separated with underscore relate to HE Tunnel page.

```bash
set interfaces tunnel tun0 encapsulation sit
set interfaces tunnel tun0 description "HE.net IPv6 Tunnel"
set interfaces tunnel tun0 local-ip 0.0.0.0
set interfaces tunnel tun0 remote-ip Server_IPv4_Address
set interfaces tunnel tun0 address Client_IPv6_Address
```

Commit interface definition and try to ping HE end of tunnel.

```bash
commit
ping6 Server_IPv6_Address
```

Add a default route that routes all IPv6 traffic over the tunnel. Commit and try to ping `google.com`.

```bash
set protocols static interface-route6 ::/0 next-hop-interface tun0

commit
ping google.com
```

At this point it might be prudent to enable hardware offloading for IPv6 forwarding.

```bash
set system offload ipv6 forwarding enable
commit
save
exit
```

At this time rebooting router might be good idea. Then check if everything still works as expected.

## Security of IPv6 tunnel

Hurricane Electric Tunnel Broker offers decent [port scan](https://tunnelbroker.net/portscan.php) facility.
Here is the outcome of scanning my IPv6 WAN interface:

```
Starting Nmap 7.01 ( https://nmap.org ) at 2022-06-26 05:19 PDT
Nmap scan report for tunnel******-pt.tunnel.tserv1.****.ipv6.he.net (2001:470:****:**::2)
Host is up (0.14s latency).
Not shown: 995 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
53/tcp    open  domain
80/tcp    open  http
443/tcp   open  https
10001/tcp open  scp-config

Nmap done: 1 IP address (1 host up) scanned in 9.38 seconds
```

This flags the lack of firewall on `tun0` interface.
The Wizard, we have run initially has created WANv6_IN and WANv6_LOCAL firewall rules.
Those rules have been applied to pppoe0 interface (along the corresponsinf IPv4 rules).

```
        pppoe 0 {
            default-route auto
            description "Vodafone Broadband"
            firewall {
                in {
                    ipv6-name WANv6_IN
                    name WAN_IN
                }
                local {
                    ipv6-name WANv6_LOCAL
                    name WAN_LOCAL
                }
            }
            mtu 1500
            name-server auto
            password *********
            user-id dsl*********@broadband.vodafone.co.uk
        }
```

We will assign now IPv6 firewall rules and re-run port scan.

```bash
set interfaces tunnel tun0 firewall in ipv6-name WANv6_IN
set interfaces tunnel tun0 firewall local ipv6-name WANv6_LOCAL
```

Executing above commands will result in this addition to config file:

```
     tunnel tun0 {
         address 2001:470:****:****::2/64
         description "HE.net IPv6 Tunnel"
         encapsulation sit
+        firewall {
+            in {
+                ipv6-name WANv6_IN
+            }
+            local {
+                ipv6-name WANv6_LOCAL
+            }
+        }
         local-ip 0.0.0.0
         multicast disable
         remote-ip ***.**.**.**
         ttl 255
     }
```

Please note that as IPv6 allows for e2e connectvity from and to Internet, we'd better have WANv6_IN, before we connect other devices in the local network.
Below results of repeated nmap scan.

```
Starting Nmap 7.01 ( https://nmap.org ) at 2022-06-26 12:01 PDT
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
Nmap done: 1 IP address (0 hosts up) scanned in 3.07 seconds
```

or, when forced:

```
Starting Nmap 7.01 ( https://nmap.org ) at 2022-06-26 12:02 PDT
Nmap scan report for tunnel******-pt.tunnel.******.****.ipv6.he.net (2001:470:*:**::2)
Host is up (0.13s latency).
Not shown: 994 filtered ports
PORT     STATE  SERVICE
6666/tcp closed irc
6667/tcp closed irc
6668/tcp closed irc
6669/tcp closed irc
7000/tcp closed afs3-fileserver
9999/tcp closed abyss

Nmap done: 1 IP address (1 host up) scanned in 41.30 seconds
```

