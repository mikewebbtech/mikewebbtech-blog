---
date: 2026-03-29T00:00:00+08:00
draft: false
title: zsh startup lifecycle
summary: As I continue refining my Zsh environment, I discovered that understanding the startup lifecycle is far more valuable than blindly tweaking configuration files. This article explores how Zsh loads its configuration, the purpose of each startup file, and the design principles that now guide every change I make to my shell.
tags:
  - shell
  - notes
categories:
  - toolage
series:
  - implementing-zsh
---
## Understanding the Zsh Startup Lifecycle

Over the past few weeks I've been slowly refining my Zsh configuration. What started as a simple exercise in tidying up a few dotfiles quickly turned into a much deeper dive into how Zsh actually starts, loads configuration, and manages different types of shell sessions.

One of the biggest lessons I've learned is that optimising Zsh isn't really about making it faster, at least not at first. It's about understanding *when* things happen. Once you understand the startup sequence, decisions about where configuration belongs become surprisingly logical.

Rather than treating Zsh as "one big `.zshrc` file", I've started thinking of it as a series of stages, each with a specific responsibility.

## The Zsh Startup Sequence

Whenever Zsh starts, it doesn't immediately jump into loading your prompt and aliases. Instead, it follows a well-defined sequence of configuration files.

```
~/.zshenv
    ↓
~/.config/zsh/.zprofile
    ↓
~/.config/zsh/.zshrc
    ↓
~/.config/zsh/.zlogin
```

Not every shell reads every file, though. Which files are loaded depends on the type of shell being started.

|File|Login Shell|Interactive Shell|Non-Interactive Shell|
|---|:-:|:-:|:-:|
|`.zshenv`|✓|✓|✓|
|`.zprofile`|✓|✓|✗|
|`.zshrc`|✓|✓|✗|
|`.zlogin`|✓|✓|✗|

This simple table completely changed the way I organise my configuration.

## Understanding Shell Types

There are three shell types that matter.

### Non-Interactive Shells

These are shells that execute scripts.

For example:

```bash
zsh backup.sh
```

There is no prompt, no aliases, and no command history. The shell simply executes the script and exits.

Because these shells are often used by automation, they should start as quickly as possible. Every unnecessary command executed here slows down scripts, cron jobs, and automation.

### Interactive Shells

These are the shells most of us spend our day working in.

When I open a new Kitty tab or split a terminal window, I'm starting an interactive shell. This is where I expect to have:
- command history
- aliases
- plugins
- syntax highlighting
- autosuggestions
- prompts
- Vim mode

Everything that improves the interactive experience belongs here.

### Login Shells

Login shells are a little different.

These usually appear when I first log into a system, connect via SSH, or start a new login session. They represent the beginning of a session rather than just another terminal window.

This makes them the ideal place for tasks that only need to happen once per login.

## What Each Configuration File Should Do

Once I understood the startup sequence, the purpose of each configuration file became much clearer.

### `.zshenv` Environment

This file is read by **every** instance of Zsh.

That makes it the home for global environment variables that scripts and interactive shells both rely on.

Examples include:
- `PATH`
- `XDG_*` directories
- `EDITOR`
- `GNUPGHOME`
- `WGETRC`

One particularly important variable also lives here:

```bash
export ZDOTDIR="$HOME/.config/zsh"
```

This tells Zsh where to find the rest of my configuration.

Interestingly, this creates a bit of a catch-22. If `ZDOTDIR` isn't defined inside the default `~/.zshenv`, Zsh has no idea that my configuration has been moved into `~/.config/zsh`. It simply keeps looking in my home directory instead.

That means `~/.zshenv` is the one configuration file that still needs to live in my home directory.

### `.zprofile` Session Setup

This file is only loaded for login shells.

Rather than filling it with general configuration, I now think of it as session initialisation.

If I ever need to:
- start an SSH agent
- initialise GPG
- configure login-specific environment variables

this is where that work belongs.

At the moment, my own `.zprofile` is intentionally quite small, and that's perfectly fine.

### `.zshrc` — The Interactive Shell

This is where most of my configuration lives.

Everything related to the shell experience belongs here.

That includes:
- history configuration
- aliases
- shell options
- plugins
- completions
- prompt configuration
- Vim mode
- FZF configuration
- command line enhancements

These features only make sense when I'm actively sitting at a terminal.

### `.zlogin` Finishing Touches

Finally, `.zlogin` runs after everything else has been configured.

I think of this as the welcome mat.

It's a good place for things like:

- running Fastfetch
- automatically attaching to tmux
- displaying login information

These are tasks that should happen once per login rather than every time I open another terminal tab.

## The Mental Model That Changed Everything

The biggest shift for me wasn't learning what each file does.

It was changing how I think about them.

Instead of memorising filenames, I think in terms of who does what and in what order.
- **`.zshenv` configures the environment.**
- **`.zprofile` prepares the session.**
- **`.zshrc` builds the interactive shell.**
- **`.zlogin` finishes the login experience.**

Once I had that sorted, reorganising my configuration became much easier. Every new setting naturally found its proper home.

## Looking Ahead

Understanding the startup lifecycle has given me a much stronger foundation for improving my shell. Instead of randomly moving configuration between files, I now understand why each setting belongs where it does.

In the next article, I'll build on this foundation by reorganising my entire Zsh configuration into an XDG-compliant structure. I'll explain how I separated aliases, functions, plugins, exports, and Vim mode into a clean, modular layout that's much easier to maintain and extend over time.
