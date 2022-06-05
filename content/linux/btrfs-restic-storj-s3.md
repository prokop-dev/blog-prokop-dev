---
title: "Backing BTRFS filesystem with Restic to Storj."
date: 2022-06-05T21:25:11+01:00
draft: false
author: "Bart Prokop"
description: "Backing BTRFS filesystem shapshot with Restic to distributed Storj using S3 gateway"
tags: ["storj", "s3", "btrfs", "restic", "backup", "linux"]
---

I'm using BTRFS on my home server. One of subvolumes is called `media` and is used to store curated collection of personal
and professional photos, movies, books, etc. for whole family. In this post, I will share my experience with backing
the whole `media` NAS share (currently 116 GiB).

# Backup procedure

## Consistency - preparing backup source

First snapshot

```bash
btrfs subvolume snapshot -r /mnt/t20raid/svols/media /mnt/t20raid/snaps/media/2022-05-31
Create a readonly snapshot of '/mnt/t20raid/svols/media' in '/mnt/t20raid/snaps/media/2022-05-31'
```


restic backup to Storj via S3 gateway

Retrieve everything from Storj....

```
export AWS_ACCESS_KEY_ID=****************************
export AWS_SECRET_ACCESS_KEY=*****************************************************
restic -r s3:gateway.storjshare.io/restic init
enter password for new repository:
enter password again:
created restic repository 805b7b695a at s3:gateway.storjshare.io/restic

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```

Here next steps.

```
restic -r s3:gateway.storjshare.io/restic --verbose backup /mnt/t20raid/snaps/media/2022-05-31
open repository
enter password for repository:
repository 805b7b69 opened successfully, password is correct
created new cache in /home/bart/.cache/restic
lock repository
load index files
no parent snapshot found, will read all files
start scan on [/mnt/t20raid/snaps/media/2022-05-31]
start backup on [/mnt/t20raid/snaps/media/2022-05-31]
scan finished in 6.158s: 40136 files, 116.443 GiB
[0:27] 0.13%  152 files 150.443 MiB, total 40136 files 116.443 GiB, 0 errors ETA 5:56:12
/mnt/t20raid/snaps/media/2022-05-31/GooglePhotosLegacy/Asia/2016/IMG_20160323_201925.jpg
/mnt/t20raid/snaps/media/2022-05-31/GooglePhotosLegacy/Asia/2016/IMG_20160323_201935.jpg
```

```
restic -r s3:gateway.storjshare.io/restic --verbose backup /mnt/t20raid/snaps/media/2022-05-31
open repository
enter password for repository:
repository 805b7b69 opened successfully, password is correct
created new cache in /home/bart/.cache/restic
lock repository
load index files
no parent snapshot found, will read all files
start scan on [/mnt/t20raid/snaps/media/2022-05-31]
start backup on [/mnt/t20raid/snaps/media/2022-05-31]
scan finished in 6.158s: 40136 files, 116.443 GiB

Files:       40136 new,     0 changed,     0 unmodified
Dirs:          585 new,     0 changed,     0 unmodified
Data Blobs:  110801 new
Tree Blobs:    531 new
Added to the repo: 109.559 GiB

processed 40136 files, 116.443 GiB in 5:40:16
snapshot 6490b681 saved

-----------------------------------------------------------------------------------------

[bart@t20 snaps]$ restic -r s3:gateway.storjshare.io/restic snapshots
enter password for repository:
repository 805b7b69 opened successfully, password is correct
ID        Time                 Host        Tags        Paths
------------------------------------------------------------------------------------------
6490b681  2022-05-31 18:28:18  t20                     /mnt/t20raid/snaps/media/2022-05-31
------------------------------------------------------------------------------------------
1 snapshots





```

# Summary
