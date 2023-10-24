---
title: "Sysbench"
type: "docs"
weight: 1
---

[https://github.com/akopytov/sysbench](https://github.com/akopytov/sysbench)

sysbench 是一个模块化、跨平台、多线程、可编写脚本的基准测试工具，基于 LuaJIT，可以执行 CPU、内存、线程、IO、数据库等方面的性能测试，主要用于数据库基准测试，支持 MySQL、Oracle、PostgreSQL。

## 安装

```shell
# Mac
brew install sysbench

# Debian/Ubuntu
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.deb.sh | sudo bash
sudo apt -y install sysbench

# RHEL/CentOS
curl -s https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh | sudo bash
sudo yum -y install sysbench
```

Win10+推荐通过 WSL 安装。

## 命令

内置测试项目：

cpu：这是对 CPU 性能的测试，主要通过执行算术运算，如算质数和圆周率等来测试 CPU 性能。  
threads：该测试用于评估线程调度器的性能，这对于在高负载情况下测试线程调度器的行为非常有用。  
mutex：互斥锁测试是一种评估并发性能和同步机制的测试。  
memory：内存分配及传输速度测试，用于评估系统的内存读写性能。  
fileio：文件 IO 性能测试，用于评估系统在处理外部存储 IO 请求时的性能。  
oltp：数据库性能（OLTP 基准测试），用于评估数据库在处理日常事务处理工作时的性能。

```shell
sysbench cpu --threads=8 run
sysbench memory --threads=8 run
sysbench fileio --threads=8 --file-test-mode=seqrewr prepare
sysbench fileio --threads=8 --file-test-mode=seqrewr run
sysbench fileio --threads=8 --file-test-mode=seqrewr cleanup
```
