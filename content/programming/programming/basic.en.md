---
title: "Basic Knowledge"
type: "docs"
weight: 1
---

## Process, Thread, Coroutine

A process is the smallest unit for resource allocation by the operating system, and a thread is the execution unit within a process, the smallest unit for CPU scheduling. Process = train, thread = carriage.

A coroutine is a lightweight thread that runs inside a thread, like a function call with context. Coroutines are suitable for I/O-intensive applications, switching during I/O blocking and other situations, improving parallel connections and I/O throughput while making full use of CPU performance.

Threads are usually preemptively scheduled, where the operating system can interrupt thread execution when a time slice expires and switch to run another thread. Coroutines are controlled by the program itself and are affected by the scheduler, such as Golang's cooperative scheduling, but since they are parasitic within threads, they inevitably are affected by the thread.

## Bit, Byte, Encoding

One byte is 8 bits, int8 is 1 byte, int32 is 4 bytes.

GBK encoding takes 2 bytes for Chinese characters; UTF-8 is variable-length encoding, commonly used Chinese characters take 3 bytes, some take 4 bytes.

UTF8MB4 (most bytes 4) emojis are 4 bytes.

## Random I/O, Sequential I/O

Random I/O means data access is unordered, where each access request can be made in any order; Sequential I/O means data access is ordered, where each access request follows a certain sequence.

For mechanical hard drives, since the head needs to move to a specific position to read or write data, random I/O operations are more difficult while sequential I/O operations are more efficient. For solid-state drives, since storage units are flash memory particles that can be read and written directly through control units, both random I/O and sequential I/O operations are efficient.

The following situations typically involve random I/O:

1. Reading or writing a file requires random access to specific data blocks in the file.
2. Updating or deleting data in a database requires random access to specific data records in the database.

The following situations typically involve sequential I/O:

1. Reading or writing a file requires sequential access to each data block according to the file's storage order.
2. Querying or inserting data in a database requires sequential execution of query or insert operations according to the database table structure.
3. In network communication, sending and receiving data requires sequential transmission and reception of each data packet according to protocol specifications.

## Common Locks

Shared lock: Allows multiple threads to simultaneously access shared resources, but other threads cannot modify the resource, only read it.
Exclusive lock: Only allows one thread to access shared resources, other threads cannot access the resource.
Pessimistic lock: Locks before accessing shared resources to ensure other threads or processes cannot modify shared resources simultaneously. This lock mechanism is simple and effective but reduces concurrency performance since each thread needs to wait during locking and unlocking.
Optimistic lock: Doesn't lock, but after modifying shared resources, compares the modified value with the cached value. If they don't match, it means other threads have modified the shared resource, requiring the operation to be re-executed.
Spin lock: A locking method where when a thread tries to acquire a lock that's already held by another thread, it continuously checks whether the lock is released until it acquires the lock.
Semaphore lock: A special lock mechanism used to control multiple threads' access to shared resources. Semaphores have a counter - when positive, it indicates available free resources; when negative, it indicates all resources are occupied.

Optimistic and pessimistic locks each have pros and cons. For scenarios with more writes than reads, pessimistic locks are more suitable; for scenarios with more reads than writes, optimistic locks are more suitable.

## Four Necessary Conditions for Deadlock

Mutual exclusion: A resource can only be used by one process at a time;
Hold and wait: When a process is blocked by requesting resources, it holds already acquired resources;
No preemption: Resources already acquired by a process cannot be forcibly taken away before completion;
Circular wait: Processes form a circular wait relationship for resources;
