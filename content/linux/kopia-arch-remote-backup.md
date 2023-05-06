---
title: "Kopia remote backup"
date: 2022-12-27T00:01:06Z
draft: false
author: "Bart Prokop"
description: "Backing my NAS drive to dedicated box"
tags: ["kopia", "backup", "arch"]
ShowToc: true
---

I lease Kimsufi dedicated server.
It has large enough HDD to use it as offsite backup.
[https://kopia.io/](Kopia) is the new backup tool, I want to try out.

# Prerequisites

## Remote server

I want a new `kopia` regular user account that will allow SFTP access and will be locked otherwise.
The `scponly` pseudo-shell can be used to achieve that.

```bash
sudo pacman -S scponly
sudo useradd -m -s /usr/bin/scponly kopia

sudo mkdir /home/kopia/.ssh
sudo chown kopia:kopia /home/kopia/.ssh
sudo chmod 700 /home/kopia/.ssh
sudo cp .ssh/authorized_keys /home/kopia/.ssh
sudo chown kopia:kopia /home/kopia/.ssh/authorized_keys
```

The above should create new user `kopia` and allow same SSH keys as your ones to facilitate remote SFTP access.
The following `kopia` credentials will be used:

```
--path=repo1
--host=your.host.tld
--username=kopia
--keyfile=~/.ssh/id_ed25519
```





```
➜  ~ ./kopia-0.12.1-linux-x64/kopia repository create sftp --path=repo1 --host=paris.prokop.dev --username=kopia --keyfile=.ssh/id_ed25519 --known-hosts=.ssh/known_hosts
Enter password to create new repository:
Re-enter password for verification:
Initializing repository with:
  block hash:          BLAKE2B-256-128
  encryption:          AES256-GCM-HMAC-SHA256
  splitter:            DYNAMIC-4M-BUZHASH
Connected to repository.

NOTICE: Kopia will check for updates on GitHub every 7 days, starting 24 hours after first use.
To disable this behavior, set environment variable KOPIA_CHECK_FOR_UPDATES=false
Alternatively you can remove the file "/home/bart/.config/kopia/repository.config.update-info.json".

Retention:
  Annual snapshots:                 3   (defined for this target)
  Monthly snapshots:               24   (defined for this target)
  Weekly snapshots:                 4   (defined for this target)
  Daily snapshots:                  7   (defined for this target)
  Hourly snapshots:                48   (defined for this target)
  Latest snapshots:                10   (defined for this target)
  Ignore identical snapshots:   false   (defined for this target)
Compression disabled.

To find more information about default policy run 'kopia policy get'.
To change the policy use 'kopia policy set' command.

NOTE: Kopia will perform quick maintenance of the repository automatically every 1h0m0s
and full maintenance every 24h0m0s when running as bart@t20.

See https://kopia.io/docs/advanced/maintenance/ for more information.

NOTE: To validate that your provider is compatible with Kopia, please run:

$ kopia repository validate-provider

```






```
➜  ~ ./kopia-0.12.1-linux-x64/kopia repository validate-provider
Validating storage capacity and usage
Validating blob list responses
Validating non-existent blob responses
Writing blob (5000000 bytes)
Validating conditional creates...
Validating list responses...
Validating partial reads...
Validating full reads...
Validating metadata...
Running concurrency test for 30s...
All good.
Cleaning up temporary data...
```


https://kopia.io/docs/faqs/#how-do-i-enable-compression

./kopia-0.12.1-linux-x64/kopia snapshot create /mnt/t20raid/svols/media


