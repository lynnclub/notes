---
title: "Mac"
type: "docs"
weight: 1
---

## Mac开发环境

第三方包管理工具 HomeBrew 安装请参考文档 [镜像](../mirror/#homebrew)。

```shell
# 基础软件
# 不推荐frpc，建议使用cloudflared tunnel
brew install nginx mariadb redis sqlite

# go、php、python、nodejs、rust
brew install golang php python@3 nvm rust

# Java8+tomcat9
brew install openjdk@8 maven tomcat@9

# Docker、Github Desktop、dbeaver-community数据库工具
# 不推荐使用fig、sequel-ace、notion
brew install --cask docker github dbeaver-community mongodb-compass another-redis-desktop-manager pgadmin4
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

UI工具建议使用 [BalenaEtcher](https://etcher.balena.io/)
