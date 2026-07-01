---
date: 2026-04-01T00:00:00+08:00
draft: false
title: A XDG compliant Config
summary: As I continued refining my Zsh environment, I realised that performance wasn't my biggest challenge, organisation was. This article follows my journey to a modular, XDG-compliant configuration, where separating responsibilities and building a clear architecture laid the foundation for a faster, more maintainable shell.
tags:
  - shell
  - notes
categories:
  - toolage
series:
  - implementing-zsh
---
## Designing a Fast, XDG-Compliant Zsh Configuration

After understanding how Zsh starts and the role of each startup file, I realised my configuration still had another problem. It worked, but it wasn't particularly organised.

Like many people, I'd gradually accumulated configuration over time. A few aliases here, some exports there, plugin configuration mixed in with shell options, and before long my `.zshrc` had become the place where everything lived.

There was nothing technically wrong with it, but every time I wanted to make a change I found myself scrolling through a growing wall of configuration trying to remember where I'd put something months earlier.

That was the point where I decided it was time to stop thinking about configuration files and start thinking about architecture.

## Moving Beyond a Monolithic `.zshrc`

My first instinct was to split everything into smaller files.

Not because the shell cared, but because I did.

Instead of one large configuration file, I wanted related settings grouped together so I could quickly find and maintain them.

My configuration now looks something like this:

```text
~/.config/zsh/
├── .zshrc
├── .zprofile
├── .zlogin
├── zsh-aliases
├── zsh-exports
├── zsh-functions
├── zsh-vim-mode
└── plugins/
```

Rather than containing all of my configuration, `.zshrc` has become the conductor. Its primary job is to load the modules that make up my shell.

That simple change made my configuration much easier to navigate.

## Why I Chose an XDG Layout

While reorganising everything, I decided to adopt the XDG Base Directory Specification.

Instead of scattering application configuration throughout my home directory, most modern applications can keep their configuration inside a dedicated `.config` directory.

For Zsh, that meant moving almost everything into:

```text
~/.config/zsh
```

There was one exception.

My `~/.zshenv` file still lives in my home directory because Zsh always looks there first. Its job is simply to tell Zsh where the rest of the configuration lives.

```bash
export ZDOTDIR="$HOME/.config/zsh"
```

Once that's set, every other startup file is loaded from my new configuration directory.

It's a tiny file, but arguably the most important one in the entire setup.

## Separating Configuration by Responsibility

Once everything lived under one directory, it became much easier to separate responsibilities.

Instead of asking, _"Where should I put this?"_, I started asking, _"What is this responsible for?"_

That mindset led me to create a handful of focused configuration files.

### Aliases

My aliases live in their own file.

This includes quality-of-life improvements like replacing `ls` with `eza`, safer versions of `cp`, `mv`, and `rm`, shortcuts for Neovim, and frequently used administrative commands.

Keeping them together means I can quickly scan and update the commands I use every day without digging through unrelated configuration.

### Environment Variables

Initially, almost all of my exports lived together in one file.

As I learned more about the Zsh startup lifecycle, I realised that not every environment variable belongs in the same place.

Global variables such as:
- `PATH`
- `XDG_*`
- `EDITOR`
- `GNUPGHOME`
are required by every shell, including scripts.

Interactive settings such as:
- `MANPAGER`
- `BAT_THEME`
- `FZF_DEFAULT_OPTS`
only matter when I'm actively using the terminal.

That distinction eventually led me to split my exports between `.zshenv` and `.zshrc`, allowing scripts to avoid loading unnecessary configuration.

## Functions Instead of Complexity

One area I hadn't expected to enjoy was writing my own shell functions.

Rather than relying on a full plugin manager, I started building a small collection of helper functions that did exactly what I needed.
- Functions for loading plugins.
- Functions for sourcing configuration files.
- Functions for updating plugins.

Nothing particularly complicated, but each one has a single responsibility.

As the functions evolved, I found myself replacing external commands with native Zsh features, making them both simpler and faster.

That ended up becoming a surprisingly enjoyable part of the project 🤓.

## Keeping Interactive Features Together

Another benefit of modular configuration was being able to isolate interactive behaviour.

My Vim key bindings, shell history configuration, prompt customisation, plugin loading, and completion settings all live together because they all contribute to the interactive shell experience.

When I'm experimenting with command-line editing or trying a new plugin, I know exactly where to look.

## It's Easier with Small Files

One unexpected benefit of this approach is that each file now has a clear purpose.

🔧 If I'm working on aliases, I open my aliases file.

🔧 If I'm experimenting with Vim mode, I know exactly which file contains those settings.

🧨 If I break something, I've dramatically reduced the search space.

Instead of trying to understand one large configuration file, I'm working with a collection of much smaller building blocks.  This makes experimentation feel much less risky.

## Organisation Before Optimisation

Looking back, I think I was originally chasing performance. I wanted a faster shell. What I needed was better organisation.

Once the configuration became modular, optimisation naturally flowed to becaming easier because every component now had a clearly defined role.

**It's much easier to improve a system when you understand its structure.**

## Looking Ahead

With my configuration now organised into logical modules, I was finally in a position to start refining the code itself.

In the next article, I'll look at the shell functions I've written to manage plugins and configuration, and how replacing external commands with native Zsh features helped create a lightweight, maintainable plugin management system without relying on a full plugin manager.