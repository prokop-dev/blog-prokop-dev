---
title: "Site To Site VPN"
date: 2024-03-30T10:20:37+00:00
draft: false
author: "Bart Prokop"
description: "Building site-to-site VPN with OpenWRT"
tags: ["EdgeRouter", "ZeroTier", "setup"]
---

# Installation

To install ZeroTier on OpenWRT, Execute the following from shell:

```bash
opkg update
opkg install zerotier
restart
```

You should see something similar to the below:

```
Installing zerotier (1.12.2-2) to root...
Downloading https://downloads.openwrt.org/releases/23.05.3/packages/mips_24kc/packages/zerotier_1.12.2-2_mips_24kc.ipk
Installing libstdcpp6 (12.3.0-4) to root...
Downloading https://downloads.openwrt.org/releases/23.05.3/targets/ath79/generic/packages/libstdcpp6_12.3.0-4_mips_24kc.ipk
Installing kmod-tun (5.15.150-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.3/targets/ath79/generic/packages/kmod-tun_5.15.150-1_mips_24kc.ipk
Installing ip-tiny (6.3.0-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.3/packages/mips_24kc/base/ip-tiny_6.3.0-1_mips_24kc.ipk
Installing libminiupnpc (2.2.3-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.3/packages/mips_24kc/packages/libminiupnpc_2.2.3-1_mips_24kc.ipk
Installing libnatpmp1 (20150609-3) to root...
Downloading https://downloads.openwrt.org/releases/23.05.3/packages/mips_24kc/packages/libnatpmp1_20150609-3_mips_24kc.ipk
Installing libatomic1 (12.3.0-4) to root...
Downloading https://downloads.openwrt.org/releases/23.05.3/targets/ath79/generic/packages/libatomic1_12.3.0-4_mips_24kc.ipk
Configuring kmod-tun.
Configuring libstdcpp6.
Configuring ip-tiny.
Configuring libminiupnpc.
Configuring libnatpmp1.
Configuring libatomic1.
Configuring zerotier.
disabled in /etc/config/zerotier
```

Then I tried to follow [the documentation](https://openwrt.org/docs/guide-user/services/vpn/zerotier), and failed with uci command line:

```
uci delete zerotier.sample_config
root@OpenWrt:~# uci delete zerotier.sample_config
root@OpenWrt:~# uci add zerotier zt_prokop_dev
cfg0217eb
root@OpenWrt:~# uci add_list zerotier.zt_prokop_dev.join=3**************c
uci: Invalid argument
root@OpenWrt:~# uci set zerotier.zt_prokop_dev.enabled='1'
uci: Invalid argument
```

So ended manually creating `/etc/config/zerotier` file:

```
config zerotier zt_prokop_dev
        option enabled 1
        list join '3**************c'
```

Finally, run the following to establish connection to your ZeroTier network for the very first time.

```
root@OpenWrt:~# service zerotier restart
Generate secret - please wait...
```

Now, you should be connected to the ZeroTier "Earth's Switch":

```bash
zt******* Link encap:Ethernet  HWaddr AE:36:60:95:0D:95
          inet6 addr: fe80::ac36:60ff:fe95:d95/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:2800  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:7 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:746 (746.0 B)
```

If you revise now `/etc/config/zerotier` file, you should notice new 'option secret' entry there.

Final step is to add ZeroTier interfaces to `lan` firewall zone.
You need to add it as `list device`, rather than as a `list network` list in the `/etc/config/firewall` file.
Ahh... that's _inconsistency_ in OpenWrt naming.

```
config zone
        option name             lan
        list   network          'lan'
        list   device           'zt+'
        option input            ACCEPT
        option output           ACCEPT
        option forward          ACCEPT
```

Note that `zt+` will cover all ZeroTier interfaces.
