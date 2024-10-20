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
        option ip6hint '21'

config interface 'vlan5'
        option device 'br-lan.5'
        option proto 'static'
        option ipaddr '192.168.37.1'
        option netmask '255.255.255.0'
        option ip6assign '64'
        option ip6hint '25'
```

produces that network numbering:

```
root@OpenWrt:~# ifconfig
br-lan.1  Link encap:Ethernet  HWaddr 78:8A:20:09:77:2E
          inet addr:192.168.33.1  Bcast:192.168.33.255  Mask:255.255.255.0
          inet6 addr: fd15:329d:90c0:21::1/64 Scope:Global
          inet6 addr: fe80::7a8a:20ff:fe09:772e/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:10232 errors:0 dropped:3 overruns:0 frame:0
          TX packets:9232 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:3260521 (3.1 MiB)  TX bytes:3099369 (2.9 MiB)

br-lan.5  Link encap:Ethernet  HWaddr 78:8A:20:09:77:2E
          inet addr:192.168.37.1  Bcast:192.168.37.255  Mask:255.255.255.0
          inet6 addr: fe80::7a8a:20ff:fe09:772e/64 Scope:Link
          inet6 addr: fd15:329d:90c0:25::1/64 Scope:Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:92 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:4440 (4.3 KiB)
```

Pretty good result - look for ULA addresses above, i.e. those starting with `fd15:329d:90c0:` prefix.
21hex == 33dec, 25hex = 27dec, etc...

## Building Raspberry Pi 5 router.

### Adding cheap USB 3.0 Gigabit adapter.

Found in my drawer that one: https://www.amazon.co.uk/gp/product/B010SEARPU/ref=ppx_yo_dt_b_search_asin_title?ie=UTF8&psc=1
Purchased long time ago, and waiting to be used. Here is what `dmesg` reports:

```
[1789154.882570] usb 2-1: new SuperSpeed USB device number 2 using xhci-hcd
[1789154.918564] usb 2-1: New USB device found, idVendor=0bda, idProduct=8153, bcdDevice=30.00
[1789154.926955] usb 2-1: New USB device strings: Mfr=1, Product=2, SerialNumber=6
[1789154.934315] usb 2-1: Product: USB 10/100/1000 LAN
[1789154.939217] usb 2-1: Manufacturer: Realtek
[1789154.943521] usb 2-1: SerialNumber: 000001
```

The above suggests it is Realtek 8153. Let's dig a bit to get more info and hopefully get it recognized as eth1.

```
opkg install usbutils
root@OpenWrt:~# lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux 6.6.52 xhci-hcd xHCI Host Controller
Bus 002 Device 001: ID 1d6b:0003 Linux 6.6.52 xhci-hcd xHCI Host Controller
Bus 002 Device 002: ID 0bda:8153 Realtek USB 10/100/1000 LAN
Bus 003 Device 001: ID 1d6b:0002 Linux 6.6.52 xhci-hcd xHCI Host Controller
Bus 004 Device 001: ID 1d6b:0003 Linux 6.6.52 xhci-hcd xHCI Host Controller
```

```
root@OpenWrt:~# lsusb -v
Bus 002 Device 002: ID 0bda:8153 Realtek USB 10/100/1000 LAN
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               3.00
  bDeviceClass            0 [unknown]
  bDeviceSubClass         0 [unknown]
  bDeviceProtocol         0
  bMaxPacketSize0         9
  idVendor           0x0bda Realtek
  idProduct          0x8153 USB 10/100/1000 LAN
  bcdDevice           30.00
  iManufacturer           1 Realtek
  iProduct                2 USB 10/100/1000 LAN
  iSerial                 6 000001
  bNumConfigurations      2
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0039
    bNumInterfaces          1
    bConfigurationValue     1
    iConfiguration          0
    bmAttributes         0xa0
      (Bus Powered)
      Remote Wakeup
    MaxPower              288mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           3
      bInterfaceClass       255 [unknown]
      bInterfaceSubClass    255 [unknown]
      bInterfaceProtocol      0
      iInterface              0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0400  1x 1024 bytes
        bInterval               0
        bMaxBurst               3
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x02  EP 2 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0400  1x 1024 bytes
        bInterval               0
        bMaxBurst               3
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0002  1x 2 bytes
        bInterval               8
        bMaxBurst               0
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x0062
    bNumInterfaces          2
    bConfigurationValue     2
    iConfiguration          0
    bmAttributes         0xa0
      (Bus Powered)
      Remote Wakeup
    MaxPower              288mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         2 [unknown]
      bInterfaceSubClass      6 [unknown]
      bInterfaceProtocol      0
      iInterface              5 CDC Communications Control
      CDC Header:
        bcdCDC               1.10
      CDC Union:
        bMasterInterface        0
        bSlaveInterface         1
      CDC Ethernet:
        iMacAddress                      3 00E04C6B1289
        bmEthernetStatistics    0x00000000
        wMaxSegmentSize               1514
        wNumberMCFilters            0x0000
        bNumberPowerFilters              0
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x83  EP 3 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0010  1x 16 bytes
        bInterval               8
        bMaxBurst               0
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       0
      bNumEndpoints           0
      bInterfaceClass        10 [unknown]
      bInterfaceSubClass      0 [unknown]
      bInterfaceProtocol      0
      iInterface              0
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       1
      bNumEndpoints           2
      bInterfaceClass        10 [unknown]
      bInterfaceSubClass      0 [unknown]
      bInterfaceProtocol      0
      iInterface              4 Ethernet Data
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0400  1x 1024 bytes
        bInterval               0
        bMaxBurst               3
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x02  EP 2 OUT
        bmAttributes            2
          Transfer Type            Bulk
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0400  1x 1024 bytes
        bInterval               0
        bMaxBurst               3
Binary Object Store Descriptor:
  bLength                 5
  bDescriptorType        15
  wTotalLength       0x0016
  bNumDeviceCaps          2
  USB 2.0 Extension Device Capability:
    bLength                 7
    bDescriptorType        16
    bDevCapabilityType      2
    bmAttributes   0x00000002
      HIRD Link Power Management (LPM) Supported
  SuperSpeed USB Device Capability:
    bLength                10
    bDescriptorType        16
    bDevCapabilityType      3
    bmAttributes         0x02
      Latency Tolerance Messages (LTM) Supported
    wSpeedsSupported   0x000e
      Device can operate at Full Speed (12Mbps)
      Device can operate at High Speed (480Mbps)
      Device can operate at SuperSpeed (5Gbps)
    bFunctionalitySupport   2
      Lowest fully-functional device speed is High Speed (480Mbps)
    bU1DevExitLat          10 micro seconds
    bU2DevExitLat        2047 micro seconds
Device Status:     0x0010
  (Bus Powered)
  Latency Tolerance Messaging (LTM) Enabled
```

From internet search, it seems that for Realtek 8153, the `kmod-usb-net-rtl8152` module should be loaded.

```
root@OpenWrt:~# opkg install kmod-usb-net-rtl8152
Installing kmod-usb-net-rtl8152 (6.6.52-r1) to root...
Downloading https://downloads.openwrt.org/snapshots/targets/bcm27xx/bcm2712/kmods/6.6.52-1-b6fec427941eaba47c30194b8e61bef5/kmod-usb-net-rtl8152_6.6.52-r1_aarch64_cortex-a76.ipk
Collected errors:
 * pkg_hash_check_unresolved: cannot find dependency kernel (= 6.6.57~f2bf037eb1aee57cefd90e0e10838bff-r1) for kmod-usb-core
 * pkg_hash_check_unresolved: cannot find dependency kernel (= 6.6.57~f2bf037eb1aee57cefd90e0e10838bff-r1) for kmod-usb-net
 * pkg_hash_check_unresolved: cannot find dependency kernel (= 6.6.57~f2bf037eb1aee57cefd90e0e10838bff-r1) for kmod-usb-net-cdc-ether
 * pkg_hash_check_unresolved: cannot find dependency kernel (= 6.6.57~f2bf037eb1aee57cefd90e0e10838bff-r1) for kmod-usb-net-rtl8152
 * satisfy_dependencies_for: Cannot satisfy the following dependencies for kmod-usb-net-rtl8152:
 *      kernel (= 6.6.57~f2bf037eb1aee57cefd90e0e10838bff-r1)
 * opkg_install_cmd: Cannot install package kmod-usb-net-rtl8152.
```

Seems it did not get that easy...
Can't find anything conclusive on web...
Maybe it is down to snapshot nature of OpenWrt RPi5 image and simply too much time passed between install and my attempt to add secondary Ethernet?
Let's try to upgrade OpenWrt install...

```
root@OpenWrt:/tmp# wget https://downloads.openwrt.org/snapshots//targets/bcm27xx/bcm2712/openwrt-bcm27xx-bcm2712-rpi-5-s
quashfs-sysupgrade.img.gz
Downloading 'https://downloads.openwrt.org/snapshots//targets/bcm27xx/bcm2712/openwrt-bcm27xx-bcm2712-rpi-5-squashfs-sysupgrade.img.gz'
Connecting to 151.101.2.132:443
Writing to 'openwrt-bcm27xx-bcm2712-rpi-5-squashfs-sysupgrade.img.gz'
openwrt-bcm27xx-bcm2 100% |*******************************| 11146k  0:00:00 ETA
Download completed (11413644 bytes)

root@OpenWrt:/tmp# sha256sum openwrt-bcm27xx-bcm2712-rpi-5-squashfs-sysupgrade.img.gz
0c2b8888348456fed5e217df966ff95d3fa9d582b3e06f141d312935f522a3d8  openwrt-bcm27xx-bcm2712-rpi-5-squashfs-sysupgrade.img.gz

root@OpenWrt:/tmp# sysupgrade -i openwrt-bcm27xx-bcm2712-rpi-5-squashfs-sysupgrade.img.gz
Sun Oct 20 19:53:55 UTC 2024 upgrade: Reading partition table from bootdisk...
Sun Oct 20 19:53:55 UTC 2024 upgrade: Reading partition table from image...
Keep config files over reflash (Y/n):
Edit config file list (y/N):
Sun Oct 20 19:54:05 UTC 2024 upgrade: Saving config files...
Sun Oct 20 19:54:06 UTC 2024 upgrade: Commencing upgrade. Closing all shell sessions.
Command failed: Connection failed
root@OpenWrt:/tmp# Connection to 192.168.17.160 closed by remote host.
Connection to 192.168.17.160 closed.
```

And let's try again:

```
root@OpenWrt:~# opkg install kmod-usb-net-rtl8152
Installing kmod-usb-net-rtl8152 (6.6.57-r1) to root...
Downloading https://downloads.openwrt.org/snapshots/targets/bcm27xx/bcm2712/kmods/6.6.57-1-f2bf037eb1aee57cefd90e0e10838bff/kmod-usb-net-rtl8152_6.6.57-r1_aarch64_cortex-a76.ipk
Installing r8152-firmware (20241017-r1) to root...
Downloading https://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a76/base/r8152-firmware_20241017-r1_aarch64_cortex-a76.ipk
Installing kmod-crypto-sha256 (6.6.57-r1) to root...
Downloading https://downloads.openwrt.org/snapshots/targets/bcm27xx/bcm2712/kmods/6.6.57-1-f2bf037eb1aee57cefd90e0e10838bff/kmod-crypto-sha256_6.6.57-r1_aarch64_cortex-a76.ipk
Installing kmod-mii (6.6.57-r1) to root...
Downloading https://downloads.openwrt.org/snapshots/targets/bcm27xx/bcm2712/kmods/6.6.57-1-f2bf037eb1aee57cefd90e0e10838bff/kmod-mii_6.6.57-r1_aarch64_cortex-a76.ipk
Installing kmod-usb-net (6.6.57-r1) to root...
Downloading https://downloads.openwrt.org/snapshots/targets/bcm27xx/bcm2712/kmods/6.6.57-1-f2bf037eb1aee57cefd90e0e10838bff/kmod-usb-net_6.6.57-r1_aarch64_cortex-a76.ipk
Installing kmod-usb-net-cdc-ether (6.6.57-r1) to root...
Downloading https://downloads.openwrt.org/snapshots/targets/bcm27xx/bcm2712/kmods/6.6.57-1-f2bf037eb1aee57cefd90e0e10838bff/kmod-usb-net-cdc-ether_6.6.57-r1_aarch64_cortex-a76.ipk
Installing kmod-usb-net-cdc-ncm (6.6.57-r1) to root...
Downloading https://downloads.openwrt.org/snapshots/targets/bcm27xx/bcm2712/kmods/6.6.57-1-f2bf037eb1aee57cefd90e0e10838bff/kmod-usb-net-cdc-ncm_6.6.57-r1_aarch64_cortex-a76.ipk
Configuring kmod-mii.
Configuring kmod-crypto-sha256.
Configuring kmod-usb-net.
Configuring kmod-usb-net-cdc-ether.
Configuring kmod-usb-net-cdc-ncm.
Configuring r8152-firmware.
Configuring kmod-usb-net-rtl8152.
```

Wow. It was just dependency problem... Let's check `dmesg` again:

```
[ 1594.247562] kmodloader: loading kernel modules from /etc/modules.d/*
[ 1594.255381] usbcore: registered new device driver r8152-cfgselector
[ 1594.497498] r8152-cfgselector 2-1: reset SuperSpeed USB device number 2 using xhci-hcd
[ 1594.645495] r8152 2-1:1.0: load rtl8153a-4 v2 02/07/20 successfully
[ 1594.707148] r8152 2-1:1.0 eth1: v1.12.13
[ 1594.711127] usbcore: registered new interface driver r8152
[ 1594.717588] usbcore: registered new interface driver cdc_ether
[ 1594.723797] usbcore: registered new interface driver cdc_ncm
[ 1594.729527] kmodloader: done loading kernel modules from /etc/modules.d/*
[ 1594.742757] kmodloader: loading kernel modules from /etc/modules.d/*
[ 1594.749678] kmodloader: done loading kernel modules from /etc/modules.d/*
[ 1594.763224] kmodloader: loading kernel modules from /etc/modules.d/*
[ 1594.770094] kmodloader: done loading kernel modules from /etc/modules.d/*
[ 1594.783342] kmodloader: loading kernel modules from /etc/modules.d/*
[ 1594.790226] kmodloader: done loading kernel modules from /etc/modules.d/*
[ 1594.803678] kmodloader: loading kernel modules from /etc/modules.d/*
[ 1594.810570] kmodloader: done loading kernel modules from /etc/modules.d/*
[ 1594.829175] kmodloader: loading kernel modules from /etc/modules.d/*
[ 1594.836123] kmodloader: done loading kernel modules from /etc/modules.d/*
```

Perfect, seems I have `eth1` now.

```
br-lan    Link encap:Ethernet  HWaddr 2C:CF:67:7B:31:16
          inet addr:192.168.17.160  Bcast:192.168.17.255  Mask:255.255.255.0
          inet6 addr: fe80::2ecf:67ff:fe7b:3116/64 Scope:Link
          inet6 addr: fd03:dd25:6304::1/60 Scope:Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:56900 errors:0 dropped:34 overruns:0 frame:0
          TX packets:6994 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:6953213 (6.6 MiB)  TX bytes:1079562 (1.0 MiB)

eth0      Link encap:Ethernet  HWaddr 2C:CF:67:7B:31:16
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:57496 errors:0 dropped:61 overruns:0 frame:0
          TX packets:6998 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:7000194 (6.6 MiB)  TX bytes:1086931 (1.0 MiB)
          Interrupt:102

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:232 errors:0 dropped:0 overruns:0 frame:0
          TX packets:232 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:33710 (32.9 KiB)  TX bytes:33710 (32.9 KiB)
```
