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
