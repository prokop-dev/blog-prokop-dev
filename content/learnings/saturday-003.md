---
title: "Saturday, 26th October 2024"
date: 2024-10-26T06:19:19Z
draft: false
author: "Bart Prokop"
description: "Break in the rain"
tags: ["arch", "openwrt", "docker", "linux"]
---

## Getting Docker on Arch Linux

My old Gist is still relevant: https://gist.github.com/bartprokop/15b07ec2502c59cf8020b1541ad57d5c but be comfortable to switch to volumes for data persistence.

```
pacman -S docker
systemctl start docker.service

docker info
```

I will need `genfstab` - https://wiki.archlinux.org/title/Genfstab

```
# mount root BTRFS volume and create Docker dedicated subvolume
mkdir /mnt/sda2
mount /dev/sda2 /mnt/sda2
cd /mnt/sda2
btrfs subvolume create svols/docker

pacman -S arch-install-scripts
mkdir /var/lib/docker
mount -o subvol=/svols/docker /dev/sda2 /var/lib/docker

genfstab  /
/dev/sda2 /var/lib/docker btrfs rw,noatime,ssd,discard=async,space_cache=v2,subvol=/svols/docker 0 0

# Reboot and ensure all filesystems are mounted as expected.

sudo systemctl enable docker.service
Created symlink '/etc/systemd/system/multi-user.target.wants/docker.service' → '/usr/lib/systemd/system/docker.service'.
```

Test if docker is operable:

```zsh
➜  ~ docker run -it --rm archlinux bash -c "echo hello world"
Unable to find image 'archlinux:latest' locally
latest: Pulling from library/archlinux
6de47c16693d: Pull complete
c864e0e7bea0: Pull complete
Digest: sha256:3fb8e79e4037db1536d331dba905e234fa18c8206191458a927013a2d2dafc50
Status: Downloaded newer image for archlinux:latest
hello world
```

Finally do not forget to install Docker compose:

```
pacman -S docker-compose
```

## Dragonfly Mail Agent on Arch Linux

On OpenWrt I'm using `msmtp-mta`, but on Arch servers, the dma seems to be better option.
So let's try to allow servers to use Google Workspace account to send automated emails.

```
resolving dependencies...
looking for conflicting packages...

Packages (1) dma-0.13-4

Total Download Size:   0.04 MiB
Total Installed Size:  0.09 MiB

:: Proceed with installation? [Y/n]
```

Wow, it is tiny ;).

So let's configure `/etc/dma/dma.conf`:

```
SMARTHOST smtp.gmail.com
PORT 465
AUTHPATH /etc/dma/auth.conf
SECURETRANSFER
MASQUERADE notifications@example.com
NULLCLIENT
```

I've also updated relevant Wiki pages here: https://wiki.archlinux.org/title/Dma

## Grafana, Prometheus and OpenWrt

### OpenWRT - node exporter

First I need metric exporter on my routers:

```
opkg update
opkg install prometheus-node-exporter-lua

netstat -tanp | grep 9100
# tcp        0      0 127.0.0.1:9100          0.0.0.0:*               LISTEN      8808/uhttpd
```

The `/etc/config/prometheus-node-exporter-lua` needs to be updated to use `lan` interface.

```
config prometheus-node-exporter-lua 'main'
        option listen_interface 'lan'
        option listen_port '9100'
```

Check what other node exporters you might need:

```
# opkg list | grep prometheus-node
prometheus-node-exporter-lua - 2024.06.16-2 - Provides node metrics as Prometheus scraping endpoint.  This service is a lightweight rewrite in LUA of the offical Prometheus node_exporter.
prometheus-node-exporter-lua-bmx7 - 2024.06.16-2 - Prometheus node exporter (bmx7 links collector)
prometheus-node-exporter-lua-dawn - 2024.06.16-2 - Prometheus node exporter (dawn collector)
prometheus-node-exporter-lua-hostapd_stations - 2024.06.16-2 - Prometheus node exporter (hostapd_stations collector) - Requires a full hostapd / wpad build
prometheus-node-exporter-lua-hostapd_ubus_stations - 2024.06.16-2 - Prometheus node exporter (hostapd_ubus_stations collector)
prometheus-node-exporter-lua-hwmon - 2024.06.16-2 - Prometheus node exporter (hwmon collector)
prometheus-node-exporter-lua-mwan3 - 2024.06.16-2 - Prometheus node exporter (mwan3 collector)
prometheus-node-exporter-lua-nat_traffic - 2024.06.16-2 - Prometheus node exporter (nat_traffic collector)
prometheus-node-exporter-lua-netstat - 2024.06.16-2 - Prometheus node exporter (netstat collector)
prometheus-node-exporter-lua-openwrt - 2024.06.16-2 - Prometheus node exporter (openwrt collector)
prometheus-node-exporter-lua-snmp6 - 2024.06.16-2 - Prometheus node exporter (snmp6 collector)
prometheus-node-exporter-lua-textfile - 2024.06.16-2 - Prometheus node exporter (textfile collector)
prometheus-node-exporter-lua-thermal - 2024.06.16-2 - Prometheus node exporter (thermal collector)
prometheus-node-exporter-lua-ubnt-manager - 2024.06.16-2 - Prometheus node exporter (ubnt-manager collector)
prometheus-node-exporter-lua-uci_dhcp_host - 2024.06.16-2 - Prometheus node exporter (uci_dhcp_host collector)
prometheus-node-exporter-lua-wifi - 2024.06.16-2 - Prometheus node exporter (wifi collector)
prometheus-node-exporter-lua-wifi_stations - 2024.06.16-2 - Prometheus node exporter (wifi_stations collector)
```

### Prometheus on Arch Linux

I will run it on Docker.
Reason is simple, I would like to be able to move the payload across homelab or cloud or SaaS.
So first attempt was just to run basic Prometheus instance:

```
docker run -p 9090:9090 --rm prom/prometheus
```

It went well, but when navigated to http://server_ip:9090, I've got below warning:


```
Warning: Error fetching server time: Detected 247.7739999294281 seconds time difference between your browser and the server. Prometheus relies on accurate time and time drift might cause unexpected query results.
```

Time to fix NTP synchronization on server (`timedatectl set-ntp true` will do), see the end of this post too.

```
➜  ~ cat prometheus.yml
scrape_configs:
  - job_name: OpenWrt
    static_configs:
      - targets: ['192.168.221.1:9100', '192.168.222.1:9100', '192.168.223.1:9100']

remote_write:
  - url: https://prometheus-prod-XX-prod-XX-XXXX-X.grafana.net/api/prom/push
    basic_auth:
      username: 1234567
      password: a...x
```

### Configure Grafana Cloud


## Fixing time on Arch Linux

```
➜  ~ timedatectl
               Local time: Sat 2024-10-26 11:38:06 BST
           Universal time: Sat 2024-10-26 10:38:06 UTC
                 RTC time: Sat 2024-10-26 10:38:06
                Time zone: Europe/Belfast (BST, +0100)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

The probably most advised fix is to run:

```
timedatectl set-ntp true
```

That seems to fix all time issues, I had on one server and also the output of `hwclock --show` is current.
Documentation: https://wiki.archlinux.org/title/Systemd-timesyncd
