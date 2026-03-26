---
date: 2026-03-26T00:00:00+08:00
draft: false
title: zsh xdg standard
summary: The ZDOTDIR environment variable in Zsh defines the configuration location, used as a bootstrap to redirect the rest of the shell startup process and align it with the XDG standards
tags:
  - shell
  - notes
categories:
  - toolage
series:
  - implementing-zsh
---


In Zsh, ZDOTDIR is a built-in environment variable that defines the directory where the shell looks for its configuration files. By default, if this variable is not set, Zsh looks in your home directory ($HOME).

Setting this variable uses the shell's native initialisation to change its own "home" base.

## How it Works: The Startup Sequence
To understand why ZDOTDIR works, you have to look at the order in which Zsh loads files. When you open a terminal, Zsh searches for files in this specific order:
1. /etc/zshenv: System-wide environment settings.
2. $HOME/.zshenv: Your personal environment settings. This is the critical step.

Zsh always looks for .zshenv in the user home directory first. Because of this, .zshenv acts as a "bootstrap", or redirector. If you put 
`export ZDOTDIR=$HOME/.config/zsh` inside that file, Zsh immediately updates its internal pointer for the next initialisation stage.

From that moment forward, for the rest of the startup process, Zsh stops looking in $HOME and starts looking in your new directory for the remaining files:
- .zprofile
- .zshrc
- .zlogin
- .zlogout

## What is XDG
XDG (Cross-Desktop Group), now known as freedesktop.org, sets standards for Linux/Unix (MacOS is Unix) environments to improve interoperability. Key standards include the [XDG Base Directory Specification](https://wiki.archlinux.org/title/XDG_Base_Directory), which organises config, data, and cache files (e.g., `~/.config`, `~/.local`)

**Why Use XDG?**
- **Consistency:** Applications behave consistently across different desktop environments.
- **Organization:** Prevents the home directory from being filled with hidden configuration files (dotfiles).
- **Portability:** Makes it easier to move user configurations between different Linux installations.

## Why use it for Zsh?
1. XDG Base Directory Compliance
There is a widely used standard in Linux and macOS called the XDG Base Directory Specification. It suggests that applications should keep their configuration files in `~/.config`, their data in `~/.local/share`, and their cache in `~/.cache`. By default, Zsh is "old school" and dumps everything into your home root. But it is flexible and give us ZDOTDIR that we can use to make Zsh respect the XDG standard.

2. Decluttering $HOME
One of the goals in the XDG standard is all application configurations under a central subdirectory.  Without ZDOTDIR, your home directory becomes cluttered with hidden files: .zshrc, .zsh_history, .zcompdump, .zsh_sessions, etc. Moving these into a sub-folder keeps your primary workspace clean.

3. Portability and Version Control
Centralising improves management. If you keep your *dotfiles* in a Git repository, it is much easier to manage them if they are all in one folder (like `~/.config/zsh`) rather than being scattered across the root of your home directory. You can simply symlink the one .zshenv file, and the rest of your complex configuration follows automatically, this is what frameworks like *stow* achieve.

4. Environment Isolation
It allows you to maintain different shell environments. For example, if you were testing a completely new setup, you could point ZDOTDIR to a temporary folder to see how it behaves without risking your "production" .zshrc.

> [!Summary]
> ZDOTDIR is the shell's configuration root. Setting it in .zshenv is the "bootstrap" method used to tell Zsh how to continue initilising". Aligning with XDG brings many advantages.