---
title: "Retrieving PGP key from protonmail using gpg"
date: 2022-06-15T23:08:44+01:00
draft: false
author: "Bart Prokop"
description: "How to retrieve public PGP keys from proton mail using gpg command line."
tags: ["personal", "blog"]
---

It seems that Proton Mail publishes its customers public keys using at leasts two methods:

- WKD (Web Key Distribution)
- Exposes hkps server (host name: api.protonmail.ch)

# Using WKD

Just retrieve public key using by issues the following command:

```bash
$ gpg --locate-keys prokop.bart@pm.me
gpg: key 6C74835C42CEF599: public key "prokop.bart@pm.me <prokop.bart@pm.me>" imported
gpg: Total number processed: 1
gpg:               imported: 1
pub   rsa2048 2018-05-26 [SC]
      49148230F11C0458BD19F45C6C74835C42CEF599
uid           [ unknown] prokop.bart@pm.me <prokop.bart@pm.me>
sub   rsa2048 2018-05-26 [E]
```

Of course it would be good to sign the key and distribute signed one to some public key server.

# Using keyserver / HKPS protocol

You need to specify `api.protonmail.ch` as keyserver for `gpg` command:

``` bash
$ gpg --keyserver hkps://api.protonmail.ch --search-key prokop.bart@proton.me
gpg: data source: https://api.protonmail.ch:443
(1)     prokop.bart@proton.me <prokop.bart@proton.me>
          EDDSA key 99411FBED9D31546, created: 2022-04-14
Keys 1-1 of 1 for "prokop.bart@proton.me".  Enter number(s), N)ext, or Q)uit > 1
gpg: key 99411FBED9D31546: public key "prokop.bart@proton.me <prokop.bart@proton.me>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```

The above command runs an "interactive" search against keyserver and prompts for option that allows to import the desired key.
