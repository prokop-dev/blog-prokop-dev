---
title: "Static Web on Google Cloud"
date: 2023-08-20T30:43:47+01:00
draft: false
author: "Bart Prokop"
description: "How to use Cloud Storage bucket to host a static website"
tags: ["Cloud Storage", "Bucket", "Web"]
---

# The needs

This post describes how I've configured a Cloud Storage bucket to host a static assets (shared) for various websites across my domains. The aim is to have something like `https://assets.prokop.dev` to offer from single URL all assets, I might ever need to handle my WEB stuff.

# Using Google Cloud Storage

The following is everything what was required to create fully operational, CDN powered, backed by object bucket asset distribution facility.

## Create a new Google Cloud Project

I headed to [Google Cloud Console](https://console.cloud.google.com) and created a new project called "Web Static Content".

I had to enable billing for this, even though I expect to get all handled within Free Tier allowance.

## Create Bucket

Navigate to [Cloud Storage](https://console.cloud.google.com/storage/browser) section of Google Cloud Console.

I have created bucked named `assets.prokop.dev`.

![Naming bucket](https://assets.prokop.dev/qg/2023/08/20230820-02.png)

I have selected South Carolina as single region for my data.

![Select location](https://assets.prokop.dev/qg/2023/08/20230820-03.png)

Standard for storage class is most appropriate option for our use case.

![Storage class](https://assets.prokop.dev/qg/2023/08/20230820-04.png)

I've disabled "Enforce public access prevention on this bucket".
Access control is uniform as we want to allow access to all files.

And I've chosen not to use any retention of data.

Then just click on Create.
As my Google account that I use for Google Cloud is managed by Google Workspace and the domain `prokop.dev` seems to be account secondary domain, Google did not prompt for domain validation.


Note that Cloud Storage Always Free quotas apply to usage in US-WEST1, US-CENTRAL1, and US-EAST1 regions.
So, to get first 5 GB per month free, just create bucket in one of above regions.

Then upload few files

Navigate to Permission tab and click "Grant Access".
You will need to add In the New principals field, enter allUsers.

In the Select a role drop down, select the Cloud Storage sub-menu, and click the Storage Object Viewer option.

Check that everything works by trying the public URL to your data:
https://storage.googleapis.com/assets.prokop.dev/index.html

Configure CDN:

https://cloud.google.com/storage/docs/request-endpoints

Add CNAME record
c.storage.googleapis.com

```
$ curl https://c.storage.googleapis.com/qg/2023/08/20230820-02.png -v -H "host: assets.prokop.dev" > /dev/null
*   Trying 142.250.180.16:443...
* Connected to c.storage.googleapis.com (142.250.180.16) port 443 (#0)
* ALPN: offers h2
* ALPN: offers http/1.1
*  CAfile: C:/Program Files/Git/mingw64/ssl/certs/ca-bundle.crt
*  CApath: none
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=*.storage.googleapis.com
*  start date: Jul 31 08:21:23 2023 GMT
*  expire date: Oct 23 08:21:22 2023 GMT
*  subjectAltName: host "c.storage.googleapis.com" matched cert's "*.storage.googleapis.com"
*  issuer: C=US; O=Google Trust Services LLC; CN=GTS CA 1C3
*  SSL certificate verify ok.
* Using HTTP2, server supports multiplexing
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* h2h3 [:method: GET]
* h2h3 [:path: /qg/2023/08/20230820-02.png]
* h2h3 [:scheme: https]
* h2h3 [:authority: assets.prokop.dev]
* h2h3 [user-agent: curl/7.83.0]
* h2h3 [accept: */*]
* Using Stream ID: 1 (easy handle 0x13bc73b02e0)
> GET /qg/2023/08/20230820-02.png HTTP/2
> Host: assets.prokop.dev
> user-agent: curl/7.83.0
> accept: */*
>
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
< HTTP/2 200
< x-guploader-uploadid: ADPycdsluPn2BKndceOHg6gy8Q-PdfLp_wjLazw-E58DtonHjGwrUY9fU_Xp3Na5PzpLm9Lrk6EYiIROPNN8jXFUIhNJRg
< expires: Tue, 22 Aug 2023 00:00:12 GMT
< date: Mon, 21 Aug 2023 23:00:12 GMT
< cache-control: public, max-age=3600
< last-modified: Mon, 21 Aug 2023 22:54:11 GMT
< etag: "fd4b90f8c101c8ad01f8c4c736fa4e6a"
< x-goog-generation: 1692658451719859
< x-goog-metageneration: 1
< x-goog-stored-content-encoding: identity
< x-goog-stored-content-length: 19397
< content-type: image/png
< x-goog-hash: crc32c=wQk65w==
< x-goog-hash: md5=/UuQ+MEByK0B+MTHNvpOag==
< x-goog-storage-class: STANDARD
< accept-ranges: bytes
< content-length: 19397
< server: UploadServer
< alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
<
Warning: Binary output can mess up your terminal. Use "--output -" to tell
Warning: curl to output it to your terminal anyway, or consider "--output
Warning: <FILE>" to save to a file.
* Failure writing output to destination
* Connection #0 to host c.storage.googleapis.com left intact
```


# Resources and references

- https://cloud.google.com/storage/docs/hosting-static-website
