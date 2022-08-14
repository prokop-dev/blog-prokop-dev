---
title: "Btrfs on Luks"
date: 2022-06-05T10:40:08+01:00
draft: false
author: "Bart Prokop"
description: "Luks and BTRFS subjective best practices"
tags: ["personal", "blog"]
---

# Further maintenance

## Scrubbing

Regularly scrub your drive...

```bash
# btrfs scrub start /mnt/t20raid

# btrfs scrub status /mnt/t20raid
# watch 'btrfs scrub status /mnt/t20raid'
```

This is example "work in progress":

```
Every 2.0s: btrfs scrub status /mnt/t20raid                                                                                                t20: Sun Aug 14 14:39:52 2022

UUID:             9f65175e-ec44-4fbf-a28d-cc10ef1a799e
Scrub started:    Sun Aug 14 14:32:07 2022
Status:           running
Duration:         0:07:40
Time left:        0:09:38
ETA:              Sun Aug 14 14:49:30 2022
Total to scrub:   302.91GiB
Bytes scrubbed:   134.13GiB  (44.28%)
Rate:             298.59MiB/s
Error summary:    no errors found
```

Note that it can take all your CPU. Also with the amount of data being read/write...
You really want to have ECC memory for BTRFS that runs on LUKS.

![top command result](https://f000.backblazeb2.com/file/cdn-prokop-dev/blog/2022/08/blog-20220814-1.png)

# Resources

- Relevant [gist](https://gist.github.com/bartprokop/594d7147cb61a04b38caf96d70caf571)
