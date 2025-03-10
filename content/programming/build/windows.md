---
title: "Windows"
type: "docs"
weight: 1
---

## 系统安装

镜像下载站 [https://hellowindows.cn](https://hellowindows.cn)

推荐使用 LSTC 企业版本，精简、强大、稳定。

使用 [Etcher](https://etcher.balena.io/#download-etcher)、软碟通 等工具刻录启动盘，开机时按 F8（华硕主板） 选择启动盘。

设置 C 盘容量 150GB 以上，为开发依赖包预留足够的空间。

Windows 默认没有无线网卡驱动，需要安装驱动。华硕 B650M-PLUS 重炮手主板的 WiFi， 360 驱动大师搞不定，必须使用[驱动总裁](https://www.sysceo.com/dc.html)。

如果没有有线网络，可以使用安卓/鸿蒙手机通过 USB 数据线共享网络，打开开发者模式，打开 USB 共享网络。

使用 hellowindows 的“一键激活脚本”激活系统，选择 KMS38 方式。

## 开发环境

### Windows Terminal

[[Windows Terminal](https://github.com/microsoft/terminal)](https://github.com/microsoft/terminal)

下载 release 最新版本安装。

### WSL

通过 WSL(Windows Subsystem for Linux) [适用于 Linux 的 Windows 子系统文档](https://learn.microsoft.com/zh-cn/windows/wsl/install) 安装 Linux。

Ubuntu 普通版本不稳定，建议使用LTS版本，或者使用 Debian，稳定、轻量。

```shell
wsl --install -d Debian
```

### WSA

Windows subsystem for Android

预览阶段，[适用于 Android™️ 的 Windows 子系统](https://learn.microsoft.com/zh-cn/windows/android/wsa/)

### Scoop

Windows 第三方包管理工具 scoop 安装请参考 [镜像](../mirror/#scoop)。

### Winget

[Windows 官方程序包管理器](https://learn.microsoft.com/zh-cn/windows/package-manager/)

## SMB 共享

SMB（Server Message Block）协议主要用于局域网文件共享，它依赖于 TCP 139或445 端口，而大多数 ISP 和防火墙默认会阻止 445 端口的外部访问，以防止安全风险（如 勒索软件、蠕虫攻击）。

参考 [https://post.smzdm.com/p/akxwkxqk/](https://post.smzdm.com/p/akxwkxqk/)

```shell
# 启用大型MTU以提升大文件的传输速度，并关闭带宽限制
Set-SmbClientConfiguration -EnableBandwidthThrottling 0 -EnableLargeMtu 1

netsh int tcp set global autotuninglevel=restricted
netsh interface tcp set heuristics disabled
```

互联网文件共享建议使用webdav。

## SSH

参考 [https://www.jianshu.com/p/04e64bfcc79b](https://www.jianshu.com/p/04e64bfcc79b)

默认 22 端口，编辑 C:\ProgramData\ssh\sshd_config 可修改端口

```shell
# 开机自启与启动
Get-Service -Name sshd | Set-Service -StartupType Automatic
Get-Service -Name ssh-agent | Set-Service -StartupType Automatic
Start-Service sshd
Start-Service ssh-agent
```
