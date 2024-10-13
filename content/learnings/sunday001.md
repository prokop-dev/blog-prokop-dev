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
Committing now, validate this...

## OpenWrt hostname

I'm in the process of aligning names of all my devices with my naming scheme.
It is unlikely that I will have more than one router per location, so routers are simply named `rt-<LOC>`.
That corresponds with following entry in `/etc/config/system`:


rtr-mro
rtr-bel
rtr-ave


router-mro
rtr-bel
rtr-ave