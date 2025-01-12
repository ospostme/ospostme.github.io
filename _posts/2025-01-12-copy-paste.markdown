---
title: 2024 12 31 Copy Paste
date: 2025-01-12 20:34:10.503000000 Z
---

<!--toc:start-->

- [Clipboard](#clipboard)
  - [macOS](#macos)
  - [Windows](#windows)
  - [Linux (Ubuntu)](#linux-ubuntu)
- [NVIM Clipboard and Register](#nvim-clipboard-and-register)
  - [Registers](#registers)
  - [Clipboard](#clipboard)
- [Tmux](#tmux)
  - [defaults](#defaults)
  - [vim-bindings](#vim-bindings)
    - [VI copy mode](#vi-copy-mode)
    - [VIM style Copy-Paste](#vim-style-copy-paste)
    - [Copy with Mouse Drag](#copy-with-mouse-drag)
    - [Extra benefit](#extra-benefit)
- [SSH](#ssh)
- [Docker](#docker)
- [NVIM within Tmux navigation on Dev Docker hosted on MacOS](#nvim-within-tmux-navigation-on-dev-docker-hosted-on-macos)
<!--toc:end-->

# Clipboard

The clipboard is a temporary buffer that allows users to copy and paste content
between applications and operating systems.

## macOS

Virtual Mode when choose something with mouse, `command + c`, `command + v`
In Terminal with `pbcopy` and `pbpaste`

```cmd
ospost(main) $ ls
Dockerfile  dev-lab.yml  dotfiles
Tue Dec 31 10:27:33 CST 2024 ospost@MacPro.local:/Users/ospost/workspace/docker
ospost(main) $ pbpaste
ospost(main) $ ls | pbcopy
Tue Dec 31 10:26:30 CST 2024 ospost@MacPro.local:/Users/ospost/workspace/docker
ospost(main) $ pbpaste
Dockerfile
dev-lab.yml

```

## Windows

`CTRL+C/CTRL+V`

## Linux (Ubuntu)

`CTRL+C` is killing signal on Linux
Once you have text highlighted within the terminal, you can use `CTRL+INSERT`,
`SHIFT+INSERT` to copy, paste.
`CTRL+SHIFT+C`, `CTRL+SHIFT+V`

# NVIM Clipboard and Register

## Registers

```
- 1. The unnamed register ""
- 2. 10 numbered registers "0 to "9
- 3. The small delete register "-
- 4. 26 named registers "a to "z or "A to "Z
- 5. Three read-only registers ":, "., "%
- 6. Alternate buffer register "#
- 7. The expression register "=
- 8. The selection registers "* and "+
- 9. The black hole register "_
- 10. Last search pattern register "/
```

- unnamed register
  Vim fills this register with text deleted with the “d”, “c”, “s”, “x” commands
  or copied with the yank “y” command

- Selection registers `quotestar quoteplus`
  Use these registers for storing and retrieving the selected text for the GUI.
  When the clipboard is not available or not working, the unnamed register is
  used instead.

## Clipboard

- X11 clipboard providers store text in “selections”. Selections are owned by an
  application. Three X11 selections: PRIMARY, SECONDARY, and CLIPBOARD.

  - PRIMARY used for the last selected text, which is generally inserted with
    the middle mouse button
  - CLIPBOARD is typically used in X11 applications for copy/paste operations
    (CTRL-c/CTRL-v)

Nvim's X11 clipboard providers only use the PRIMARY and CLIPBOARD selections,
for the `"*"` and `"+"` registers, respectively.

# Tmux

[Everything you need to know about Tmux copy and paste - Ubuntu](https://www.rushiagr.com/blog/2016/06/16/everything-you-need-to-know-about-tmux-copy-pasting-ubuntu/)
`CTRL+b then ?` for tmux key help

Copying with tmux is more like copying in Vim, where it's best done with a
keyboard rather than a mouse. This is done in tmux copy mode. Tmux has its
own buffer for copying. By default, moving around in copy mode using arrows.

## defaults

- 1. Enter ‘copy mode’ by pressing `CTRL`+`b, [`
- 2. Use the arrow keys to go to the position from where you want to start
     copying. Press `CTRL+SPACE` to start copying.
- 3. Use arrow keys to go to the end of text you want to copy. Press `ALT+w` or
     `CTRL+w` to copy into Tmux buffer.
- 4. Press `CTRL+b, ]` to paste in a possibly different Tmux pane/window.

## vim-bindings

### VI copy mode

To navigate your terminal history and to copy text, you need to switch to tmux
copy mode. With following config you can move around using vim navigation keys.
In this mode you can move around the terminal like vim.
`setw -g mode-keys via`

### VIM style Copy-Paste

```
bind P paste-buffer
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X rectangle-toggle
unbind -T copy-mode-vi Enter
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'xclip -sel clip -i'

```

- Navigate copy mode with vi-like-key bindings
- Selecting copy block just like in vim
- yank with `y` to tmux buffer, and to X11 CLIPBOARD at the same time
- Automatically cancel copy mode
- Paste workable with `p` or `CTRL-v` or `CTRL-b, P`

### Copy with Mouse Drag

```
set -g mouse on
bind-key -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel 'xclip -sel clip -i'

```

### Extra benefit

Since in vi-copy-mode, navigation in the terminal history contents just like
in vim, `? + keywords` search backward for all the keywords with highlight.

# SSH

Remote login into Linux Server with tmux config

```
bind -t vi-copy y copy-pipe "xclip -sel clip -i"
ssh -X name@hostname

```

Everything copy into remote tmux buffer, will get copied over to local
clipboard also. Use CTRL-V could paste to the local host.

# Docker

[X11 forwarding on macOS and docker](https://gist.github.com/sorny/969fe55d85c9b0035b0109a31cbcb088)
[How to yank to host clipboard from inside a Docker container?](https://stackoverflow.com/questions/43075050/how-to-yank-to-host-clipboard-from-inside-a-docker-container)
[Access Unix Clipboard.](https://unix.stackexchange.com/questions/44204/access-unix-clipboard)

- macOS

  - install xorg-server `brew install --cask xquartz`
  - start server `open -a XQuartz`
  - enable forwarding within localhost `xhost +localhost`

- Docker
  - Install xclip
  * Add DISPLAY environment variable for x forwarding `DISPLAY=host.docker.internal:0`

Install X11 server on Docker is also an option, but that will increase docker images size
significantly.

# NVIM within Tmux navigation on Dev Docker hosted on macOS

- Tmux

  - `christoomey/vim-tmux-navigator'` allow you to navigate seamlessly between
    vim and tmux splits using a consistent set of hotkeys.

- NVIM
  - `christoomey/vim-tmux-navigator` navigate seamlessly between vim and tmux
    splits using a consistent set of hotkeys
  - `preservim/vimux` interacting with tmux from vim effortless. Open a small
    panel for command execution without lose vim focus. Easy to copy execution
    output, execute the last command, zoom the runner panel.
    Although it was originally designed for running test, copy results, still
    deserve to notice, especially `VimuxInspectRunner` and `VimuxZoomRunner`.

```vim
return {
  -- https://www.bugsnag.com/blog/tmux-and-vim
  'preservim/vimux',
  config = function()
    vim.cmd [[ nnoremap <leader>tp :VimuxPromptCommand<CR> ]]
    vim.cmd [[ nnoremap <Leader>tl :VimuxRunLastCommand<CR> ]]
    vim.cmd [[ nnoremap <Leader>ti :VimuxInspectRunner<CR> ]]
    vim.cmd [[ nnoremap <leader>tz :VimuxZoomRunner<CR> ]]
    vim.cmd [[ let g:VimuxCommandShell = 1 ]]
  end,
}
```
