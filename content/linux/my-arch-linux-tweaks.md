---
title: "My Arch Linux Tweaks"
date: 2022-06-25T10:05:59+01:00
draft: false
author: "Bart Prokop"
description: "What I do, when install Arch - my postinstall steps"
tags: ["personal", "blog"]
---

# Switching my shell to zsh

First, zsh usually does not come pre-installed.

```bash
sudo pacman -S zsh
chsh -l
chsh -s /usr/bin/zsh
```

Then after re-logging, I go with "do nothing":

```
This is the Z Shell configuration function for new users,
zsh-newuser-install.
You are seeing this message because you have no zsh startup files
(the files .zshenv, .zprofile, .zshrc, .zlogin in the directory
~).  This function can help you with a few settings that should
make your use of the shell easier.

You can:

(q)  Quit and do nothing.  The function will be run again next time.

(0)  Exit, creating the file ~/.zshrc containing just a comment.
     That will prevent this function being run again.

(1)  Continue to the main menu.

--- Type one of the keys in parentheses ---
```

And, I finally install Oh My Zsh.

```zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
