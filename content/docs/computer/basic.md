---
title: "计算机基础"
weight: 1
---

**进程、线程、协程**

1. 进程是操作系统进行资源分配的最小单位，线程是进程内的执行单元，是 CPU 调度的最小单位。进程=火车，线程=车厢。
2. 协程是一种轻量级线程，在线程内部运行，像是带上下文的函数调用，适合 IO 密集型。真正的异步靠多核，协程是 IO 异步，可以充分利用 cpu 处理能力，在 IO 等待时切换，提高 IO 吞吐量。
3. 线程通常是抢占式调度，操作系统可以在任何时候中断线程的运行，转而运行另一个线程，通常基于时间片实现。协程通常是非抢占式调度，由程序而不是操作系统控制，比如 golang 的协作式调度。
4. Go 支持多线程，可以利用多核 CPU；PHP 单线程，但是可以使用多进程。

**位 bit、字节 byte、编码**

一个字节 8 位，int8 是 1 字节，int32 是 4 字节。

汉字 GBK 编码占 2 字节，utf8 是变长编码，常用汉字占 3 字节，部分 4 字节。

utf8mb4（most bytes 4）的 emoji 是 4 字节。

**随机 IO、顺序 IO**

随机 IO 是指对数据的访问是没有顺序的，即每个访问请求都可以按照任意顺序进行。例如，文件系统中的读写操作通常需要随机 IO，因为文件中的数据可以任意访问。

顺序 IO 是指对数据的访问是有顺序的，即每个访问请求都是按照一定的顺序进行。例如，在磁盘中读取一个文件时，需要按照文件的存储顺序依次读取每个数据块。

对于机械硬盘，由于磁头需要移动到指定的位置才能读取或写入数据，因此随机 IO 操作比较困难，顺序 IO 操作比较高效。对于固态硬盘，由于存储单元是闪存颗粒，可以直接通过控制单元读写数据，因此随机 IO 和顺序 IO 操作都比较高效。

以下情况通常是随机 IO：

1. 读取或写入一个文件时，需要随机访问文件中的某个数据块。
2. 在数据库中更新或删除数据时，需要随机访问数据库中的某个数据记录。

以下情况通常是顺序 IO：

1. 读取或写入一个文件时，需要按照文件的存储顺序依次读取或写入每个数据块。
2. 在数据库中查询或插入数据时，需要按照数据库中的表结构顺序依次执行查询或插入操作。
3. 在网络通信中，发送和接收数据时需要按照协议规定的顺序依次发送和接收每个数据包。

**死锁的四个必要条件**

互斥：一个资源每次只能被一个进程使用；
请求与保持：一个进程因请求资源而阻塞时，对已获得的资源保持不放；

不剥夺：进程已获得的资源，在末使用完之前，不能强行剥夺；

循环等待：若干进程之间形成一种头尾相接的循环等待资源关系；

**常见锁**

共享锁：允许多个线程同时访问共享资源，并且其他线程不能修改该资源，只能读取。

排它锁：则只允许一个线程访问共享资源，其他线程无法访问该资源。

悲观锁：保守，访问共享资源之前先加锁，以保证其他线程或进程无法同时修改共享资源。这种锁机制简单有效，但是会导致并发性能下降，因为每个线程或进程在加锁和解锁之间都需要等待。

乐观锁：假设多个线程或进程不会同时访问共享资源，因此不需要立即加锁。而是在对共享资源的修改完成后，将修改后的值与缓存中的值进行比较，如果两者不一致，则说明其他线程或进程已经修改了共享资源，此时需要重新执行操作。

自旋锁：一种加锁方式，当一个线程试图获取锁时，如果该锁已经被其他线程占用，则该线程会一直循环检查该锁是否被释放，直到获取到锁为止。

信号量锁：一种特殊的锁机制，它可以用于控制多个线程对共享资源的访问。信号量有一个计数器，当计数器为正数时，表示还有空闲的资源可供使用；当计数器为负数时，表示资源已经全部被占用。

乐观锁和悲观锁各有优缺点，对于写多读少的场景，悲观锁比较适合；而对于读多写少的场景，乐观锁比较适合。