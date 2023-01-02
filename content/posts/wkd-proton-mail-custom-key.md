---
title: "WKD setup for your domain"
date: 2022-06-13T08:03:01+01:00
draft: false
author: "Bart Prokop"
description: "How to publish your PGP key on your webserver"
tags: ["security", "blog"]
---

I have recently tried and was keen to see that Proton Mail supports WKD.

```bash
$ gpg --locate-key user@protonmail.com
gpg: key 4DE32C2A10A7EBC2: public key "user@protonmail.com <user@protonmail.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
pub   ed25519 2021-10-13 [SC]
      67731B189D0908618DF665144DE32C2A10A7EBC2
uid           [ unknown] user@protonmail.com <user@protonmail.com>
sub   cv25519 2021-10-13 [E]
```

So here is quick instruction how to setup WKD for any email (i.e. not hosted by Proton Mail).

First you need to create empty file relative to your webserver ROOT.

```bash
/srv/www/fizjoterapia.uk$ mkdir -p .well-known/openpgpkey
/srv/www/fizjoterapia.uk$ touch .well-known/openpgpkey/policy
```

The `policy` file acts as a marker for clients to confirm that WKD files exists on this server.

The WKD is designed to facilitate end-to-end encrypted email communication.
So GPG keys are looked by email address.
It is obvious to look for keys at the HTTPS server running on apex email domain.
The file with GPG key is saved under path `hu\<SHA1 of email address encoded using the Z-Base-32>`.
`hu` stands for "user hash".
Get your hash by running:

```bash
$ gpg --with-wkd-hash -K privacy@fizjoterapia.uk
sec   ed25519 2019-05-17 [SC]
      220C7BCD6A8D9CF1EC8307859F45CF0D77C0B76D
uid           [ultimate] Physio Effect JB Ltd (NI661516) <privacy@fizjoterapia.uk>
              15siaihjsf4kyfkzxrqe7r5gqqzr5f39@fizjoterapia.uk
ssb   cv25519 2019-05-17 [E]
```

And save binary form of public key inside `.well-known/openpgpkey/hu` directory (note the lack of the usual --armour flag):

```bash
mkdir -p .well-known/openpgpkey/hu
gpg --export privacy@fizjoterapia.uk > .well-known/openpgpkey/hu/15siaihjsf4kyfkzxrqe7r5gqqzr5f39
```

I have found [https://metacode.biz/openpgp/web-key-directory](WKD checker) by running some Google search.
Here are my results and I've confirmed that ProtonMail sends now encrypted emails to WKD enabled email.
