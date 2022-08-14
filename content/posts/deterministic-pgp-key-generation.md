---
title: "Deterministic PGP key generation"
date: 2022-08-13T18:20:23+01:00
draft: false
author: "Bart Prokop"
description: "How to generate GPG key with passphrase"
tags: ["personal", "blog"]
---

# Passphrase

I wrote small utility to generate BIP-39 compatible passphrases. One needs Java runtime to run it.

```bash
# Note - when downloading, check for latest version
$ wget https://repo1.maven.org/maven2/dev/prokop/crypto/crypto-bips/1.0.2/crypto-bips-1.0.2-standalone.jar

$ java -jar crypto-bips-1.0.2-standalone.jar bip39 -l 512
trim mango orphan craft together topic unique merry autumn little economy actress brief dog deny syrup turkey mother slab detail crucial doll water rug original trash course bid option assume pulse witness upon steak ranch whisper great beach enhance delay junior couple twelve bargain rib mass hazard panther
```

The above uses 512 bits for seed and then convert it to a pass phrase. Source code available on [GitHub](https://github.com/bartprokop/crypto-bips). This is of course overkill.

# Passphrase to GPG Key

## Getting key generator

I'm using [Chris Wellons's](https://nullprogram.com/blog/2019/07/10/) excellent [passphrase2pgp](https://github.com/skeeto/passphrase2pgp).
I have installed it just by using ```go``` command:

```zsh
➜  ~ go install nullprogram.com/x/passphrase2pgp@latest
go: downloading nullprogram.com/x/passphrase2pgp v1.2.0
go: downloading golang.org/x/crypto v0.0.0-20200423211502-4bdfaf469ed5
go: downloading nullprogram.com/x/optparse v1.0.0
go: downloading golang.org/x/sys v0.0.0-20190412213103-97732733099d```
```

## Key generation

Let's assume that our organization was founded on 1st December 2020 and you want to create key-pair for privacy@sample.org.
The epoch time for you will be: 1606780800. The following command will generate you a PGP key from passphrase:

```bash
$ passphrase2pgp "Sample Ltd <privacy@sample.org>" -t 1606780800 -s | gpg --import
passphrase:
passphrase (repeat):
gpg: key F33DB40CC339BBBC: public key "Sample Ltd <privacy@sample.org>" imported
gpg: key F33DB40CC339BBBC: secret key imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1

$ gpg -K
------------------------------------------------
sec   ed25519 2020-12-01 [SC]
      D268E7D8CFEB40B7665BCA61F33DB40CC339BBBC
uid           [ unknown] Sample Ltd <privacy@sample.org>
ssb   cv25519 2020-12-01 [E]
```

Note, above example uses empty passphrase, so you can reproduce steps above.

The key can be easily reproduced, just by reusing the same phrase.
This time we need to enter passphrase only once, as we will additionally validate the match of key fingerprint.

```bash
$ passphrase2pgp -a -u "Sample Ltd <privacy@sample.org>" -t 1606780800 -c F33DB40CC339BBBC
passphrase:
-----BEGIN PGP PRIVATE KEY BLOCK-----

xVgEX8WHgBYJKwYBBAHaRw8BAQdAswRWuGCZms8g+jlNGkQP960Qo8iCK6mZkFSM
10i2zhMAAQDCb17n/IONrugT5txBtEpgL4ZtpFGc5O+4pBhNRnRiqRGZzR9TYW1w
bGUgTHRkIDxwcml2YWN5QHNhbXBsZS5vcmc+wmEEExYIABMFAl/Fh4AJEPM9tAzD
Obu8AhsDAACR7wEAiqPFpF3CK2V3ZuC5LM1I+jXoNcYf58ekOVzWiPCkMNgA/1kF
5DTpTSkCUF3tOilswT9X/rTKOSEXU1nJTxPF/x8O
=fQtx
-----END PGP PRIVATE KEY BLOCK-----

$ gpg -K
------------------------------------------------
sec   ed25519 2020-12-01 [SC]
      D268E7D8CFEB40B7665BCA61F33DB40CC339BBBC
uid           [ unknown] Sample Ltd <privacy@sample.org>
```

Note: this time we did not generate encryption sub-key.
The private key block can be inspected [here](https://cirw.in/gpg-decoder/#-----BEGIN%20PGP%20PRIVATE%20KEY%20BLOCK-----xVgEX8WHgBYJKwYBBAHaRw8BAQdAswRWuGCZms8g+jlNGkQP960Qo8iCK6mZkFSM10i2zhMAAQDCb17n/IONrugT5txBtEpgL4ZtpFGc5O+4pBhNRnRiqRGZzR9TYW1wbGUgTHRkIDxwcml2YWN5QHNhbXBsZS5vcmc+wmEEExYIABMFAl/Fh4AJEPM9tAzDObu8AhsDAACR7wEAiqPFpF3CK2V3ZuC5LM1I+jXoNcYf58ekOVzWiPCkMNgA/1kF5DTpTSkCUF3tOilswT9X/rTKOSEXU1nJTxPF/x8O=fQtx-----END%20PGP%20PRIVATE%20KEY%20BLOCK-----).

# Useful links

1. [PGP / GPG Decoder](https://cirw.in/gpg-decoder/)
2. [Mailvelope Key Server](https://keys.mailvelope.com/)
3. [Ubuntu Keyserver](https://keyserver.ubuntu.com/)
4. [Epoch Time Converter](https://www.epochconverter.com/)
