---
title: "Certbot, DNS, and CloudFlare"
date: 2024-03-30T10:20:37+00:00
draft: false
author: "Bart Prokop"
description: "Building site-to-site VPN with OpenWRT"
tags: ["EdgeRouter", "ZeroTier", "setup"]
---

# Certbot, DNS, and CloudFlare on Arch Linux

## Setup

In this post, I cover how to configure Let's Encrypt DNS challenge with DNS-01 challenge.

```bash
pacman -S certbot
pacman -S certbot-dns-cloudflare
```

Navigate to <https://dash.cloudflare.com/profile/api-tokens> and create API Token.
![API Token](https://assets.prokop.dev/qg/2024/08/CloudFlareApiToken.png "CloudFlare API Token")
Then preserve that token in local file:

```bash
$ vi .secrets/certbot-cloudflare.ini
# Cloudflare API token used by Certbot
dns_cloudflare_api_token = <YOUR_TOKEN_HERE>
```

To avoid seeing `Unsafe permissions on credentials configuration file: /root/.secrets/certbot-cloudflare.ini` when running certbot, execute the following:

```bash
chmod 600 .secrets/certbot-cloudflare.ini
```

Finally try to register and create certificate for one of your domains:

```bash
# certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot-cloudflare.ini
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): bart@prokop.uk

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.4-April-3-2024.pdf. You must agree in
order to register with the ACME server. Do you agree?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing, once your first certificate is successfully issued, to
share your email address with the Electronic Frontier Foundation, a founding
partner of the Let's Encrypt project and the non-profit organization that
develops Certbot? We'd like to send you email about our work encrypting the web,
EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: y
Account registered.
Please enter the domain name(s) you would like on your certificate (comma and/or
space separated) (Enter 'c' to cancel):
```
