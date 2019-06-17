---
title: mac终端配置
date: 2017-08-22 16:43:29
tags: mac
---

### 1.安装iTerm2
### 2.安装oh-my-zsh

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
* 使用该指令能自动切换bash为zsh
* 将oh-my-zsh的主题切换为ys
* `brew install autojump`，在.zshrc中添加plugins中的autojump，并添加`[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.    d/autojump.sh`

### 3.安装dircolors-solarized
* 安装之后能够使得终端中使用ls指令具有彩色的输出
* `brew install coreutils`
* `git clone https://github.com/liujiayi771/dircolors-solarized.git`，将clone文件夹中的dircolors.ansi-dark复制到~/.dir_colors
* 在.zshrc中添加如下内容

```
if brew list | grep coreutils > /dev/null ; then
	PATH="$(brew --prefix coreutils)/libexec/gnubin:$PATH"
	alias ls='ls -F --show-control-chars --color=auto'
	eval `gdircolors -b $HOME/.dir_colors`
fi
```
### 4.修改vim主题
* `git clone git://github.com/altercation/solarized.git`
* `mkdir -p ~/.vim/colors`
* `cp solarized/vim-colors-solarized/colors/solarized.vim ~/.vim/colors`
* `vim ~/.vimrc`，添加如下内容

```
syntax enable
set background=dark
colorscheme solarized
set nu
```