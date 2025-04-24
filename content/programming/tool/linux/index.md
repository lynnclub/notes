---
title: "Linux"
type: "docs"
weight: 1
---

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

# 获取用户id与组id，不指定用户默认当前用户
id root
# 只取用户id
id -u root
# 只取组id
id -g root
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

### 查看进程

```shell
# BSD风格的展示方式
ps aux
# 过滤
ps aux | grep firefox
# 查看内存排名前10的进程
ps aux --sort=-%mem | head -n 10
# 查看CPU排名前10的进程
ps aux --sort=-%cpu | head -n 10

# 完整信息
ps -ef
# 查看进程启动时间
ps -p <进程ID> -o lstart
```

### 杀死进程

```shell
# 按名称查找进程再杀死
ps aux | grep "x" | grep -v grep | awk '{print $2}' | xargs --no-run-if-empty kill -9
```

### 查看 CPU、内存

```shell
top

# 查看特定进程
top -p pid
# 匹配指定进程名
top -p $(pgrep -d ',' -f 'process_name')
```

显示进程维度的 CPU、内存利用率信息。

参数：

-c 显示进程详细名称
-d 刷新间隔，秒，默认 5 秒  
-n 刷新几次，与-b 配合使用  
-p 观察指定 pid

还可以类似 vi/vim，输入命令：

? 显示支持的命令
P 按 CPU 排序
M 按内存排序
N 按 pid 排序
T 累计时间排序
k 按 pid 杀死进程(-9)
r 修改优先级
q 退出（等于 ctrl+c）
c 显示进程全名

```shell
lscpu

cat /proc/cpuinfo
```

### 查看内存

```shell
free -h
```

这将显示总内存、已使用内存、可用内存等信息。

### 查看网络

```shell
netstat -tuln

iftop

# 端口
nc -vz -w 2 192.168.0.107 8080
telnet 192.168.0.107 8080
nmap -p 8080 192.168.0.107
```

这些命令将显示有关网络连接和流量的信息。

### 查看硬盘

```shell
df -h

# 查看当前目录中各个文件和子目录的大小
du -ah

# 显示当前目录及其直接子目录大小
du -h --max-depth=1 ./

# 查看当前目录与文件大小
du -sh ./* | sort -rh

# 查看io状态
iostat -d -m 1 10

#-d 选项表示显示磁盘统计信息。
#-m 选项表示以 MB 为单位显示。
#1 表示每隔 1 秒更新一次。
#10 表示显示 10 次更新。

# 非常强大的Linux性能监控工具，由Nigel Griffiths开发，可以用来收集和显示Linux系统的性能数据，包括CPU、内存、磁盘I/O、网络、文件系统以及进程信息等
apt install nmon
yum install nmon

# top命令的一个增强版
brew install htop
```

这将显示磁盘分区的使用情况，包括总容量、已用空间和可用空间等信息。

### 查看文件

```shell
# 查看文件尾部最后几行内容
tail -n 20 filename
# 监视文件尾部内容变化
tail -f filename
```

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

## 端口扫描

```shell
nmap 192.168.0.1 80
nmap -p 1-65535 10.0.0.1
```

nmap 图形界面软件 [https://nmap.org/download.html](https://nmap.org/download.html)

## 信号

SIGHUP 挂起（hangup），当终端关闭或者连接的会话结束时，由内核发送给进程  
SIGINT 中断（interrupt），通常由用户按下 Ctrl+C 产生，进程接收到信号后应立即停止当前的工作  
SIGQUIT 退出（quit），通常由用户按下 Ctrl+\ 产生，进程接收到信号后应立即退出，并清理自己占用的资源  
SIGTERM 终止（terminate），这是一个通用信号，通常用于要求进程正常终止  
SIGFPE 在发生致命的算术运算错误时发出，如除零操作、数据溢出等  
SIGKILL 立即结束程序的运行  
SIGALRM 时钟定时信号  
SIGBUS SIGSEGV 进程访问非法地址

## time

time 是一个用于测量命令执行时间的简单工具。它可以显示命令的总执行时间、用户 CPU 时间和系统 CPU 时间，帮助开发者优化性能。

```shell
# -o 选项输出到指定文件
# -a 选项用于追加而不是覆盖
time -a -o output.txt ls -l
# 输出
real    0m0.005s
user    0m0.003s
sys     0m0.002s
```

real：实际时间（墙钟时间），即命令执行的总耗时。  
user：用户态 CPU 时间，即 CPU 在用户模式下为该命令消耗的时间。  
sys：内核态 CPU 时间，即 CPU 在内核模式下为该命令消耗的时间。  

## strace

strace 是一个强大的 Linux 调试工具，用于跟踪用户空间程序与内核交互的系统调用和信号。

```shell
# 跟踪程序执行
strace ./your_program
# 跟踪已有进程
strace -p <pid>
# 统计系统调用
strace -c ./your_program
strace -c -p <pid>
# 显示每个系统调用的时间花费
strace -T ./your_program

# 仅跟踪特定系统调用
strace -e trace=all,open,read,write,network,file ./your_program
# 排除特定系统调用
strace -e '!open,read' ./your_program
# 跟踪信号
strace -e signal=all,SIGTERM ./your_program
# 分析动态链接库
strace -e trace=process ./your_program
# 单项记录
strace -p 24878 -e connect -s 256 -o strace.log

# 输出到文件
strace -o strace.log ./your_program
# 默认字符串输出长度为 32，可调整
strace -s 512 ./your_program
# 显示相对时间
strace -r ./your_program
# 显示详细信息
strace -v ./your_program

# 跟踪子进程
strace -f ./your_program
# 只跟踪主进程
strace -ff -p <pid>
# 跟踪特定线程
strace -p <tid>
# 仅显示与指定文件路径相关的调用
strace -P /path/to/file ./your_program
# 跟踪执行路径
strace -y ./your_program

# 输出进程明细到文件
sudo strace -f -T -o output.txt -p 15527
```

strace 只跟踪系统调用，不跟踪用户态任务。

需要系统调用的任务

1. 文件 I/O：如 open、read、write。
2. 网络通信：如 socket、connect、send、recv。
3. 内存分配：动态分配大块内存可能调用 mmap 或 brk。
4. 进程管理：创建、销毁或同步进程时使用 fork、execve 或 waitpid。
5. 设备访问：如磁盘、打印机、摄像头等外设的操作。

不需要系统调用的任务

1. 纯计算：如数学运算、逻辑判断、数据处理等。
2. 内存操作：用户态程序直接访问分配的内存（如 malloc 返回的内存块）无需系统调用。
3. 本地缓存：例如，大多数应用程序会缓存数据以减少对磁盘或网络的访问。
4. 字符串处理：像 strcpy、strlen 这样的操作完全在用户态执行。

## perf

perf可以用于性能分析、性能调优、代码分析等方面。它可以用于查看 CPU 性能计数器，跟踪系统调用、内核函数执行、上下文切换等。

```shell
# 查看版本
perf version
# 列出所有支持事件
perf list

# 查看进程性能
perf top -p <pid>
# 查看线程性能
perf top -t <tid>
# 查看指定线程性能
perf top -p <pid> -t <tid>

# 分析CPU性能
perf stat <command>
# 分析多核性能
perf stat -a <command>
# 监控特定事件
perf stat -e cycles,instructions,cache-misses <command>
# 查看KVM虚拟化环境的性能
perf kvm stat <command>

# 跟踪系统调用
perf trace
# 跟踪特定的系统调用
perf trace -e 'syscalls:sys_enter_read'
# 通过BPF跟踪用户级应用
perf trace -e 'bpf:perf_event' <command>

# 记录性能数据，默认生成 perf.data 文件
perf record <command>
# 生成堆栈调用
perf record -g <command>
# 指定时间窗口分析，-F设置采样频率，-a表示全系统采样
perf record -F 99 -a
# 生成 Flame Graph
perf record -g -p <pid>
perf script > out.perf
./flamegraph.pl out.perf > flamegraph.svg
# 查看记录数据
perf report
# 查看调用图
perf report --call-graph
# 查看指定文件
perf report -i perf.data
# 查看文件堆栈信息
perf script
```
