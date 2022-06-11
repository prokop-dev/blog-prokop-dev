---
title: "Marginal Trust"
date: 2022-06-11T22:23:14+01:00
draft: false
author: "Bart Prokop"
description: "How GPG trusts unknown keys"
tags: ["personal", "blog"]
---

# Marginal Trust

```
downloading required keys...
:: Import PGP key 0F65C7D881506130, "Maxime Gauduin <alucryd@archlinux.org>"? [Y/n]
(38/38) checking package integrity                                     


Import PGP key 0F65C7D881506130, "Maxime Gauduin <alucryd@archlinux.org>"

  237  curl "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x0e8b644079f599dfc1ddc3973348882f6ac6a4c2" > 0x6AC6A4C2.gpg
  241  curl "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0xab19265e5d7d20687d303246ba1dfb64fff979e7" > 0xFFF979E7.gpg
  246  curl "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x91ffe0700e80619ceb73235ca88e23e377514e00" > 0x77514E00.gpg
  248  curl "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0xd8afdda07a5b6edfa7d8ccdad6d055f927843f1c" > 0x27843F1C.gpg
  250  curl "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2ac0a42efb0b5cbc7a0402ed4dc95b6d7be9892e" > 0x7BE9892E.gpg
  251  curl "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x159f3a43aeb246c5746c033814bc4f30b3b92eba" > 0xB3B92EBA.gpg

prokop_bart@cloudshell:~/code/gpg-keys/trusted/arch-master$ gpg --import *
gpg: key D6D055F927843F1C: 6 signatures not checked due to missing keys
gpg: key D6D055F927843F1C: public key "Levente Polyak (Arch Linux Master Key) <anthraxx@master-key.archlinux.org>" imported
gpg: key 3348882F6AC6A4C2: 24 signatures not checked due to missing keys
gpg: key 3348882F6AC6A4C2: public key "Pierre Schmitz (Arch Linux Master Key) <pierre@master-key.archlinux.org>" imported
gpg: key A88E23E377514E00: 17 signatures not checked due to missing keys
gpg: key A88E23E377514E00: public key "Florian Pritz (Arch Linux Master Key) <florian@master-key.archlinux.org>" imported
gpg: key 4DC95B6D7BE9892E: public key "David Runge (Arch Linux Master Key) <dvzrv@master-key.archlinux.org>" imported
gpg: key 14BC4F30B3B92EBA: public key "Giancarlo Razzolini (Arch Linux Master Key) <grazzolini@master-key.archlinux.org>" imported
gpg: key BA1DFB64FFF979E7: 19 signatures not checked due to missing keys
gpg: key BA1DFB64FFF979E7: public key "Allan McRae (Arch Linux Master Key) <allan@master-key.archlinux.org>" imported
gpg: Total number processed: 6
gpg:               imported: 6
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   2  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:   2  signed:   0  trust: 2-, 0q, 0n, 0m, 0f, 0u



prokop_bart@cloudshell:~/code/gpg-keys/trusted/arch-master$ gpg --sign-key D6D055F927843F1C

pub  rsa4096/D6D055F927843F1C
     created: 2018-11-08  expires: never       usage: SC
     trust: unknown       validity: unknown
sub  rsa4096/DEDF3FE3104A16F6
     created: 2018-11-08  expires: never       usage: A
sub  rsa4096/FEB12332C13054E7
     created: 2018-11-08  expires: never       usage: E
[ unknown] (1). Levente Polyak (Arch Linux Master Key) <anthraxx@master-key.archlinux.org>


pub  rsa4096/D6D055F927843F1C
     created: 2018-11-08  expires: never       usage: SC
     trust: unknown       validity: unknown
 Primary key fingerprint: D8AF DDA0 7A5B 6EDF A7D8  CCDA D6D0 55F9 2784 3F1C

     Levente Polyak (Arch Linux Master Key) <anthraxx@master-key.archlinux.org>

Are you sure that you want to sign this key with your
key "Bart Prokop (gcs) <bart@prokop.dev>" (B61CDFDAC50F3186)

Really sign? (y/N) y


---------

prokop_bart@cloudshell:~/code/gpg-keys/trusted$ gpg --import 0F65C7D881506130
gpg: key 0F65C7D881506130: 3 signatures not checked due to missing keys
gpg: key 0F65C7D881506130: public key "Maxime Gauduin <alucryd@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   7  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:   7  signed:   1  trust: 7-, 0q, 0n, 0m, 0f, 0u


----

prokop_bart@cloudshell:~/code/gpg-keys/trusted$ gpg --list-signatures 95220BE99CE6FF778AE0DC670F65C7D881506130
pub   rsa4096 2022-01-30 [SC]
      95220BE99CE6FF778AE0DC670F65C7D881506130
uid           [ unknown] Maxime Gauduin <alucryd@gmail.com>
sig 3        0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
sig          AFF5D95098BC6FF5 2022-01-30  [User ID not found]
uid           [ unknown] Maxime Gauduin <alucryd@alucryd.xyz>
sig 3        0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
sig          AFF5D95098BC6FF5 2022-01-30  [User ID not found]
uid           [  undef ] Maxime Gauduin <alucryd@archlinux.org>
sig 3        0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
sig          AFF5D95098BC6FF5 2022-01-30  [User ID not found]
sig          4DC95B6D7BE9892E 2022-02-24  David Runge (Arch Linux Master Key) <dvzrv@master-key.archlinux.org>
sig          A88E23E377514E00 2022-03-06  Florian Pritz (Arch Linux Master Key) <florian@master-key.archlinux.org>
sig          3348882F6AC6A4C2 2022-04-24  Pierre Schmitz (Arch Linux Master Key) <pierre@master-key.archlinux.org>
sig          D6D055F927843F1C 2022-04-25  Levente Polyak (Arch Linux Master Key) <anthraxx@master-key.archlinux.org>
sub   rsa4096 2022-01-30 [A]
sig          0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
sub   rsa4096 2022-01-30 [E]
sig          0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>




gpg --export-ownertrust
# List of assigned trustvalues, created Sat 11 Jun 2022 05:18:06 PM UTC
# (Use "gpg --import-ownertrust" to restore them)
D9E1B3CEB81FCDFD20264CF0B61CDFDAC50F3186:6:
D8AFDDA07A5B6EDFA7D8CCDAD6D055F927843F1C:4:
0E8B644079F599DFC1DDC3973348882F6AC6A4C2:4:
91FFE0700E80619CEB73235CA88E23E377514E00:4:
2AC0A42EFB0B5CBC7A0402ED4DC95B6D7BE9892E:4:
AB19265E5D7D20687D303246BA1DFB64FFF979E7:4:


prokop_bart@cloudshell:~/code/gpg-keys/trusted$ gpg --import 0F65C7D881506130
gpg: key 0F65C7D881506130: 3 signatures not checked due to missing keys
gpg: key 0F65C7D881506130: public key "Maxime Gauduin <alucryd@gmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   8  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:   8  signed:   1  trust: 4-, 0q, 0n, 4m, 0f, 0u
gpg: depth: 2  valid:   1  signed:   0  trust: 1-, 0q, 0n, 0m, 0f, 0u




prokop_bart@cloudshell:~/code/gpg-keys$ gpg --list-sig
--list-sig         --list-signatures  --list-sigs
prokop_bart@cloudshell:~/code/gpg-keys$ gpg --list-signatures 95220BE99CE6FF778AE0DC670F65C7D881506130
pub   rsa4096 2022-01-30 [SC]
      95220BE99CE6FF778AE0DC670F65C7D881506130
uid           [ unknown] Maxime Gauduin <alucryd@gmail.com>
sig 3        0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
sig          AFF5D95098BC6FF5 2022-01-30  [User ID not found]
uid           [ unknown] Maxime Gauduin <alucryd@alucryd.xyz>
sig 3        0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
sig          AFF5D95098BC6FF5 2022-01-30  [User ID not found]
uid           [  full  ] Maxime Gauduin <alucryd@archlinux.org>
sig 3        0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
sig          AFF5D95098BC6FF5 2022-01-30  [User ID not found]
sig          4DC95B6D7BE9892E 2022-02-24  David Runge (Arch Linux Master Key) <dvzrv@master-key.archlinux.org>
sig          A88E23E377514E00 2022-03-06  Florian Pritz (Arch Linux Master Key) <florian@master-key.archlinux.org>
sig          3348882F6AC6A4C2 2022-04-24  Pierre Schmitz (Arch Linux Master Key) <pierre@master-key.archlinux.org>
sig          D6D055F927843F1C 2022-04-25  Levente Polyak (Arch Linux Master Key) <anthraxx@master-key.archlinux.org>
sub   rsa4096 2022-01-30 [A]
sig          0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
sub   rsa4096 2022-01-30 [E]
sig          0F65C7D881506130 2022-01-30  Maxime Gauduin <alucryd@gmail.com>
```

That is a piece of technology that has no comparable in today's to be 2020's cryptography with trying to announce PGP being dead in water.