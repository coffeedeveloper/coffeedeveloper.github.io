---
layout: default
title: Linux下安装node.js、express环境
---

## Linux下安装node.js、express环境

### 安装nvm
```
curl https://raw.github.com/creationix/nvm/master/install.sh | sh
source ~/.nvm/nvm.sh
```

> [nvm github project](https://github.com/creationix/nvm)

如果你使用的是zsh的话可以在你的`~/.zshrc`文件中添加`source $HOME/.nvm/nvm.sh`

### 安装node.js
```
nvm install 0.10
nvm use 0.10
```
> [node.js project](http://www.nodejs.org)

### 安装express
```
sudo nvm install -g express
```
> 有一些系统需要sudo才能够用`-g`全局安装
