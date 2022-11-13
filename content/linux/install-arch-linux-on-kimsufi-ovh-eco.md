---
title: "Install Arch Linux on Kimsufi Ovh Eco"
date: 2022-11-13T02:56:56Z
draft: false
author: "Bart Prokop"
description: "Description how easiest way to install Arch Linux on Kimsufi KS1 aka OVH Eco dedicated server"
tags: ["Linux", "OVH", "hosting", "Cloud"]
---

Install "Cloud Ready" images are available here: https://geo.mirror.pkgbuild.com/images/latest/

## Post install

Update local server fingerprint... If you seeing the below when trying to connect:

```
$ ssh arch@SERVER_IP
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
```

Use the following to remove old server instance SSH key pinning:

```
ssh-keygen -f ~/.ssh/known_hosts -R "SERVER_IP"
```

## Security

Server listens on the following ports:

```
sudo ss -tunlp
Netid       State         Recv-Q        Send-Q                  Local Address:Port               Peer Address:Port       Process
udp         UNCONN        0             0                             0.0.0.0:5355                    0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=11))
udp         UNCONN        0             0                          127.0.0.54:53                      0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=19))
udp         UNCONN        0             0                       127.0.0.53%lo:53                      0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=17))
udp         UNCONN        0             0                    5.196.72.20%eth0:68                      0.0.0.0:*           users:(("systemd-network",pid=351,fd=18))
udp         UNCONN        0             0                                [::]:5355                       [::]:*           users:(("systemd-resolve",pid=315,fd=13))
tcp         LISTEN        0             128                           0.0.0.0:22                      0.0.0.0:*           users:(("sshd",pid=399,fd=3))
tcp         LISTEN        0             4096                          0.0.0.0:5355                    0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=12))
tcp         LISTEN        0             4096                       127.0.0.54:53                      0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=20))
tcp         LISTEN        0             4096                    127.0.0.53%lo:53                      0.0.0.0:*           users:(("systemd-resolve",pid=315,fd=18))
tcp         LISTEN        0             128                              [::]:22                         [::]:*           users:(("sshd",pid=399,fd=4))
tcp         LISTEN        0             4096                             [::]:5355                       [::]:*           users:(("systemd-resolve",pid=315,fd=14))
```

Installed packages

```bash
$ pacman -Q
acl 2.3.1-2
archlinux-keyring 20221110-1
argon2 20190702-4
attr 2.5.1-2
audit 3.0.8-1
base 3-1
bash 5.1.016-1
binutils 2.39-3
brotli 1.0.9-8
btrfs-progs 6.0.1-1
bzip2 1.0.8-4
ca-certificates 20220905-1
ca-certificates-mozilla 3.85-1
ca-certificates-utils 20220905-1
cloud-guest-utils 0.33-1
cloud-init 22.3.4-1
coreutils 9.1-3
cryptsetup 2.5.0-4
curl 7.86.0-3
dbus 1.14.4-1
device-mapper 2.03.17-1
dhclient 4.4.3.P1-1
diffutils 3.8-1
dnssec-anchors 20190629-3
e2fsprogs 1.46.5-4
expat 2.5.0-1
file 5.43-1
filesystem 2022.10.18-1
findutils 4.9.0-1
gawk 5.2.0-3
gcc-libs 12.2.0-1
gdbm 1.23-1
gettext 0.21.1-2
glib2 2.74.1-1
glibc 2.36-6
gmp 6.2.1-2
gnupg 2.2.40-1
gnutls 3.7.8-4
gpgme 1.18.0-1
grep 3.8-2
grub 2:2.06.r334.g340377470-1
gzip 1.12-1
hwdata 0.364-1
iana-etc 20221107-1
icu 72.1-1
iproute2 6.0.0-1
iptables 1:1.8.8-2
iputils 20211215-1
jansson 2.14-2
json-c 0.16-1
kbd 2.5.1-1
keyutils 1.6.3-1
kmod 30-3
krb5 1.20-3
ldns 1.8.3-2
less 1:608-1
libarchive 3.6.1-5
libassuan 2.5.5-1
libbpf 1.0.1-1
libcap 2.66-1
libcap-ng 0.8.3-1
libedit 20210910_3.1-1
libelf 0.187-2
libevent 2.1.12-4
libffi 3.4.4-1
libgcrypt 1.10.1-1
libgpg-error 1.46-1
libidn2 2.3.4-3
libksba 1.6.2-1
libldap 2.6.3-2
libmnl 1.0.5-1
libnetfilter_conntrack 1.0.9-1
libnfnetlink 1.0.2-1
libnftnl 1.2.3-1
libnghttp2 1.50.0-1
libnl 3.7.0-1
libnsl 2.0.0-2
libp11-kit 0.24.1-1
libpcap 1.10.1-2
libpsl 0.21.1-3
libsasl 2.1.28-3
libseccomp 2.5.4-1
libsecret 0.20.5-2
libssh2 1.10.0-3
libsysprof-capture 3.46.0-1
libtasn1 4.19.0-1
libtirpc 1.3.3-1
libunistring 1.1-2
libverto 0.3.2-4
libxcrypt 4.4.30-1
libxml2 2.10.3-2
libyaml 0.2.5-1
licenses 20220125-1
linux 6.0.8.arch1-1
linux-api-headers 5.18.15-1
lz4 1:1.9.4-1
lzo 2.10-3
mkinitcpio 32-2
mkinitcpio-busybox 1.35.0-1
mpfr 4.1.0.p13-3
ncurses 6.3-3
netplan 0.105-1
nettle 3.8.1-1
npth 1.6-3
openssh 9.1p1-3
openssl 3.0.7-2
p11-kit 0.24.1-1
pacman 6.0.2-5
pacman-mirrorlist 20221016-1
pam 1.5.2-1
pambase 20221020-1
pciutils 3.8.0-2
pcre 8.45-3
pcre2 10.40-3
pinentry 1.2.1-1
popt 1.19-1
procps-ng 3.3.17-1
psmisc 23.5-1
python 3.10.8-3
python-attrs 22.1.0-1
python-cffi 1.15.1-1
python-chardet 5.0.0-1
python-configobj 5.0.6.r110.g3e2f4cc-3
python-cryptography 38.0.3-1
python-idna 3.4-1
python-jinja 1:3.1.2-2
python-jsonpatch 1.32-3
python-jsonpointer 2.3-1
python-jsonschema 4.17.0-1
python-markupsafe 2.1.1-1
python-netifaces 0.11.0-3
python-oauthlib 3.2.2-1
python-ply 3.11-10
python-pycparser 2.21-3
python-pyrsistent 0.19.2-1
python-pyserial 3.5-4
python-requests 2.28.1-1
python-six 1.16.0-6
python-typing_extensions 4.4.0-1
python-urllib3 1.26.12-1
python-yaml 6.0-1
readline 8.2.001-1
reflector 2021.11-5
run-parts 5.5-1
sed 4.8-1
shadow 4.12.3-2
sqlite 3.39.4-1
sudo 1.9.12.p1-1
systemd 252.1-1
systemd-libs 252.1-1
systemd-sysvcompat 252.1-1
tar 1.34-1
tpm2-tss 3.2.0-3
tzdata 2022f-1
util-linux 2.38.1-1
util-linux-libs 2.38.1-1
xz 5.2.7-1
zlib 1:1.2.13-1
zsh 5.9-1
zstd 1.5.2-7
```

# Tailoring your install

## Upgrade all

First step is always to perform full system upgrade.

```bash
sudo pacman -Syu
sudo sync
sudo reboot
```

## Add your own user, shell and SSH key

First let's have zsh installed

```bash
sudo pacman -S zsh
cat /etc/shells
```

Add new local user

```bash
sudo useradd -m -s /usr/bin/zsh bart
sudo pacman -S git # required for OMZ installation
sudo su - bart
```

```zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
rm .zshrc.pre-oh-my-zsh
cp .zshrc .zshrc.backup-2022-11-13
```