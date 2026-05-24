---
date: 2026-05-24T00:00:00+08:00
draft: false
title: My custom vim mode in zsh
summary: This article describes how I navigate and edit in the Zsh command line using my custom Vi-mode configuration. This setup transforms the Zsh prompt into a mini-Vim editor, complete with mode-specific cursors and custom productivity macros.
tags:
  - opensource
  - shell
categories:
  - toolage
series:
  - implementing-zsh
---


> ## What Is vi-mode
> {.one}
**`vi-mode`** in a terminal shell (like Zsh or Bash) replaces the default command-line editing behaviour with the modal editing system used by the **Vim** text editor.

Instead of working in the command line like a simple text box, `vi-mode` turns it into working like it is a mini Vim buffer.

> ## Why Use It?
> {.three}

### Total Efficiency for Vim Users

I already use Vim or Neovim for coding, and my muscle memory is tuned to specific keystrokes. Without `vi-mode`, trying to move around a long terminal command forces me to reach for the arrow keys or use Emacs shortcuts (like `Ctrl-A` to go to the beginning of the line). With `vi-mode`, my hands never leave the home row.

### Speed with Long or Complex Commands

Standard terminal editing is slow—you hold down the left arrow key to fix a typo ten words back. In `vi-mode` (Normal mode), you can do that instantly:
- `0` — Jump to the absolute beginning of the command.
- `$` — Jump to the very end of the command.
- `fa` — Jump straight forward to the next letter "a".
- `ciw` — **"Change Inner Word"** (Deletes the current word and drops you back into Insert mode to type the replacement).
If your are not familiar with using vi/vim/neovim all this sounds confusing, don't worry, understanding vim motions is something that time and by using vim, I have been using vim for around 20 years, but not at *power user* levels and every time I use it, learn and try something new.  Checkout the popular [Vim Cheat Sheet](https://vim.rtorr.com) 

### Power Editing via the Buffer Shortcut

In my configuration file, there is a shortcut bound to `Ctrl-e`. If you type a massive, multi-line loop or an intricate network command and realise it is getting too messy to track on a single line, hitting `Ctrl-e` instantly launches a full instance of Neovim containing the command. I can edit it using your full editor setup, save and exit (`:wq` or `:x`), and Zsh will immediately run the edited script.

### Visual Mode Selections

I can press `v` while in normal mode to launch a visual selection directly on the command line, letting me highlight, yank (copy) to system clipboard, or delete precise blocks of text just like I would inside neovim.

> ## The How it works
> {.two}

### My Configuration file

I configure my vi-mode settings outside of my main .zshrc file. As I have covered previously, I have moved my zsh config files to ~/.config/zsh/ and my.zshrc file configures my base zsh environment and I do a `zsh_add_file` call to all other configuration settings that live outside in their own config file.

For ease of management, my zsh vi-mode configuration lives in `zsh-vim-mode`

```bash

# ~/.config/zsh/zsh-vim-mode

# Enable Vim command-line editing. 
# Activates 'vi' mode instead of the default Emacs-style keybindings.
bindkey -v

# Set the key timeout to 10 milliseconds (1 = 0.1 deciseconds).
# This removes the noticeable delay when pressing 'Esc' to switch from Insert to Normal mode.
export KEYTIMEOUT=1

# --- Keep your custom keys working in vi-mode ---

# -M viins: Applies the binding specifically to 'vi insert' mode.
# -s: Treats the second argument as a literal string sequence of keystrokes.
# '^M': Represents the 'Enter/Return' key, causing the command to execute immediately.

# Ctrl-n: Opens the fzf fuzzy finder and passes the selected file directly into Neovim
bindkey -M viins -s '^n' 'nvim $(fzf)^M'

# Ctrl-v: Opens a blank Neovim instance directly from the command line
bindkey -M viins -s '^v' 'nvim^M'

# Ctrl-z: Quickly executes the 'zi' command (typically an interactive cd/zoxide shortcut)
bindkey -M viins -s '^z' 'zi^M'

# Fix the 'Delete' key behavior so it works as expected across modes.
# '^[[P' is the terminal escape sequence generated when you press the physical Delete key.
bindkey -M viins '^[[P' delete-char # Fixes Delete key in insert mode
bindkey -M vicmd '^[[P' delete-char # Fixes Delete key in normal/command mode

# History navigation using Ctrl-k and Ctrl-j.
# 'up-line-or-beginning-search' matches the text you've already typed against your history.
bindkey -M viins '^k' up-line-or-beginning-search # Search history up in insert mode
bindkey -M viins '^j' down-line-or-beginning-search # Search history down in insert mode
bindkey -M vicmd '^k' up-line-or-beginning-search # Search history up in normal mode
bindkey -M vicmd '^j' down-line-or-beginning-search # Search history down in normal mode

# Optional traditional Ctrl-p fallback for upward history searching
bindkey -M viins '^p' up-line-or-beginning-search
bindkey -M vicmd '^p' up-line-or-beginning-search

# Use Vim-like navigation keys inside the Zsh tab-completion menu.
# -M menuselect: Targets the specific keymap active when browsing completions.
bindkey -M menuselect '^h' vi-backward-char					# Ctrl-h: Move cursor left
bindkey -M menuselect '^j' vi-down-line-or-history  # Ctrl-j: Move cursor up
bindkey -M menuselect '^k' vi-up-line-or-history 		# Ctrl-K: Move cursor down
bindkey -M menuselect '^l' vi-forward-char					# Ctrl-d: Move cursor right

bindkey -M menuselect '^[[Z' vi-up-line-or-history # Shift-Tab: Move backward through menu items

# Ensure that Backspace ('^?') safely deletes characters backward in insert mode
bindkey -v '^?' backward-delete-char

# Allow editing the current command line inside a full Vim buffer.
# Great for long, complex scripts or multi-line commands.
autoload edit-command-line; zle -N edit-command-line # Load and register the widget
# bindkey '^e' edit-command-line # (Commented out global binding)
bindkey -M viins '^e' edit-command-line # Ctrl-e triggers full Vim editing from insert mode
bindkey -M vicmd '^e' edit-command-line # Ctrl-e triggers full Vim editing from normal mode

# Automatically change the cursor shape depending on the current Vi mode.
# '\e[1 q' tells the terminal emulator to render a blinking solid block.
# '\e[5 q' tells the terminal emulator to render a blinking vertical bar (beam).
function zle-keymap-select () {
    case $KEYMAP in
        vicmd) echo -ne '\e[1 q';;      # Normal mode -> Change to block cursor
        viins|main) echo -ne '\e[5 q';; # Insert mode -> Change to beam cursor
    esac
}
zle -N zle-keymap-select # Register the function as a Zsh Line Editor (ZLE) widget

# Initialize the command line defaults when a new line is drawn.
zle-line-init() {
    zle -K viins # Force the command line to always start fresh in Vi Insert mode
    echo -ne "\e[5 q" # Ensure the cursor resets to a beam shape
}
zle -N zle-line-init # Register the initialization widget

echo -ne '\e[5 q' # Set the beam cursor shape immediately when a new shell launches.

preexec() { echo -ne '\e[5 q' ;} # Reset cursor to a beam right after a command is sent, before drawing the next prompt.
````

> ## Understanding the Modes
> {.four}
 
When my  terminal now operates in two primary states. The cursor shape tells you which mode you are in:

- **Insert Mode (Beam Cursor `|`):** This is the default state. You type commands as you normally would.
- **Command Mode (Block Cursor `█`):** Triggered by hitting **`Esc`**. In this mode, keys perform editing actions (like `w`to jump words) rather than typing characters.

To return to typing (insert mode), use standard Vim commands like **`i`** (insert at cursor) or **`a`** (append after cursor).

> ## Custom Productivity Macros
. {.five}

You have mapped several powerful shortcuts that work while you are in **Insert Mode**:
- **`Ctrl-n`**: Opens the `fzf` fuzzy finder. Once you select a file, it automatically opens it in `nvim`.
- **`Ctrl-v`**: Instantly launches a blank `nvim` session.
- **`Ctrl-z`**: Runs `zi`, the interactive directory jumper (zoxide). This is the fastest way to navigate your filesystem.
- **`Ctrl-e`**: If you are typing a long, complex command and realize you need a full editor, hit this to move the current line into a full `nvim` buffer. Saving and exiting `nvim` will bring the command back to your prompt.

> ## History and Navigation
> {.one}

Navigating your command history is unified across both Insert and Command modes:
- **`Ctrl-k`**: Search backwards through history (Up).
- **`Ctrl-j`**: Search forwards through history (Down).
- **`Ctrl-p`**: Also functions as a backwards history search.

Because `KEYTIMEOUT` is set to `1`, the switch between hitting `Esc` and entering a command is near-instant, making the transition feel very responsive.

> ## Advanced Line Editing (Command Mode)
> {.two}

Once you hit **`Esc`**, you can use standard Vim motions to manipulate the command line:
- **`w` / `b`**: Jump forward or backward by word.
- **`0` / `$`**: Jump to the absolute beginning or end of the line.
- **`cw`**: Change a word (deletes the word and puts you back in Insert mode).
- **`dd`**: Clear the entire line.
- **`u`**: Undo the last change.

> ## Enhanced Tab Completion Menu
> {.three}

When you type a partial command and hit **Tab**, a completion menu appears. Your configuration allows you to navigate this menu using specific keys:
- **`Ctrl-k`**: Move selection Up.
- **`Ctrl-j`**: Move selection Down.
- **`Ctrl-h`**: Move selection Left.
- **`Ctrl-l`**: Move selection Right.

> ## Summary of Keybindings
> {.four}

|**Key**|**Mode**|**Action**|
|---|---|---|
|**`Esc`**|Insert|Switch to Command Mode (Block cursor)|
|**`i` / `a`**|Command|Return to Insert Mode (Beam cursor)|
|**`Ctrl-n`**|Insert|Fuzzy find file + open in `nvim`|
|**`Ctrl-z`**|Insert|Interactive directory jump (`zi`)|
|**`Ctrl-e`**|Both|Edit current line in full `nvim`|
|**`Ctrl-k/j`**|Both|Search through history|
|**`Ctrl-h/j/k/l`**|Menu|Menu mode navigate tab-completion suggestions|