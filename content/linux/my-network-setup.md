---
title: "My Network Setup"
date: 2022-07-02T20:11:02+01:00
draft: false
author: "Bart Prokop"
description: "How I over-engineered my home network"
tags: ["EdgeRouter", "Internet", "IPv6"]
ShowToc: false
---

In this blog entry, I will cover how badly I over-engineered home network.

# Site VLANs

So for each "site" (house/flat/car/boat/portable router), I keep constant VLAN schema.
VLANs are always implemented on edge router and switches.

| VLAN | Description     | Interface | Remarks and use-case |
|------|-----------------|-----------|----------------------|
|    1 | Default LAN     | eth1      | This is always untagged on all Ethernet LAN ports |
|    2 | Vendor Reserved |           | My Netgear Switch reserves those |
|    3 | Vendor Reserved |           | My Netgear Switch reserves those |
|    4 | IPv4 Only (DMZ) | eth1.4    | All internet exposed servers are here |
|    5 | Guest Network   | eth1.5    | The Guest Network with FW Isolation |
|    6 | IPv6 Only       | eth1.6    | Globally routable IPv6 only network |
|    7 | IoT Devices     |           | IoT devices, I prefer not to mix with trusted stuff for security reasons |
|    8 | Unassigned      |           | Future Use |
|    9 | ISP Circuit     | WAN       | ONT connected via switch |
|   10 | Unassigned      |           | Future Use |
|   11 | Unassigned      |           | Future Use |
|   12 | Unassigned      |           | Future Use |
|   13 | Unassigned      |           | Future Use |
|   14 | Unassigned      |           | Future Use |
|   15 | Unassigned      |           | Future Use |

The above VLANs are configured on EdgeRouter as well as on all the switches.
Below are actual names used in my networks across VLANs.
I try always stick to those when configuring routers and switches.

```bash
set interfaces ethernet eth1 vif 4 description "IPv4 DMZ"
set interfaces ethernet eth1 vif 6 description "IPv6 Only"
```

## Performance

It is advised that VLAN offloading is enabled for IPv4 and IPv6.

```bash
set system offload ipv4 vlan enable
set system offload ipv6 vlan enable
```

# Networks

I'm trying to align network numbering with VLANs.
For IPv4 network, I match last 4 bits of network number with VLAN number.
For IPv6 network, I match network number with VLAN number.
For example:

| Site   | VLAN | IPv4 Network    | IPv6 Network         |
|--------|------|-----------------|----------------------|
| Berlin |    1 | 192.168.1.0/24  | 2001:db8:0:0001::/64 |
| Berlin |    5 | 192.168.5.0/24  | 2001:db8:0:0005::/64 |
| Berlin |    6 |                 | 2001:db8:0:0006::/64 |
| Berlin |    7 | 192.168.7.0/24  | 2001:db8:0:0007::/64 |
| London |    1 | 192.168.17.0/24 | 2001:db8:0:1601::/64 |
| London |    5 | 192.168.21.0/24 | 2001:db8:0:1605::/64 |
| London |    6 |                 | 2001:db8:0:1606::/64 |
| London |    7 | 192.168.23.0/24 | 2001:db8:0:1607::/64 |

While analyzing the above table, note that I might have up to 16 networks per site.
I think it is quite a number for a personal use, even for "power user".
It helps a lot with routing, as it is very easy to use network sub-classing.

## Guest Network

```bash
set interfaces ethernet eth1 vif 5 address 192.168.21.1/24
set interfaces ethernet eth1 vif 5 description "Guest Network"

set service dhcp-server shared-network-name GUEST description "Guest Network dhcpd"
set service dhcp-server shared-network-name GUEST authoritative enable
set service dhcp-server shared-network-name GUEST subnet 192.168.21.0/24 start 192.168.21.2 stop 192.168.21.254
set service dhcp-server shared-network-name GUEST subnet 192.168.21.0/24 default-router 192.168.21.1
set service dhcp-server shared-network-name GUEST subnet 192.168.21.0/24 dns-server 8.8.8.8
set service dhcp-server shared-network-name GUEST subnet 192.168.21.0/24 dns-server 8.8.4.4
set service dhcp-server shared-network-name GUEST subnet 192.168.21.0/24 lease 3600
```

This network is also always broadcasted on WiFi Access Points as `SiteGuest`.

## IoT Devices

```bash
set interfaces ethernet eth1 vif 7 address 192.168.23.1/24
set interfaces ethernet eth1 vif 7 description "IoT Devices"

set service dhcp-server shared-network-name IOT description "IoT Devices dhcpd"
set service dhcp-server shared-network-name IOT authoritative enable
set service dhcp-server shared-network-name IOT subnet 192.168.23.0/24 start 192.168.23.101 stop 192.168.23.199
set service dhcp-server shared-network-name IOT subnet 192.168.23.0/24 default-router 192.168.23.1
set service dhcp-server shared-network-name IOT subnet 192.168.23.0/24 dns-server 1.1.1.1
set service dhcp-server shared-network-name IOT subnet 192.168.23.0/24 dns-server 1.0.0.1
set service dhcp-server shared-network-name IOT subnet 192.168.23.0/24 lease 86400
```

This network is also always broadcasted on WiFi Access Points as `IoT`.

## IPv6 Only

This VLAN offers only globally routable IPv6 addresses.
There are multiple reasons why I have dedicated VLAN for IPv6 network:

- Curiosity about pure IPv6 mileage
- It makes easier to configure servers with IPv4, IPv6 or both, if you can just connect server to one, other or both VLANs.
- Solves the problem of prefix preference. For example I use ULA (less preferred to IPv4 on basic LAN) and will connect host to IPv6 only VLAN if I really want it to have routable global IPv6. Note that ULAs are also providing IPv6 connectivity on basic LAN.

The following configuration will 

```bash
set interfaces ethernet eth1 vif 6 address 2001:db8:****:**::6/64
set interfaces ethernet eth1 vif 6 ipv6 router-advert prefix 2001:470:****:**::/64
set interfaces ethernet eth1 vif 6 ipv6 router-advert name-server 2001:4860:4860::8888
set interfaces ethernet eth1 vif 6 ipv6 router-advert name-server 2001:4860:4860::8844
```
