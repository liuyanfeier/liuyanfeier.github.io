---
title:      item2和zsh配置
author:     liuyan
catalog:    true
tags:
  - iTerm2
  - zsh
  - linux
date: 2017-01-04 16:13:01
urlname:
categories: linux
---

## iTerm2安装与配置

### 安装iTerm2

iTerm2官网：https://www.iterm2.com/

### Solarized主题配置

iTerm2支持许多的主题配色，可以自己定义，也可以参考网上现成的主题配色。我个人比较喜欢Solarized配色，因为可以配合Vim里面的Solarized主题。

<!-- more -->

![](1.png)

```shell
# git下Solarized 的源码
git clone git://github.com/altercation/solarized.git
cd solarized/vim-colors-solarized/colors

sudo mkdir -p ~/.vim/colors
sudo cp solarized.vim ~/.vim/colors/

# 把下面这三行复制到~/.vimrc
syntax enable
set background=dark
colorscheme solarized
```

要使ITerm2和vim的显示效果保持一致，还需要最后一步设置：**Preferences -> Profiles ->Text**中取消Draw bold text in bright color的勾选。

### Powerline字体

```sh
git clone https://github.com/powerline/fonts.git
cd fonts
./install.sh			#安装所有Powerline字体
```

然后到 iterm2 配置，设置字体为`Roboto Mono for Powerline`

![](2.png)

## zsh安装与配置

### 安装zsh

```shell
cat /etc/shells				#查看系统内置的shell
sudo brew install zsh		#安装zsh
chsh -s /bin/zsh			#将zsh配置为默认的shell
chsh -s /bin/bash			#默认shell切换回bash
echo $SHELL					#查看当前shell
```

### 安装oh-my-zsh

```shell
git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
cp ~/.zshrc ~/.zshrc.bk
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

cd ~/.oh-my-zsh/tools
./install.sh
```

### 主题配置

通过修改`~/.zshrc`中的环境变量`ZSH_THEME`可以进行zsh主题配置

查看主题：https://github.com/robbyrussell/oh-my-zsh/wiki/Themes

```shell
ZSH_THEME="agnoster"      
ZSH_THEME="random" 		#主题随机变化
```

