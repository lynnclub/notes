---
title: "Windows"
type: "docs"
weight: 3
---

## 系统安装

纯净镜像下载站 [https://hellowindows.cn](https://hellowindows.cn)

使用 软碟通 刻录启动盘。

华硕主板按 F8 选择启动盘。

C 盘容量设置为 150GB，预留足够的空间。

分区设置完毕，进入“下一步”前注意别选错了系统盘。

win10 企业 LSTC 版本等大多数 windows，默认没有无线网卡驱动，需要使用 360/驱动总裁 安装驱动。华硕 B650M-PLUS 重炮手主板的 WiFi 360 搞不定，必须使用驱动总裁。

有线网络可以直接联网，或者使用鸿蒙手机通过 USB 数据线共享网络，打开开发者模式，打开 USB 共享网络。

激活系统，使用一键激活脚本，选择 KMS38。

## 环境

安装 [Windows Terminal](https://github.com/microsoft/terminal)，下载最新 release 包。

通过 [WSL(Windows Subsystem for Linux)](https://learn.microsoft.com/zh-cn/windows/wsl/install) 安装 Debian。

```bash
wsl --install -d Debian
```

安装 [scoop](https://scoop.sh/)

## SMB共享

[https://post.smzdm.com/p/akxwkxqk/](https://post.smzdm.com/p/akxwkxqk/)

```bash
#启用大型MTU以提升大文件的传输速度，并关闭带宽限制
Set-SmbClientConfiguration -EnableBandwidthThrottling 0 -EnableLargeMtu 1

netsh int tcp set global autotuninglevel=restricted
netsh interface tcp set heuristics disabled
```
