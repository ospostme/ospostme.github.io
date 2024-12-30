---
title: Dev Enviroment with Docker
date: 2024-12-12 14:31:00 Z
---

It's quite time-consuming to rebuild a dev env, especially cross different platforms. And due to the app/software version dependency is different, maintain a suit of unified working env on different platform is really not an interesting hobby. After a couple of weeks digging, I realize that it will be more reasonable to spend enough time to create a dev environment within docker.

- Once it's ready, it could be used on any platforms.
- Only need to save those important stuffs like dockerfile, customized dotfiles, install scripts, and notes(like this one).
- You can re-create the dev environment anytime

```

              Docker Image
      ┌──────────────────────────┐  ┌──────────────────────────────────────┐
      │   Basic ubuntu packages  │  │    Install scripts (manually trigger)│
      │   ┌───────────────────┐  │  │                                      │
      │   │  cmake,gcc,apt,gdb│  │  │ 1. Start v2ray, set proxy env        │
      │   │  ssh,git,wget,cul │  │  │ 2. Copy dotfiles to volumes($HOME)   │
      │   │  locales,htop,    │  │  │ 3. Create links of dotfiles          │
      │   │  fd-find, bat,    │  │  │ 4. Copy fff/fzf/nvim binary          │
      │   │  httping,net-tools│  │  │ 5. ruby-install/ruby/chruby          │
      │   │  build-essential  │  │  │ 6. gems sources                      │
      │   │  ...              │  │  │ 7. node version manager NVM          │
      │   │  ...              │  │  │ 8. Tmux & TPM (tmux package manager) │
      │   │  libs             │  │  │ 9. Necessary python lib with pip     │
      │   │                   │  │  │10. sdk man                           │
      │   └───────────────────┘  │  │11. desired java/flink/mavin/gradle   │
      │                          │  │                                      │
      │   Docker env setting     │  └──────────────────────────────────────┘
      │   ┌───────────────────┐  │         ▲
      │   │  locale, DISPLAY  │  │         │    Manual trigger after running
      │   │  TERM             │  │         │      ┌────────────────────────┐
      │   │  dotfiles         │  │         │      │ install brew           │
      │   │                   │  │         │─────►│ install conda/mimiforge│
      │   └───────────────────┘  │         │      └────────────────────────┘
      │                          │         │
      └──────────────────────────┘         │
      ┌────────────────────────────────────────────────────────────────────┐
      │   Container Compose                │                               │
      │   ┌───────────────────┐            │  ┌────────────────────────┐   │
      │   │  docker volumes───┼────────────┘  │  v2ray configs on host │   │
      │   │  bind volumes─────┼──────────────►│  workspace on host     │   │
      │   │  expose ports     │               │                        │   │
      │   └───────────────────┘               └────────────────────────┘   │
      └────────────────────────────────────────────────────────────────────┘


```

# v2ray

[v2ray](/2024/10/23/free-internet-with-v2ray)

# dotfiles

| file                | usage                                             |
| :------------------ | :------------------------------------------------ |
| .bashrc             | bash config/PATH                                  |
| .bash_aliases       | bash alias command                                |
| .pip/pip.conf       | python install package settting                   |
| .gitconfig          | git account/proxy/ssl settings                    |
| .fzf.conf           | fuzzy finder config                               |
| .tmux.conf          | tmux setting                                      |
| .tmux               | tmux plugin directory                             |
| .config/nvim        | nvim configuration                                |
| .fff                | desired version of file finder                    |
| .fzf                | desired version of fuzzy finder                   |
| .nvim-linux64       | desired version of nvim                           |
| .sdk-init/.sdkmanrc | records of java/flink/mavin/gradle/kotlin version |

## gitconfig

```config
[http "https://github.com"]
	proxy = socks5h://127.0.0.1:1080
[core]
	editor = vim
	excludesfile = /Users/ospost/.gitignore
[core]
	gitpath = /usr/local/Cellar/git/2.47.0/bin

[http "https://gitlab.xxx:1443"]
	sslVerify = false
[init]
	defaultBranch = main
[user]
	name =  xxxx
	email = xxxx
[pager]
	branch = false
[credential]
	helper = store

```

## pip.conf

```config
[global]
timeout = 6000
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn

break-system-packages = true

#user = true

```

## fzf

```bash
source "${HOME}/.fzf/fzf-completion.bash"
source "${HOME}/.fzf/fzf-key-bindings.bash"

export FZF_DEFAULT_COMMAND='fdfind --type f --color=never --hidden'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_ALT_C_COMMAND='fdfind --type d . --color=never --hidden'
# Open in tmux popup if on tmux, otherwise use --height mode
export FZF_DEFAULT_OPTS='--multi --height 40% --tmux center,80%,60% --layout reverse --border rounded'
# Open in tmux popup if on tmux, otherwise use --height mode
# Preview file content using bat (https://github.com/sharkdp/bat)
export FZF_CTRL_T_OPTS="
  --walker-skip .git,node_modules,target
  --multi
  --height 70%
  --tmux center,80%,60%
  --preview 'bat -n --color=always {}'"
export FZF_ALT_C_OPTS=" --height 70% --tmux center,80%,60% --preview 'tree -C {}'"

#export FZF_DEFAULT_OPTS='--multi --no-height --tmux center'
#export FZF_CTRL_T_OPTS="--tmux center,80%,60% --preview 'bat --color=always {}'"
#export FZF_ALT_C_OPTS="--preview 'tree -C {} | head -50'"

```

## bashrc

```bash
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

# don't put duplicate lines or lines starting with space in the history.
# See bash(1) for more options
HISTCONTROL=ignoreboth

# append to the history file, don't overwrite it
shopt -s histappend

# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000

# check the window size after each command and, if necessary,
# update the values of LINES and COLUMNS.
shopt -s checkwinsize

# If set, the pattern "**" used in a pathname expansion context will
# match all files and zero or more directories and subdirectories.
#shopt -s globstar

# make less more friendly for non-text input files, see lesspipe(1)
[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
#force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
	# We have color support; assume it's compliant with Ecma-48
	# (ISO/IEC-6429). (Lack of such support is extremely rare, and such
	# a case would tend to support setf rather than setaf.)
	color_prompt=yes
    else
	color_prompt=
    fi
fi

if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
unset color_prompt force_color_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*)
    PS1="\[\e]0;${debian_chroot:+($debian_chroot)}\u@\h: \w\a\]$PS1"
    ;;
*)
    ;;
esac

# enable color support of ls and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
fi

# colored GCC warnings and errors
#export GCC_COLORS='error=01;31:warning=01;35:note=01;36:caret=01;32:locus=01:quote=01'

# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'

# Add an "alert" alias for long running commands.  Use like so:
#   sleep 10; alert
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'

# Alias definitions.
# You may want to put all your additions into a separate file like
# ~/.bash_aliases, instead of adding them here directly.
# See /usr/share/doc/bash-doc/examples in the bash-doc package.

if [ -f ~/.bash_aliases ]; then
    . ~/.bash_aliases
fi

# enable programmable completion features (you don't need to enable
# this, if it's already enabled in /etc/bash.bashrc and /etc/profile
# sources /etc/bash.bashrc).
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
#------------------------------------------------------------------------------
# End of original bashrc
#------------------------------------------------------------------------------
export PATH=${HOME}/bin:$PATH
export PATH=${HOME}/usr/local/bin:$PATH

source "/home/ospost/usr/local/share/chruby/chruby.sh"
source "/home/ospost/usr/local/share/chruby/auto.sh"
chruby ruby-3.3.5

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
export GOPROXY=https://goproxy.cn
source "$HOME/.fzf.conf"

export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"

eval "$(/home/ospost/linuxbrew/.linuxbrew/bin/brew shellenv)"

#THIS MUST BE AT THE END OF THE FILE FOR SDKMAN TO WORK!!!
export SDKMAN_DIR="$HOME/.sdkman"
[[ -s "$HOME/.sdkman/bin/sdkman-init.sh" ]] && source "$HOME/.sdkman/bin/sdkman-init.sh"

# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/ospost/miniforge3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/ospost/miniforge3/etc/profile.d/conda.sh" ]; then
        . "/home/ospost/miniforge3/etc/profile.d/conda.sh"
    else
        export PATH="/home/ospost/miniforge3/bin:$PATH"
    fi
fi
unset __conda_setup

if [ -f "/home/ospost/miniforge3/etc/profile.d/mamba.sh" ]; then
    . "/home/ospost/miniforge3/etc/profile.d/mamba.sh"
fi
# <<< conda initialize <<<
```

## tmux

[Tmux crack](https://youtu.be/niuOc02Rvrc?list=PLsz00TDipIfdrJDjpULKY7mQlIFi4HjdR)

```config
# Default leader CTRL+B
# Easy config reload
unbind r
bind-key r source-file ~/.tmux.conf \; display-message "tmux.conf reloaded."

# vi is good
#setw -g mode-keys vi

# mouse behavior
set -g mouse on

set-option -g default-terminal screen-256color

# use vim-like keys for splits and windows
#bind-key h select-pane -L
#bind-key j select-pane -D
#bind-key k select-pane -U
#bind-key l select-pane -R

#bind-key V split-window -h -c "#{pane_current_path}"
#bind-key S split-window -v -c "#{pane_current_path}"

# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'dracula/tmux'
set -g @plugin 'christoomey/vim-tmux-navigator'
set -g @plugin 'jimeh/tmuxifier'
#set -g @plugin 'oluevaera/tmux-conda-inherit'

# uncomment below stanza to enable smart pane switching with awareness of vim splits
#bind -n C-h run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)vim$' && tmux send-keys C-h) || tmux select-pane -L"
#bind -n C-j run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)vim$' && tmux send-keys C-j) || tmux select-pane -D"
#bind -n C-k run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)vim$' && tmux send-keys C-k) || tmux select-pane -U"
#bind -n C-l run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)vim$' && tmux send-keys C-l) || tmux select-pane -R"
#bind -n C-\ run "(tmux display-message -p '#{pane_current_command}' | grep -iqE '(^|\/)vim$' && tmux send-keys 'C-\\') || tmux select-pane -l"
#bind C-l send-keys 'C-l'

# available plugins: battery, cpu-usage, git, gpu-usage, ram-usage, tmux-ram-usage,
# network, network-bandwidth, network-ping, ssh-session, attached-clients, network-vpn, weather, time, mpc, spotify-tui, krbtgt,
# playerctl, kubernetes-context, synchronize-panes
set -g @dracula-plugins "cpu-usage ram-usage battery"
set -g @dracula-show-powerline true

# Make the status line pretty and add some modules
#set -g status-right-length 100
#set -g status-left-length 100
#set -g status-left ""

run-shell "tmux setenv -g TMUX_VERSION $(tmux -V | cut -c 6-)"

# Setup 'v' to begin selection as in Vim
# Update default binding of `Enter` to also use copy-pipe
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "reattach-to-user-namespace pbcopy"
unbind -T copy-mode-vi Enter
bind-key -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel "reattach-to-user-namespace pbcopy"

set-window-option -g display-panes-time 1500

# Status Bar
#set-option -g status-interval 1
#set-option -g status-left ''
#set-option -g status-right '%l:%M%p'
#set-window-option -g window-status-current-style fg=magenta
#set-option -g status-style fg=default
#set-option -g status-position top

# Status Bar solarized-dark (default)
#set-option -g status-style bg=black
#set-option -g pane-active-border-style fg=white
#set-option -g pane-border-style fg=white

# Status Bar solarized-light
#if-shell "[ \"$COLORFGBG\" = \"11;15\" ]" "set-option -g status-style bg=white"
#if-shell "[ \"$COLORFGBG\" = \"11;15\" ]" "set-option -g pane-active-border-style fg=white"
#if-shell "[ \"$COLORFGBG\" = \"11;15\" ]" "set-option -g pane-border-style fg=white"

# Set window notifications
setw -g monitor-activity on
set -g visual-activity on

# Enable native Mac OS X copy/paste
#set -g default-command "/usr/local/bin/bash"
#set-option -g default-command "$SHELL -c 'which reattach-to-user-namespace >/dev/null && exec reattach-to-user-namespace $SHELL -l || exec $SHELL -l'"

# Allow the arrow key to be used immediately after changing windows
set-option -g repeat-time 0

# Fix to allow mousewheel/trackpad scrolling in tmux 2.1
bind-key -T root WheelUpPane if-shell -F -t = "#{alternate_on}" "send-keys -M" "select-pane -t =; copy-mode -e; send-keys -M"
bind-key -T root WheelDownPane if-shell -F -t = "#{alternate_on}" "send-keys -M" "select-pane -t =; send-keys -M"

# Disable assume-paste-time, so that iTerm2's "Send Hex Codes" feature works
# with tmux 2.1. This is backwards-compatible with earlier versions of tmux,
# AFAICT.
set-option -g assume-paste-time 0
#set-option -ag update-environment 'CONDA_DEFAULT_ENV'

source-file ~/.tmux.local.conf

# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'

```

# Ruby/Jekeyll

```bash

CMD_RUBY_INSTALL_DL='wget https://github.com/postmodern/ruby-install/releases/download/v0.9.3/ruby-install-0.9.3.tar.gz'
CMD_CHRUBY_DL='wget https://github.com/postmodern/chruby/releases/download/v0.3.9/chruby-0.3.9.tar.gz'
MAX_REPEAT=20

cd "${HOME}" || exit

HTTP_PROXY=http://127.0.0.1:8888
export https_proxy=${HTTP_PROXY}

for ((i = 0; i < $MAX_REPEAT; i++)); do
    $CMD_RUBY_INSTALL_DL
    if [ $? -ne 0 ]; then
        message "wget failed :( tray again !" 3 10
    fi
    break
done
[ $i -lt $MAX_REPEAT ] || exit

tar -xzvf ruby-install-0.9.3.tar.gz

cd ruby-install-0.9.3/ || exit
echo "Faint1231" | sudo -S make install DESTDIR="${HOME}"

# 4 of the 10 steps
message "ruby-install is ready!" 4 10

cat >>~/.$(basename $SHELL)rc <<EOF
export PATH=${HOME}/usr/local/bin:$PATH
EOF
export PATH=${HOME}/usr/local/bin:$PATH

ruby-install ruby 3.3.6

# 5 of the 10 steps
message "ruby 3.3.6 is ready!" 5 10

cd "${HOME}" || exit

for ((i = 0; i < $MAX_REPEAT; i++)); do
    $CMD_CHRUBY_DL
    if [ $? -ne 0 ]; then
        message "wget failed :( tray again !" 5 10
    fi
    break
done
[ $i -lt $MAX_REPEAT ] || exit

tar -xzvf chruby-0.3.9.tar.gz
cd chruby-0.3.9/ || exit
echo "password" | sudo -S make install DESTDIR="${HOME}"

export PREFIX="${HOME}"
echo "password" | sudo -S ./scripts/setup.sh
unset PREFIX

cat >>~/.$(basename $SHELL)rc <<EOF
source "${HOME}/usr/local/share/chruby/chruby.sh"
source "${HOME}/usr/local/share/chruby/auto.sh"
chruby ruby-3.3.6
EOF
source "${HOME}/usr/local/share/chruby/chruby.sh"
source "${HOME}/usr/local/share/chruby/auto.sh"

# 6 of the 10 steps
message "chruby is ready!" 6 10

chruby ruby-3.3.6
# gem install
gem sources --remove https://rubygems.org/
gem sources -a https://gems.ruby-china.com/
gem install jekyll bundler

# 7 of the 10 steps
message "Jekyll is ready!" 7 10

cd "${HOME}" || exit
rm -fr chruby-0.3.9.tar.gz chruby-0.3.9/ ruby-install-0.9.3.tar.gz ruby-install-0.9.3/
```

# NVM/Node

```bash
# Node Version Manager
# https://github.com/nvm-sh/nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
nvm install --lts

# 8 of the 10 steps
message "Node version manger is ready!" 8 10

for ((i = 0; i < $MAX_REPEAT; i++)); do
    $CMD_TMUX_DL
    if [ $? -ne 0 ]; then
        message "wget failed :( tray again !" 5 10
    fi
    break
done
[ $i -lt $MAX_REPEAT ] || exit

# 8 of the 10 steps
message "Node version manger is ready!" 8 10
```

# tmux

```bash

CMD_TMUX_DL='wget https://github.com/tmux/tmux/releases/download/3.5a/tmux-3.5a.tar.gz'
CMD_TPM_DL='git clone https://github.com/tmux-plugins/tpm .tmux/plugins/tpm'
MAX_REPEAT=20

for ((i = 0; i < $MAX_REPEAT; i++)); do
    $CMD_TMUX_DL
    if [ $? -ne 0 ]; then
        message "wget failed :( tray again !" 5 10
    fi
    break
done
[ $i -lt $MAX_REPEAT ] || exit

tar zxvf tmux-3.5a.tar.gz
cd tmux-3.5a/ || exit
./configure --prefix="${HOME}/usr/local"

echo "Faint1231" | sudo -S make install

# has been exported
#cat >>~/.$(basename $SHELL)rc <<EOF
#export PATH=${HOME}/usr/local/bin:$PATH
#EOF
#export PATH=${HOME}/usr/local/bin:$PATH
#cd "${HOME}" || exit

cd "${HOME}" || exit
rm -fr tmux-*

$CMD_TPM_DL

#  9 of the 10 steps
message "tmux installed" 9 10

```

# sdk-man

```bash
CMD_SDKMAN_DL='sdk env install'
MAX_REPEAT=20

curl -s "https://get.sdkman.io" | bash
source "${HOME}/.sdkman/bin/sdkman-init.sh"

cd "${dotfiledir}/.sdk-init" || exit

for ((i = 0; i < $MAX_REPEAT; i++)); do
    $CMD_SDKMAN_DL
    if [ $? -ne 0 ]; then
        message "wget failed :( tray again !" 3 10
    fi
    break
done
[ $i -lt $MAX_REPEAT ] || exit

#  10 of the 10 steps
message "sdkman and basic java is ready!" 10 10


```

# nvim

Necessary proxy, dependency for nvim lazy install, Mason LSP install

```bash

HTTP_PROXY=http://127.0.0.1:8888
export https_proxy=${HTTP_PROXY}

# NVIM LSP
cat >>~/.$(basename $SHELL)rc <<EOF
export GOPROXY=https://goproxy.cn
source "${HOME}/.fzf.conf"
EOF
export GOPROXY=https://goproxy.cn
source "${HOME}/.fzf.conf"

# NVIM LSP
pip install pysocks

```

## Common

- kickstart

  [NVIM kickstart](https://youtu.be/m8C0Cq9Uv9o)

  [A Great NVIM kickstart template](https://github.com/nvim-lua/kickstart.nvim)

  [From 0 to IDE in NEOVIM from scratch](https://youtu.be/zHTeCSVAFNY?list=PLsz00TDipIffreIaUNk64KxTIkQaGguqn)

  [workflow example](https://www.youtube.com/watch?v=G7-qUMKSH_Y&t=784s)

- MarkDown

  [MarkDown Crash Course](https://youtu.be/_PPWWRV6gbA)

  [Syntax highlighting](https://github.com/preservim/vim-markdown)

  [Markdown direct preview in nvim](https://github.com/ellisonleao/glow.nvim)

  [Zoom focus/Distraction-free writing](https://github.com/junegunn/goyo.vim)

  [Hyperfocus-writing](https://github.com/junegunn/limelight.vim)

  [improve viewing Markdown files in Neovim](https://github.com/MeanderingProgrammer/render-markdown.nvim)

  [Draw ASCII diagrams in Neovim](https://github.com/jbyuki/venn.nvim)

  [Preview Markdown in your modern browser with synchronised scrolling and flexible configuration](https://github.com/iamcco/markdown-preview.nvim)

- Mason
  Portable package manager for Neovim that runs everywhere Neovim runs.Easily install and manage LSP servers, DAP servers, linters, and formatters.

- Treesitter Highlight edit and navigate code

  [What is Treesitter](https://youtu.be/09-9LltqWLY)

- LSP

  [LSP in nvim](https://youtu.be/S-xzYgTLVJE)

  [Learn By Building: Language Server Protocol](https://youtu.be/YsdlcQoHqPY)

- Completion

  [vim built-in autocomplete](https://youtu.be/tFD2Ia5TIQ8)

  [Vim Autocomplete Mini-Overview](https://youtu.be/bu_AIAp7hCY)

  [Autocomplete and Snippets in Neovim](https://youtu.be/iXIwm4mCpuc)

  [nvim complete](https://github.com/hrsh7th/nvim-cmp)

  [nvim-cmp source for buffer words](https://github.com/hrsh7th/cmp-buffer)

  [A dictionary completion source for nvim-cmp](https://github.com/uga-rosa/cmp-dictionary)

  [wamerican](https://unix.stackexchange.com/questions/213628/where-do-the-words-in-usr-share-dict-words-come-from)

  [spell source for nvim-cmp based on vim's spellsuggest](https://github.com/f3fora/cmp-spell)

  [Tags generator/management for old school vimers in Neovim](https://github.com/linrongbin16/gentags.nvim)

  [tags completion source for nvim-cmp](https://github.com/quangnguyen30192/cmp-nvim-tags)

- Debugger

  [TBD](TBD)

## C/C++ dev

- CMake

  [CMake vs Make](https://keasigmadelta.com/blog/cmake-vs-make-a-developers-perspective/?srsltid=AfmBOor4RTmyF6eVtFlLbJhein3xl2cjBeERAJ3_Vd-tVcyC5BxHZW_q)
  [CMake Learning](https://cliutils.gitlab.io/modern-cmake/README.html)

- Conan
  Conan is a dependency and package manager for C and C++ languages. It is free and open-source, works in all platforms ( Windows, Linux, OSX, FreeBSD, Solaris, etc.), and can be used to develop for all targets including embedded, mobile (iOS, Android), and bare metal. It also integrates with all build systems like CMake, Visual Studio (MSBuild), Makefiles, SCons, etc., including proprietary ones.

- gcc/gdb/clang/llvm

- nvim plugin

  [cmake plugin](https://github.com/cdelledonne/vim-cmake)

  [Gtest]()

  [lsp]()

  [treesitter]()

  [debug adapter]()

  [linter]()

  [format]()

## Python dev

[lsp]()

[treesitter]()

[debug adapter]()

[linter]()

[format]()

[vim test runner]()

## Java dev

# Brew

```bash

export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"

#/bin/bash -c ./brew.sh
curl https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh  > brew.sh
```

Change HOMEBREW_PREFIX in brew.sh, then install

# miniforge3/conda

Community-driven packaging for conda
