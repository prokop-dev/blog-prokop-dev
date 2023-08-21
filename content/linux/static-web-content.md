---
title: "Static Web on Google Cloud"
date: 2023-08-20T30:43:47Z
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

"Standard" for storage class is the most appropriate option for our use case.

![Storage class](https://assets.prokop.dev/qg/2023/08/20230820-04.png)

I've disabled "Enforce public access prevention on this bucket".
Access control is uniform as we want to allow access to all files.

![Access Control](https://assets.prokop.dev/qg/2023/08/20230820-05.png)

And I've chosen not to use any retention of data.

![Storage class](https://assets.prokop.dev/qg/2023/08/20230820-06.png)

Then just clicked "Create".

As my Google account that I use for Google Cloud is managed by Google Workspace and the domain `prokop.dev` seems to be account secondary domain, Google did not prompt for domain validation.

Note that Cloud Storage Always Free quotas apply to usage in `US-WEST1`, `US-CENTRAL1`, and `US-EAST1` regions.
So, to get first 5 GB per month free, just create bucket in one of above regions.

Finally, I've upload few files.

![Files in Bucket](https://assets.prokop.dev/qg/2023/08/20230820-01.png)

Navigate to Permission tab and click "Grant Access".
You will need to add In the New principals field, enter `allUsers`.

In the Select a role drop down, select the Cloud Storage sub-menu, and click the Storage Object Viewer option.

![Viewer Role](https://assets.prokop.dev/qg/2023/08/20230820-07.png)

Confirm that you want to make access to bucket public:

![Public Access](https://assets.prokop.dev/qg/2023/08/20230820-08.png)

Check that everything works by trying the public URL to your data:
https://storage.googleapis.com/assets.prokop.dev/index.html

## Configure CDN

```
$ curl http://c.storage.googleapis.com/style.css -v -H "host: assets.prokop.dev"

> GET /style.css HTTP/1.1
> Host: assets.prokop.dev
> User-Agent: curl/7.83.0
> Accept: */*
>

< HTTP/1.1 200 OK
< X-GUploader-UploadID: ADPycdtmn2NkzHCjlqVAEthSS0a-2be67QuIStVgAcTcODbEFIaeKtsffXmBmNogIKkXoiuoxVsKWqc167c1I8n5gsGHqw
< x-goog-generation: 1692656828987613
< x-goog-metageneration: 1
< x-goog-stored-content-encoding: identity
< x-goog-stored-content-length: 3652
< x-goog-hash: crc32c=SfZ7qg==
< x-goog-hash: md5=gKrqkiiDXLVw+Fntq/B5Ng==
< x-goog-storage-class: STANDARD
< Accept-Ranges: bytes
< Content-Length: 3652
< Server: UploadServer
< Date: Mon, 21 Aug 2023 22:48:02 GMT
< Expires: Mon, 21 Aug 2023 23:48:02 GMT
< Cache-Control: public, max-age=3600
< Last-Modified: Mon, 21 Aug 2023 22:27:08 GMT
< ETag: "80aaea9228835cb570f859edabf07936"
< Content-Type: text/css
< Age: 2815
```


Add CNAME record
c.storage.googleapis.com

![Viewer Role](https://assets.prokop.dev/qg/2023/08/20230820-09.png)

After:

```
$ curl https://assets.prokop.dev/style.css -v > /dev/null

> GET /style.css HTTP/2
> Host: assets.prokop.dev
> user-agent: curl/7.83.0
> accept: */*
>

< HTTP/2 200
< date: Mon, 21 Aug 2023 23:39:04 GMT
< content-type: text/css
< content-length: 3652
< x-guploader-uploadid: ADPycdtmn2NkzHCjlqVAEthSS0a-2be67QuIStVgAcTcODbEFIaeKtsffXmBmNogIKkXoiuoxVsKWqc167c1I8n5gsGHqw
< x-goog-generation: 1692656828987613
< x-goog-metageneration: 1
< x-goog-stored-content-encoding: identity
< x-goog-stored-content-length: 3652
< x-goog-hash: crc32c=SfZ7qg==
< x-goog-hash: md5=gKrqkiiDXLVw+Fntq/B5Ng==
< x-goog-storage-class: STANDARD
< expires: Mon, 21 Aug 2023 23:48:02 GMT
< cache-control: public, max-age=14400
< last-modified: Mon, 21 Aug 2023 22:27:08 GMT
< etag: "80aaea9228835cb570f859edabf07936"
< cf-cache-status: MISS
< accept-ranges: bytes
< report-to: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report\/v3?s=Q9AGp2BU4qhhXjSNqET%2BODPdIam5YRgYZob%2F8FWJ6EE3HyHKKHLJwGGqS%2FmKBrgYYm5OMCtoJZPOBVdQXkvrR2Uc0XR0eskxDRNFLNc%2BtPR7BO7%2FCz%2F6kcHXSsCsRGQKHWx1ZQ%3D%3D"}],"group":"cf-nel","max_age":604800}
< nel: {"success_fraction":0,"report_to":"cf-nel","max_age":604800}
< strict-transport-security: max-age=31536000; includeSubDomains; preload
< x-content-type-options: nosniff
< server: cloudflare
< cf-ray: 7fa6b69a1a980763-MAN
< alt-svc: h3=":443"; ma=86400
```

# Resources and references

- https://cloud.google.com/storage/docs/hosting-static-website
- https://cloud.google.com/storage/docs/request-endpoints
