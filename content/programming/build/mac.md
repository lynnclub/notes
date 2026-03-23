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
brew install go php python@3 nvm rust
# Java8+tomcat9
brew install openjdk@8 tomcat@9 maven

# Docker Desktop 容器环境
# Sublime Text 文本编辑器
# Github Desktop 代码版本控制
# Dbeaver Community 数据库工具，不推荐使用sequel-ace
# 不推荐使用fig、notion
brew install --cask docker-desktop sublime-text github dbeaver-community mongodb-compass redis-insight another-redis-desktop-manager pgadmin4 cyberduck apifox

# Tailscale 组网工具
brew install --cask tailscale-app

# tailscale-app只能登录后启动，作为服务器开机自启不能用GUI
brew install tailscale
# 安装服务并启用开机自启
sudo tailscaled install-system-daemon
# 启动服务
sudo launchctl start com.tailscale.tailscaled
# 登录
tailscale login
# 作为出口节点
sudo tailscale set --advertise-exit-node
# 允许访问本地网络
sudo tailscale set --exit-node-allow-lan-access=true
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
