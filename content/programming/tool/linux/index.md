---
title: "Linux"
type: "docs"
weight: 1
---

## 常用命令

```shell
# 查看文件尾部最后几行内容
tail -n 20 filename
# 监视文件尾部内容变化
tail -f filename
```

## 发行版本

![relation](relation.png)

![package](package.png)

## 系统信息

```shell
# 显示包括发行版名称、版本号、描述和发行日期在内的系统信息。
lsb_release -a

# 显示包含系统版本信息的文本文件，包括发行版名称、版本号等。
cat /etc/os-release

# 显示有关系统内核的信息，包括系统版本号和架构。
uname -a
```

```shell
# 显示当前时间，包括时区信息。
date

# 显示当前时间，包括更丰富的信息。
timedatectl

# 查看所有环境变量
printenv
```

## 设置限制

```shell
# 显示资源限制
ulimit -a
# 关闭core文件上限
ulimit -c 0
# 设置打开文件数
ulimit -n 1024000
# 修改文件，永久生效
vi /etc/security/limits.conf
```

参数：

-a 　显示目前资源限制的设定。  
-c <core 文件上限>　设定 core 文件的最大值，单位为区块。  
-d <数据节区大小>　程序数据节区的最大值，单位为 KB。  
-f <文件大小>　 shell 所能建立的最大文件，单位为区块。  
-m <内存大小>　指定可使用内存的上限，单位为 KB。  
-n <文件数目>　指定同一时间最多可开启的文件数。  
-p <缓冲区大小>　指定管道缓冲区的大小，单位 512 字节。  
-s <堆叠大小>　指定堆叠的上限，单位为 KB。  
-t <CPU 时间>　指定 CPU 使用时间的上限，单位为秒。  
-u <程序数目>　用户最多可开启的程序数目。  
-v <虚拟内存大小>　指定可使用的虚拟内存上限，单位为 KB。

## 资源情况

你可以使用以下命令来获取有关进程、CPU、内存、磁盘和网络等资源的信息：

查看进程

```shell
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

```shell
top
```

这些命令会显示有关进程和 CPU 利用率的信息。

查看内存使用情况

```shell
free -h
```

这将显示总内存、已使用内存、可用内存等信息。

查看磁盘使用情况

```shell
df -h
```

这将显示磁盘分区的使用情况，包括总容量、已用空间和可用空间等信息。

查看网络使用情况

```shell
netstat -tuln

iftop
```

这些命令将显示有关网络连接和流量的信息。

## 交换内存 swap

```shell
# 查看内存
free -h

# 创建文件
dd if=/dev/zero of=/swap bs=1M count=2048
# 格式化交换文件
mkswap /swap
chmod 0600 /swap

# 开机自启
vi /etc/fstab
# 内容
/swap swap swap defaults 0 0

# 激活
swapon -a
# 关闭
swapoff -a
```
