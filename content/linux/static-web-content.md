---
title: "Static Web on Google Cloud"
date: 2023-08-20T30:43:47+01:00
draft: false
author: "Bart Prokop"
description: "How to use Cloud Storage bucket to host a static website"
tags: ["Cloud Storage", "Bucket", "Web"]
---

# The needs

This post describes how I've configured a Cloud Storage bucket to host a static assets for various websites across my domains.

# Resources and references

- https://cloud.google.com/storage/docs/hosting-static-website

## Prerequisites

There are two:

1. Call ISP and get your PPPoE username and password. Quite often you need make few calls/online chats.
2. Bin the provided "router" and connect something decent to ONT. I will use [Ubiquiti EdgeRouter](https://eu.store.ui.com/collections/operator-edgemax-routers).

## End goal

Get as much as possible out of bad "consumer grade" service by leveraging Prosumer Router.
Target will look like this:

TBD_PICTURE

# Standard setup

This describes general procedure just to get connectivity in place plus some tweaks.

## Basic Wizard

TBD

## Fix MTU

Change eth0 MTU to 1508 and pppoe0 to 1500. Then test if Internet still works.

This can be tested from Windows Command Prompt:

```cmd
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

## Give meaningful description to interfaces

This will make EdgeOS Dashboard a little more readable.

```bash
set interfaces ethernet eth0 description "OpenReach ONT"
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

I use Google Domains and it seems that EdgeOS bundles version of `ddclient` that supports `googledomains` protocol:

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

As we use dynamic IP, it is important to keep our end up-to-date. We need to set-up another `ddclient` instance.
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

At this point it might be prudent to enable hardware offloading for IPv6 forwarding, as it seems to be disabled by default.

```
$ show ubnt offload

IP offload module   : loaded
IPv4
  forwarding: enabled
  vlan      : disabled
  pppoe     : enabled
  gre       : disabled
  bonding   : disabled
IPv6
  forwarding: disabled
  vlan      : disabled
  pppoe     : disabled
  bonding   : disabled

IPSec offload module: loaded

Traffic Analysis    :
  export    : disabled
  dpi       : disabled
    version       : 1.564
```

To enable offloading, invoke the following commands.

```bash
configure
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
Those rules have been applied to pppoe0 interface (along the corresponding IPv4 rules).

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

Executing above commands will result in below addition to `/config/config.boot` file:

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
Below results of repeated `nmap` scan.

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

It still makes a sense to check if WAN interface is ping-able, using some [external ping6 test](https://tools.keycdn.com/ipv6-ping).

## Gift of IPv6 to local networks

If you have more than one LAN, then request `/48` prefix via Tunnel Broker web page.

### Setting up routable /64 for single LAN segment

Those two configuration commands should do the job with "routable /64 prefix".
Copy the relevant information from "Routed IPv6 Prefixes" from tunnelbroker.net.
For adding the IPv6 address, just assign first one available `::1` in the subnet to router LAN interface.

```bash
set interfaces ethernet eth1 address 2001:470:****:****::1/64
commit

set interfaces ethernet eth1 ipv6 router-advert prefix 2001:470:****:****::/64
commit
```

### Enabling ICMP version 6 for LAN

Using the [IPv6 test](https://ipv6-test.com/) or similar will quickly reveal broken IPv6 stack, as default rules in WANv6_IN are missing ICMP rule.
You are likely to get the following recommendation.

> 1. Reconfigure your firewall
> Your router or firewall is filtering ICMPv6 messages sent to your computer.
> An IPv6 host that cannot receive ICMP messages may encounter problems like some web pages loading partially or not at all.

The remedy is to add relevant rule to WANv6_IN:

```bash
set firewall ipv6-name WANv6_IN rule 30 action accept
set firewall ipv6-name WANv6_IN rule 30 description "Allow ICMPv6"
set firewall ipv6-name WANv6_IN rule 30 log disable
set firewall ipv6-name WANv6_IN rule 30 protocol icmpv6
```

## Poor performance

Unfortunatelly there is no hardware offloading for SIT tunnels. It means performance will be restricted to something like less than 100Mbit/s.
The best test is to try download large file from Internet - [Tele2 is hosting such file](http://speedtest.tele2.net/) using both IPv4 and IPv6.
Here are my results when pulling 10GB file from server connected via Ethernet cable (my broadband is 500Mbit down FTTH).
The IPv6 test was maxing router CPU at 100%.

```
➜  ~ curl -4 http://speedtest.tele2.net/10GB.zip > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 10.0G  100 10.0G    0     0  53.7M      0  0:03:10  0:03:10 --:--:-- 57.0M
➜  ~ curl -6 http://speedtest.tele2.net/10GB.zip > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
 18 10.0G   18 1919M    0     0  9703k      0  0:18:00  0:03:22  0:14:38 9805k
```

# Closing thoughts

Let me share some end state of the router as well as plan for further work

## Configuration dumps

Here are router interfaces:

```
$ show interfaces
Codes: S - State, L - Link, u - Up, D - Down, A - Admin Down
Interface    IP Address                        S/L  Description
---------    ----------                        ---  -----------
eth0         -                                 u/u  Openreach ONT
eth1         10.111.1.1/24                     u/u  Local Network
eth2         10.111.2.1/24                     u/D  Guest Network
lo           127.0.0.1/8                       u/u
             ::1/128
pppoe0       212.***.**.**5                    u/u  Vodafone Broadband
tun0         2001:470:****:**::2/64            u/u  HE.net IPv6 Tunnel
```

## Future work

This is what I plan to work in future:

- Get Hurricane Electric / Tunnel Broker IPv6 Certification at highest (Sage) level.
- Get multiple VLANs to facilitate "Home Lab".
- Implement full site2site connectivity and routing using IPv4 and IPv6.
- Get connectivity to Cloud SDN.
