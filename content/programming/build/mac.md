---
title: "Mac"
type: "docs"
weight: 1
---

## 开发环境

安装 homebrew 请参考 [镜像 HomeBrew 章节](../mirror/#homebrew) 。

```shell
# 基础软件
brew install nginx mariadb redis sqlite frpc

# go、php、python、nodejs、rust
brew install golang php python@3 nvm rust

# Java8+tomcat9
brew install openjdk@8 maven tomcat@9

# Docker、Github Desktop、fig命令行补全、dbeaver-community数据库工具、notion笔记
brew install --cask docker github fig dbeaver-community mongodb-compass sequel-ace another-redis-desktop-manager pgadmin4 notion
```

## 刻录 Windows 系统盘

```shell
#硬盘列表
diskutil list

# 清除U盘，重命名卷，建立MBR引导，不支持大于2TB容量，对应Legacy BIOS
diskutil eraseDisk MS-DOS "WINDOWS10" MBR disk#
# GPT引导，需要主板支持UEFI BIOS，老主板必须使用MBR
diskutil eraseDisk MS-DOS "WINDOWS10" GPT disk#

# 打开iso镜像，查看win10卷名，复制到U盘
cp -rp /Volumes/CES_X64FREV_ZH-CN_DV9/* /Volumes/WINDOWS10/
```
