---
title: "My Network Setup"
date: 2022-07-02T20:11:02+01:00
draft: false
author: "Bart Prokop"
description: "EdgeRouter OS to the rescue of broken UK broadband"
tags: ["EdgeRouter", "Internet", "setup"]
---

# Site VLANs

So for each "site" (i.e. house/flat), I keep constant VLAN schema

| VLAN | Description     | Interface | Remarks |
|------|-----------------|-----------|---------|
|    1 | Default LAN     | eth1 eth2 |         |
|    2 | Vendor Reserved |           | x       |
|    3 | Vendor Reserved |           | x       |
|    4 | IPv4 Only (DMZ) | eth1.4    | x       |
|    5 | Guest Network   | eth1.5    |       x |
|    6 | IPv6 Only       | eth1.6    | x       |
|    7 | Unassigned      | .         |         |
|    8 | Unassigned      | .         | x       |
|    9 | ISP Circuit     | .         | x       |

The above VLANs are configured on Edgeouter as well as on all the switches.

```bash
set interfaces ethernet eth1 vif 4 address 10.0.4.1/24
set interfaces ethernet eth1 vif 4 description "IPv4 DMZ"

set interfaces ethernet eth1 vif 5 address  10.0.5.1/24
set interfaces ethernet eth1 vif 5 description "Guest network"

set interfaces ethernet eth1 vif 6 description "IPv6 only"
set interfaces ethernet eth1 vif 6 address 2001:470:****:**::1/64
set interfaces ethernet eth1 vif 6 ipv6 router-advert prefix 2001:470:****:**::/64
```

