---
title: "Certbot, DNS, and CloudFlare"
date: 2024-08-03T22:20:37+00:00
draft: false
author: "Bart Prokop"
description: "Wildcard certificate for my domains"
tags: ["CloudFlare", "Certificate", "setup"]
---

# Certbot on Arch Linux

In this post, I cover how to configure Let's Encrypt DNS challenge with DNS-01 challenge.

## Setup

Install the following packages (certbot and CloudFlare plug-in):

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

## Testing

Finally try to run the following and "register your account" with Let's Encrypt:

```bash
# certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot-cloudflare.ini
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): b***@prokop.uk

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

I've cancelled here.

## Request wildcard certificate

The following command will create certificate with a SAN extension, i.e. certificate for both the apex domain and the wildcard domain.
For example this command will issue single certificate for `prokop.uk` and `*.prokop.uk`:

```bash
certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot-cloudflare.ini -d 'prokop.uk' -d '*.prokop.uk'
```

Resulting certificate:
![certificate](https://assets.prokop.dev/qg/2024/08/WildcardCertificate.png)

And SAN extensions are as expected:
![SAN](https://assets.prokop.dev/qg/2024/08/WildcardCertificateDetails.png)

## Test if renewal will be successful

There is dry-run option to validate renewal of certificates:

```bash
# certbot renew --dry-run
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/prokop.dev.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Account registered.
Simulating renewal of an existing certificate for prokop.dev and *.prokop.dev
Waiting 10 seconds for DNS changes to propagate

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/prokop.uk.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for prokop.uk and *.prokop.uk
Waiting 10 seconds for DNS changes to propagate

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/prokop.dev/fullchain.pem (success)
  /etc/letsencrypt/live/prokop.uk/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

## Automatic renewal

Issue the following to enable certificates renewal prior to 30 days to expiry:

```bash
# systemctl enable --now certbot-renew.time
```

## Resources

- <https://letsencrypt.org/docs/challenge-types/>
- <https://wiki.archlinux.org/title/Certbot>
- <https://certbot.eff.org/instructions>
