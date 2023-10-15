---
title: "Windows"
type: "docs"
weight: 1
---

## 系统安装

镜像下载站 [https://hellowindows.cn](https://hellowindows.cn)，推荐 LSTC 企业版本，精简稳定。

使用 [Etcher](https://etcher.balena.io/#download-etcher)、软碟通 等工具刻录启动盘，开机时按 F8（华硕主板） 选择启动盘。

设置 C 盘容量 150GB 以上，为开发预留足够的空间。分区设置完毕，进入“下一步”前注意别选错了系统盘。

windows 默认没有无线网卡驱动，需要安装驱动。华硕 B650M-PLUS 重炮手主板的 WiFi， 360 驱动大师搞不定，必须使用驱动总裁。

如果没有有线网络，可以使用鸿蒙手机通过 USB 数据线共享网络，打开开发者模式，打开 USB 共享网络。

使用 hellowindows 的“一键激活脚本”激活系统，选择 KMS38方式。

## 开发环境

### Windows Terminal

打开 [Windows Terminal](https://github.com/microsoft/terminal)，下载最新 release 包安装。

### WSL

通过 WSL(Windows Subsystem for Linux) [适用于 Linux 的 Windows 子系统文档](https://learn.microsoft.com/zh-cn/windows/wsl/install) 安装 Debian。

Ubuntu 不稳定，推荐使用 Debian，稳定、轻量。

```shell
wsl --install -d Debian
```

### WSA

Windows subsystem for Android

预览阶段，[适用于 Android™️ 的 Windows 子系统](https://learn.microsoft.com/zh-cn/windows/android/wsa/)

### Scoop

Linux 风格的第三方开源 Windows 包管理工具。

[https://scoop.sh/](https://scoop.sh/)

[https://github.com/ScoopInstaller/Scoop](https://github.com/ScoopInstaller/Scoop)

```shell
# PowerShell(version 5.1 or later)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser # Optional: Needed to run a remote script the first time
irm get.scoop.sh | iex
```

### Winget

预览阶段，[Windows 官方程序包管理器](https://learn.microsoft.com/zh-cn/windows/package-manager/)

## SMB 共享

参考 [https://post.smzdm.com/p/akxwkxqk/](https://post.smzdm.com/p/akxwkxqk/)

```shell
# 启用大型MTU以提升大文件的传输速度，并关闭带宽限制
Set-SmbClientConfiguration -EnableBandwidthThrottling 0 -EnableLargeMtu 1

netsh int tcp set global autotuninglevel=restricted
netsh interface tcp set heuristics disabled
```
