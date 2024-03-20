---
title: "Email security with Google Workspace"
date: 2023-07-23T08:23:01+01:00
draft: false
author: "Bart Prokop"
description: "What I learned about setting email with Google Workspaces"
tags: ["security", "blog", "email"]
---

# TL;DR; What records needs to be added to DNS?

```
v=spf1 include:_spf.google.com -all
v=DKIM1; k=rsa; p=MIIBI...6lwIDAQAB
google._domainkey v=DMARC1; p=reject; rua=mailto:e...c@dmarc-reports.cloudflare.net
default._bimi v=BIMI1; l=https://fizjoterapia.uk/bimi.svg
```

# SPF

Sender Policy Framework (SPF) is an email authentication method which ensures the sending mail server is authorized to originate mail from the email sender's domain.
It is absolutely necessary to have SPF entry in DNS zone - otherwise your email will end-up in SPAM folder or even will be discarded.

```
v=spf1 include:_spf.google.com -all
```

A receiving mail server can use the IP addresses in your SPF record to authenticate the sending mail server of a received email with your domain as sender. The accompanying policy can be used to take appropriate action if authentication fails. Having more than one SPF record in the same domain is not valid and will lead to a test failure.

SPF policy should be sufficiently strict.

Test explanation:
We check if the syntax of your SPF record is correct and if it contains a sufficiently strict policy i.e. ~all (softfail) or -all (fail) in order to prevent abuse by phishers and spammers. To be effective against abuse of your domain, a liberal policy i.e. +all (pass) or ?all (neutral) is insufficient. If all is missing and a redirect modifier is not present in your SPF record, the default is ?all.

We also follow include and redirect for valid SPF records. In addition, we check that your SPF record does not exceed the allowed number of 10 DNS lookups making it ineffective. Each of the following SPF terms counts as a DNS lookup: redirect, include, a, mx, ptr and exist. While the SPF terms all, ip4, ip6 and exp do not count as DNS lookups.

Note that if the include or redirect domain consists of macros, we will not follow them as we do not have the necessary information from an actual mail or mail server connection to expand those macros. This means that we do not evaluate the outcome of macros. Moreover, we do not count DNS lookups caused by macros to determine whether the allowed number of DNS lookups has been exceeded.

For sending domains, using ~all (softfail) instead of -all (fail) is usually preferred. This is because when SPF authentication fails with an SPF fail, a receiving mail server may already block the connection without evaluating the DKIM signature and DMARC policy. This can lead to falsely blocked mails (e.g. when mails are automatically forwarded by the receiving mail server to another receiving mail server). Note that a triggered SPF softfail still ensures that a strict DMARC policy protects your domain if DKIM authentication also fails.

For no-mail domains see the good practice on explanation of the "DMARC policy" subtest.

# DKIM

Verdict:
Your domain supports DKIM records.

Test explanation:
We check if your domain supports DKIM records. A receiving mail server can use the public key in your DKIM record to validate the signature in an email with a user from your domain as sender and determine its authenticity.

Currently we are not able to query and evaluate the public key in your DKIM record, because we would need the DKIM selector (that should be in the mails you send) to do so.
To pass this test we expect your name server to answer NOERROR to our query for _domainkey.example.nl. When _domainkey.example.nl is an 'empty non- terminal' some name servers that are not conformant with the standard RFC2308 incorrectly answer NXDOMAIN instead of NOERROR. This makes it impossible for us to detect support for DKIM records.
For this test we assume 'strict alignment' which is conformant with DMARC. The given domain is considered to be the sender domain in the mail body domain (From:) and this must be identical to the DKIM domain (d=) and the SPF domain (envelope sender, i.e. return-path that shows up in MAIL FROM).
For a "good practice on domains without mail" see the explanation of the "DMARC policy" subtest.

# DMARC

Verdict:
Your DMARC policy is sufficiently strict.

Test explanation:
We check if the syntax of your DMARC record is correct and if it contains a sufficiently strict policy (p=quarantine or p=reject) in order to prevent abuse of your domain by phishers and spammers. Even without a strict policy, DMARC can be useful to get more insight in legitimate and illegitimate outbound mail flows through DMARC reports. However to be effective against abuse of your domain, a liberal policy (p=none) is insufficient.

We also check whether the mail addresses under rua= and ruf= are valid. In case they contain an external mail address we check whether the external domain is authorised to receive DMARC reports. Make sure that the DMARC authorisation record on the external domain contains at least "v=DMARC1;".

Good practice for domains without mail:

Non-sending: Use the most strict DMARC policy (p=reject) and SPF policy (-all) for your domain that you do not use for sending mail in order to prevent abuse ('spoofing'). Note that DKIM is not necessary in this case.
Non-receiving: For more information on "Null MX" and "No MX" see the explanation of the "STARTTLS available" subtest.
Below you will find two basic example DNS configurations for domain names that are not used to send and receive mail.

A. Single domain without A or AAAA record

example.nl. TXT "v=spf1 -all"
*.example.nl. TXT "v=spf1 -all"
_dmarc.example.nl. TXT "v=DMARC1; p=reject;"
 

B. Single no-mail domain with A or AAAA record

example.nl. AAAA 2a00:d78:0:712:94:198:159:35
example.nl. A 94.198.159.35
example.nl. MX 0 .
example.nl. TXT "v=spf1 -all"
*.example.nl. TXT "v=spf1 -all"
www.example.nl. CNAME example.nl
_dmarc.example.nl. TXT "v=DMARC1; p=reject;"

Verdict:
Your domain has a DMARC record.

Technical details:
DMARC record	Found on
v=DMARC1; p=reject; rua=mailto:e...c@dmarc-reports.cloudflare.net	prokop.ovh
Test explanation:
We check if DMARC is available for your domain. A receiving mail server may use your DMARC policy to evaluate how to handle a mail with your domain as sender that could not be authenticated with both DKIM and SPF, and it may use your mail address from the DMARC record to provide feedback reports on the authentication to you. Having more than one DMARC record in the same domain is not valid and will lead to a test failure. Note: DMARC requires the SPF domain (envelope sender, i.e. return-path that shows up in MAIL FROM) and the DKIM domain (d=) to align with the mail body domain (From:).