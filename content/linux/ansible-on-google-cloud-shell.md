---
title: "Ansible on Google Cloud Shell"
date: 2022-11-13T09:17:55Z
draft: false
author: "Bart Prokop"
description: "How to use Ansible with Google Cloud Shell"
tags: ["Ansible", "Google Cloud Shell"]
ShowToc: false
---

The [Google Cloud Shell](https://shell.cloud.google.com/?show=terminal) is fantastic power tool. Unfortunately for me it comes without [Ansible](https://github.com/ansible/ansible) preinstalled.

Google Cloud Shell is basically Debian, so Ansible can be easily added using `apt-get` command.
However it has drawback of any changes to underlying VM being wiped-up frequently as underlying machine is ephemeral one.

# Installation

There is a better way, though. Use `pip`.

```bash
python3 -m pip install --user ansible

Collecting ansible
  Downloading ansible-6.6.0-py3-none-any.whl (42.3 MB)
     |████████████████████████████████| 42.3 MB 214 kB/s
Collecting ansible-core~=2.13.6
  Downloading ansible_core-2.13.6-py3-none-any.whl (2.1 MB)
     |████████████████████████████████| 2.1 MB 40.3 MB/s
Requirement already satisfied: PyYAML>=5.1 in /usr/local/lib/python3.9/dist-packages (from ansible-core~=2.13.6->ansible) (6.0)
Requirement already satisfied: cryptography in /usr/local/lib/python3.9/dist-packages (from ansible-core~=2.13.6->ansible) (38.0.3)
Requirement already satisfied: packaging in /usr/local/lib/python3.9/dist-packages (from ansible-core~=2.13.6->ansible) (21.3)
Collecting resolvelib<0.9.0,>=0.5.3
  Downloading resolvelib-0.8.1-py2.py3-none-any.whl (16 kB)
Requirement already satisfied: jinja2>=3.0.0 in /usr/local/lib/python3.9/dist-packages (from ansible-core~=2.13.6->ansible) (3.1.2)
Requirement already satisfied: MarkupSafe>=2.0 in /usr/local/lib/python3.9/dist-packages (from jinja2>=3.0.0->ansible-core~=2.13.6->ansible) (2.1.1)
Requirement already satisfied: cffi>=1.12 in /usr/local/lib/python3.9/dist-packages (from cryptography->ansible-core~=2.13.6->ansible) (1.15.1)
Requirement already satisfied: pycparser in /usr/local/lib/python3.9/dist-packages (from cffi>=1.12->cryptography->ansible-core~=2.13.6->ansible) (2.21)
Requirement already satisfied: pyparsing!=3.0.5,>=2.0.2 in /usr/local/lib/python3.9/dist-packages (from packaging->ansible-core~=2.13.6->ansible) (3.0.9)
Installing collected packages: resolvelib, ansible-core, ansible
  WARNING: The scripts ansible, ansible-config, ansible-connection, ansible-console, ansible-doc, ansible-galaxy, ansible-inventory, ansible-playbook, ansible-pull and ansible-vault are installed in '/home/prokop_bart/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
```

Please note the warning above. Actually GCS home account contains `.profile` script that would add to `PATH` variable `~/bin` and `~/.local/bin` folders if those are present.
Close and open again Google Cloud Shell and ansible should be permanently available:

```
$ ansible --version

ansible [core 2.13.6]
  config file = None
  configured module search path = ['/home/prokop_bart/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/prokop_bart/.local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/prokop_bart/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/prokop_bart/.local/bin/ansible
  python version = 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
  jinja version = 3.1.2
  libyaml = True
```

```
$ python3 -m pip show ansible
Name: ansible
Version: 6.6.0
Summary: Radically simple IT automation
Home-page: https://ansible.com/
Author: Ansible, Inc.
Author-email: info@ansible.com
License: GPLv3+
Location: /home/prokop_bart/.local/lib/python3.9/site-packages
Requires: ansible-core
Required-by:
```

# Maintenance

Remember to upgrade Ansible from time to time :

```bash
python3 -m pip install --upgrade --user ansible
```

# Usage

Try if Ansible works with some remote host:

```
$ ansible -i ansible-hosts all --list-hosts
  hosts (1):
    ks1.prokop.eu
```

```
$ ansible -i ansible-hosts all -m ping
ks1.prokop.eu | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.10"
    },
    "changed": false,
    "ping": "pong"
}
```
