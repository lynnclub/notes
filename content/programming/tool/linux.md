---
title: "Linux"
type: "docs"
weight: 1
---

## 系统信息

```bash
lsb_release -a
```

显示包括发行版名称、版本号、描述和发行日期在内的系统信息。

```bash
cat /etc/os-release
```

显示包含系统版本信息的文本文件，包括发行版名称、版本号等。

```bash
uname -a
```

显示有关系统内核的信息，包括系统版本号和架构。

```bash
date
```

显示当前时间，包括时区信息。

```bash
timedatectl
```

显示当前时间，包括更丰富的信息。

## 资源情况

你可以使用以下命令来获取有关进程、CPU、内存、磁盘和网络等资源的信息：

查看进程

```bash
# BSD风格的展示方式
ps aux
# 过滤
ps aux | grep firefox
# 完整信息
ps -ef
# 查看进程启动时间
ps -p <进程ID> -o lstart
```

查看 CPU 使用情况

```bash
top
```

这些命令会显示有关进程和 CPU 利用率的信息。

查看内存使用情况

```bash
free -h
```

这将显示总内存、已使用内存、可用内存等信息。

查看磁盘使用情况

```bash
df -h
```

这将显示磁盘分区的使用情况，包括总容量、已用空间和可用空间等信息。

查看网络使用情况

```bash
netstat -tuln

iftop
```

这些命令将显示有关网络连接和流量的信息。
