---
title: "Sunday, 13th October 2024"
date: 2024-10-13T13:03:45Z
draft: false
author: "Bart Prokop"
description: "Lazy Sunday experiments"
tags: ["Linux", "openwrt", "monitoring"]
---

## Few nice command lines

### How to check processor you have on your Linux box

There is `lscpu` command, but in case it is not availabe, the following will do the trick:

```zsh
cat /proc/cpuinfo | grep 'model name' | head -n 1
model name      : Intel(R) Atom(TM) CPU  E3827  @ 1.74GHz
```

### Changes in last git commit

This is mega usefull to show last commit changes:

```zsh
git diff HEAD^ HEAD
```

## Fixing this blog on CloudFlare pages

I updated the PaperMod theme and got CloudFlare build failing with:

```
13:13:34.465	WARN 2024/10/13 12:13:34 The "twitter_simple" shortcode will soon require two named parameters: user and id. See "/opt/buildhome/repo/content/demo/rich-content.md:24:1"
13:13:34.817	ERROR 2024/10/13 12:13:34 render of "home" failed: "/opt/buildhome/repo/themes/PaperMod/layouts/_default/rss.xml:59:17": execute of template failed: template: _default/rss.xml:59:17: executing "_default/rss.xml" at <site>: can't evaluate field LanguageCode in type *langs.Language
13:13:34.817	ERROR 2024/10/13 12:13:34 render of "section" failed: "/opt/buildhome/repo/themes/PaperMod/layouts/_default/rss.xml:59:17": execute of template failed: template: _default/rss.xml:59:17: executing "_default/rss.xml" at <site>: can't evaluate field LanguageCode in type *langs.Language
13:13:34.817	ERROR 2024/10/13 12:13:34 render of "section" failed: "/opt/buildhome/repo/themes/PaperMod/layouts/_default/rss.xml:59:17": execute of template failed: template: _default/rss.xml:59:17: executing "_default/rss.xml" at <site>: can't evaluate field LanguageCode in type *langs.Language
13:13:34.818	ERROR 2024/10/13 12:13:34 render of "section" failed: "/opt/buildhome/repo/themes/PaperMod/layouts/_default/rss.xml:59:17": execute of template failed: template: _default/rss.xml:59:17: executing "_default/rss.xml" at <site>: can't evaluate field LanguageCode in type *langs.Language
13:13:34.818	Error: Error building site: failed to render pages: render of "section" failed: "/opt/buildhome/repo/themes/PaperMod/layouts/_default/rss.xml:59:17": execute of template failed: template: _default/rss.xml:59:17: executing "_default/rss.xml" at <site>: can't evaluate field LanguageCode in type *langs.Language
13:13:34.818	Total in 439 ms
13:13:34.827	Failed: build command exited with code: 255
13:13:49.008	Failed: error occurred while running build command
```

The fix was simple, update to `HUGO_VERSION` veriable from `0.96.0` to `0.123.4`.
Committing now, validate this... Failed, now with:

```
ERROR The "twitter_simple" shortcode requires two named parameters: user and id. See "/opt/buildhome/repo/content/demo/rich-content.md:24:1"
```

Interestingly, I should have pay more attention to previous failure logs...
It seems, I have some old "PaperMod demo" page using old "twitter_simple" shortcode.
Just updated it to show one of my (dormant) Twitter account post.

The fix:
```
## Twitter Simple Shortcode
-twitter_simple 1085870671291310081
+twitter user="bartprokop" id="1445337420967391235"
```

It works now!

## OpenWrt hostname

I'm in the process of aligning names of all my devices with my naming scheme.
It is unlikely that I will have more than one router per location, so routers are simply named `rt-<LOC>`.
That corresponds with following entry in `/etc/config/system`:

```
config system
        option hostname 'rt-LOC'
        option description 'Aguas Verdes Router'
```

After restart, the prompt is much more friendlier:

```
$ ssh root@rt-LOC.prokop.dev

BusyBox v1.36.1 (2024-03-22 22:09:42 UTC) built-in shell (ash)
  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
 OpenWrt 23.05.3, r23809-234f1a2efa
 -----------------------------------------------------
root@rt-LOC:~# 
```

## Grafana for home needs

Grafana Cloud might be really useful for monitoring smart home and home lab.
Register here: [Grafana Cloud](https://grafana.com/auth/sign-in).

## Getting MTA on OpenWRT

Install `msmtp` package with sendmail symlink:

```
# opkg update
# opkg install msmtp-mta
Installing msmtp-mta (1.8.25-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.5/packages/mipsel_24kc/packages/msmtp-mta_1.8.25-1_mipsel_24kc.ipk
Installing libgmp10 (6.2.1-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.5/packages/mipsel_24kc/base/libgmp10_6.2.1-1_mipsel_24kc.ipk
Installing libnettle8 (3.9.1-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.5/packages/mipsel_24kc/base/libnettle8_3.9.1-1_mipsel_24kc.ipk
Installing libgnutls (3.8.3-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.5/packages/mipsel_24kc/packages/libgnutls_3.8.3-1_mipsel_24kc.ipk
Installing msmtp (1.8.25-1) to root...
Downloading https://downloads.openwrt.org/releases/23.05.5/packages/mipsel_24kc/packages/msmtp_1.8.25-1_mipsel_24kc.ipk
Configuring libgmp10.
Configuring libnettle8.
Configuring libgnutls.
Configuring msmtp.
Configuring msmtp-mta.
```

I have created new user Hermes Mercury <hermes.mercury@prokop.tld> on my Google Workspace account.
Then I logged in and enabled 2FA.
If you want to skip phone/secondary email verfication, add Google Authenticator prior to enabling 2FA.

Now it is time to update `/etc/msmtprc`.

```
account default

host smtp.gmail.com

# Use TLS on port 465. On this port, TLS starts without STARTTLS.
port 465
auth on
tls on
tls_starttls off

from hermes.mercury@prokop.tld
user hermes.mercury@prokop.tld
password <APP password>

syslog LOG_MAIL
```

Then test it:

```bash
echo -e "Subject: Test mail\n\nThis is a test \"message\"." | sendmail "<bart@example.com>"
```

