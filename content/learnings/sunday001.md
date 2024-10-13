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

