---
title: "Hurricane Electric Filtered Tunnel"
date: 2022-11-21T23:11:21Z
draft: false
author: "Bart Prokop"
description: "Starting new, starting fresh"
tags: ["personal", "blog"]
---

```
➜  ~ sudo nmap -p - -6 paris.prokop.dev
Starting Nmap 7.93 ( https://nmap.org ) at 2022-11-21 23:04 GMT
Nmap scan report for paris.prokop.dev (2001:41d0:e:514::1)
Host is up (0.062s latency).
Other addresses for paris.prokop.dev (not scanned): 5.196.72.20
Not shown: 65518 closed tcp ports (reset)
PORT     STATE    SERVICE
22/tcp   open     ssh
25/tcp   filtered smtp
5355/tcp filtered llmnr
6660/tcp filtered unknown
6661/tcp filtered unknown
6662/tcp filtered radmind
6663/tcp filtered unknown
6664/tcp filtered unknown
6665/tcp filtered irc
6666/tcp filtered irc
6667/tcp filtered irc
6668/tcp filtered irc
6669/tcp filtered irc
6670/tcp filtered irc
6697/tcp filtered ircs-u
7000/tcp filtered afs3-fileserver
9999/tcp filtered abyss

Nmap done: 1 IP address (1 host up) scanned in 119.41 seconds
➜  ~ sudo nmap -p - -4 paris.prokop.dev
Starting Nmap 7.93 ( https://nmap.org ) at 2022-11-21 23:08 GMT
Nmap scan report for paris.prokop.dev (5.196.72.20)
Host is up (0.027s latency).
Other addresses for paris.prokop.dev (not scanned): 2001:41d0:e:514::1
Not shown: 65533 closed tcp ports (reset)
PORT     STATE    SERVICE
22/tcp   open     ssh
5355/tcp filtered llmnr

Nmap done: 1 IP address (1 host up) scanned in 11.84 seconds
➜  ~
```
