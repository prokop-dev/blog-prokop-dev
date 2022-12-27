---
title: "SSHd Server on Android"
date: 2022-12-04T21:51:46Z
draft: false
author: "Bart Prokop"
description: "Mounting Android phone as local Linux directory"
tags: ["personal", "blog"]
ShowToc: false
---

Description how I transfer files from my Android phone to Linux server.
This will use FUSE to mount Android storage as linux directory.
Example solution is to backup all photos from Android phone.

## Android setup

Please install [https://play.google.com/store/apps/details?id=org.galexander.sshd](SimpleSSHD) from Android Play store.
This is port of [https://en.wikipedia.org/wiki/Dropbear_(software)](Dropbear) to Android.

## Linux Setup

Use ssh from terminal to log-in to your Android phone.

```
# username does not matter, use password shown in Android console for the very first login
ssh bart@192.168.123.321 -p 2222
```

Then use the below to add your SSH public key to `authorized_keys` file.

```
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHbcHsshkeaayM0QOP09P7HevjXz73THSVPjYkMoENeS" >> authorized_keys
```

Ensure that your "home directory" looks similar to this and `cat` public key(s) to ensure subsequent connection can use SSH keys.

```
:/data/user/0/org.galexander.sshd/files $ ls -la
total 26
drwxrwx--x 2 u0_a303 u0_a303 3452 2022-12-04 21:40 .
drwx------ 6 u0_a303 u0_a303 3452 2022-11-24 03:53 ..
-rw------- 1 u0_a303 u0_a303  405 2022-12-04 21:42 authorized_keys
-rw------- 1 u0_a303 u0_a303 1144 2022-12-04 21:57 dropbear.err
-rw------- 1 u0_a303 u0_a303   60 2022-12-04 21:35 dropbear.err.old
-rw------- 1 u0_a303 u0_a303    6 2022-12-04 21:36 dropbear.pid
-rw------- 1 u0_a303 u0_a303   83 2022-12-04 21:37 dropbear_ed25519_host_key
```

Finally install `sshfs` on the computer, you want use to connect to your phone:

```zsh
sudo pacman -S sshfs
```

## Accessing Android phone

Below commands will mount Android phone as directory using FUSE (the path `/storage/emulated/0` works for me):

```bash
mkdir pixel
sshfs -p 2222 bart@192.168.2.222:/storage/emulated/0 ./pixel
```

Files on my phone are now accessible to Linux machine as mounted filesystem:

```bash
$ cd pixel

$ ls -la
total 52
drwxrws--- 1 10223 1023 3452 Aug 28 10:49 Alarms
drwxrws--x 1  1023 1023 3452 Aug 28 10:49 Android
drwxrws--- 1 10223 1023 3452 Aug 28 10:49 Audiobooks
drwxrws--- 1 10223 1023 3452 Nov  6 18:43 DCIM
drwxrws--- 1 10223 1023 3452 Aug 28 10:49 Documents
drwxrws--- 1 10223 1023 3452 Nov 20 13:35 Download
drwxrws--- 1 10223 1023 3452 Nov 29 13:11 Movies
drwxrws--- 1 10223 1023 3452 Aug 28 10:49 Music
drwxrws--- 1 10223 1023 3452 Aug 28 10:49 Notifications
drwxrws--- 1 10223 1023 3452 Oct 31 14:03 Pictures
drwxrws--- 1 10223 1023 3452 Aug 28 10:49 Podcasts
drwxrws--- 1 10223 1023 3452 Aug 28 10:49 Recordings
drwxrws--- 1 10223 1023 3452 Aug 28 10:49 Ringtones

$ sha256sum DCIM/GoPro-Exports/GOPR0...56963.JPG
2d93...b7894  DCIM/GoPro-Exports/GOPR0...56963.JPG
```

To un-mount SSHFS filesystem, just use umount:

```zsh
umount pixel
```
