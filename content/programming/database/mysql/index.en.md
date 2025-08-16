---
title: "MySQL"
type: "docs"
weight: 1
---

## Chapter 1: Database Fundamentals and Architecture

Concurrency issues: Read-write locks (shared locks, exclusive locks), deadlocks and their handling (timeout unlock or detection followed by transaction rollback)

Transactions: Atomicity, Consistency, Isolation, Durability (ACID properties)

Lock granularity: Table locks, row-level locks (minimum row-level exclusive lock transaction rollback)

Isolation levels: Read uncommitted, read committed, repeatable read, serializable (dirty read, phantom read problems)

MySQL mixed engine transactions: In MySQL transactions are implemented by engines, mixed engines may cause unreliable transactions.

Multi-Version Concurrency Control (MVCC): Row-level locks are not the best choice for implementing transactions, most use MVCC snapshots (snapshots retain records of all changes, this is the basis for transaction commit or rollback). InnoDB engine implements snapshot transactions through incrementing system version numbers.

### Architecture

First layer: Connection handling, authorization authentication, security, etc.;
Second layer: MySQL server, responsible for parsing, analysis, optimization, caching, and all built-in functions, stored procedures, triggers, views, etc.;
Third layer (bottom): Storage engines, such as InnoDB, MyISAM.

### InnoDB Overview

The engine is very complex.

InnoDB's MVCC is implemented by saving two hidden columns after each row record: row creation version number and row deletion version number. System version number (transaction ID).

1. Implements transactions through MVCC, prevents phantom reads through gap lock strategy.
2. Tables built on clustered indexes, high performance for primary key queries, but secondary indexes must contain primary key columns. If primary key columns are large, all other indexes will be large.
3. Predictable read-ahead;
4. Has insert buffer;
5. Supports hot backup.

### Implicit Locking

1. InnoDB automatically adds intention locks, users cannot operate, must obtain intention before locking.
2. For UPDATE, DELETE and INSERT statements, InnoDB automatically adds exclusive locks (X) to involved data sets;
3. For ordinary SELECT statements, InnoDB doesn't add any locks and doesn't conflict with locks;
4. No index or index failure, row locks upgrade to table locks;
5. Range updates add row locks to involved data sets, producing gap locks;

### Explicit Locking

1. Shared lock (read lock, S lock): SELECT * FROM table_name WHERE ... LOCK IN SHARE MODE
2. Exclusive lock (write lock, X lock): SELECT * FROM table_name WHERE ... FOR UPDATE
3. Optimistic lock: Add version number to implement yourself;
4. Pessimistic lock: i.e., exclusive lock;
5. Row lock: i.e., shared lock, exclusive lock;
6. Table lock: LOCK TABLES managed by MySQL server layer, settings don't apply to InnoDB which cannot recognize;

## Chapter 4: Schema and Data Type Optimization

1. Smaller is usually better.
2. Integers have lower operation cost than characters, character sorting is more complex during value comparison.
3. Try to avoid NULL, indexing, index statistics, value comparison more complex and takes space.
4. Integer types: Signed and unsigned use same storage space with same performance, choose as needed. However integer calculations generally use 64-bit BIGINT, even on 32-bit systems.
5. Real number types: FLOAT and DOUBLE use standard floating-point arithmetic for approximate calculations, DECIMAL supports exact calculations but space and computational performance inferior to floating-point. Generally convert minimum units to integers.
6. String types: Starting from version 4.1, columns can customize character sets and collation rules, greatly affecting performance. VARCHAR needs 1-2 extra bytes per row to record length, if table ROW_FORMAT=FIXED stores fixed-length. Variable-length produces fragments when updating, too long MyISAM splits storage, InnoDB splits pages and extra-long stored as BLOB.
7. Enum type: Enums very compact, saved as integer mapping, occupying 1-2 bytes, sorted by integer, suitable to replace string types. For strings that may change, or for association with VARCHAR, should not replace with enum.
8. Time types: TIMESTAMP occupies 4 bytes, DATETIME occupies 8 bytes, both stored as integers. TIMESTAMP uses half the space of DATETIME and ignores timezone, but small time range, only represents 1970-2038.
9. Bit data types: MySQL treats BIT as string type, MyISAM packs all BIT columns for storage, however engines like InnoDB use minimum integer type sufficient, cannot save space. MySQL combines scenarios to return binary ASCII code characters or decimal calculation results, should use BIT type cautiously.
10. Identifier selection: Integer types are best choice, avoid string types, especially MyISAM compresses strings making queries slow. Note random strings like md5, sha1 hashes cause random disk access, clustered index fragmentation, cache invalidation. UUID has certain order, can convert to binary BINARY storage through UNHEX.
11. Special type data: For example IPv4 should not use string type, use 32-bit unsigned integer type. IPv4 decimal points only for easy reading, can convert through MySQL's INET_ATON, INET_NTOA().
12. MySQL schema design traps: Due to server+engine API communication architecture, MyISAM variable-length rows and InnoDB rows always need conversion, cost very high and depends on column count. MySQL limits associations to maximum 61 tables, experience suggests within 12. Enums avoid too many values, can use integers as foreign keys to associate dictionary tables or lookup tables.
13. Normalization and denormalization: Field atomicity, primary key record uniqueness, remove redundancy; appropriate redundancy can avoid association queries and can use triggers to update cached values. Cache tables and summary tables. Denormalization appropriately sacrifices write operation performance but significantly improves read operation performance.

## Chapter 5: Creating High-Performance Indexes

1. Larger data volume, greater index impact on performance.
2. MyISAM indexes reference physical positions of rows, InnoDB indexes reference primary keys.
3. Indexes can be used for ORDER BY operations in queries, for multi-value sorting, according to column order when defining indexes in Schema.
4. Order in multi-column composite indexes very important, MySQL can only efficiently use leftmost prefix columns of indexes, place columns with highest selectivity at front of index. B-Tree cannot use indexes if not searching starting from leftmost column of index, can build indexes with same columns but different order.
5. InnoDB supports "adaptive hash index" for B-Tree indexes.
6. Hash indexes not suitable for scenarios with many hash collisions, suitable for lookup tables being associated.
7. Custom pseudo-hash indexes, create pure numeric CRC32 hash column, add B-Tree index on hash column, search hash column and indexed column when querying.
8. For TB-level data, can build metadata tables recording which table user information is in.
9. Doing operations on indexed where query condition columns causes MySQL not to use indexes.
10. For too long data can select high-selectivity columns, take prefix as prefix index, advantage is small index, disadvantage is ORDER, GROUP cannot use indexes.
11. Clustered indexes store data rows and adjacent key values compactly together. InnoDB clusters data through primary key, without primary key will implicitly define primary key. Clustered index queries fast, but insertions and updates may require page splits, and secondary index access needs two lookups.
12. Covering index means index contains all field values needed for query, very efficient due to not returning to table for rows. B-Tree supports "queries accessing only indexes", i.e., covering indexes.
13. Why MySQL indexes use B+ trees: Few levels help reduce disk I/O, leaf page doubly-linked lists better suited for range queries.

Key points: B+tree, hash, selectivity.

Primary key index
Unique index  
Normal index
Multi-column index (composite index)
Covering index
Prefix index
Prefix compression index (MyISAM only)
Clustered index

Q: Non-clustered index?

Q: Are all indexes based on clustered index?

Q: Table has composite index (a,b,c), query condition a=1,b>2,c=3, why does index on condition c fail?

A: Simply put, this composite index is a B+ tree sorted by field a with b and c relatively ordered. Engine can locate data with a=1 through binary search. b is ordered when a=1 is determined (so b's order is relative), can still use binary search to get all data with b greater than 2. But this data's b field may have many different values, so c field is unordered, cannot use binary search to query data with c=3, so c cannot use index.

## Database Sharding and Partitioning vs Sharding Tables

Partitioned tables and database/table sharding are common database sharding techniques that can improve database performance and scalability. Each has advantages and disadvantages, can choose based on actual situations.

Advantages of partitioned tables:

1. High query efficiency: Partitioned tables can divide data by partition keys, can scan only needed partitions when querying, improving query efficiency.
2. Low management cost: Partitioned tables only need one table, relatively low management cost.
3. Data consistency: Partitions of partitioned tables are in same physical table, relatively good data consistency.
4. Distributed databases can place partitions on different disks, greatly improving performance.

Disadvantages of partitioned tables:

1. Partition key selection quite important, if poorly chosen may cause uneven data distribution, affecting query efficiency.
2. Single partition data volume may still be large, not suitable for single node carrying large amounts of data.

Advantages of database/table sharding:

1. Good scalability: Database/table sharding can distribute data across multiple nodes, can improve database scalability and load balancing.
2. Relatively small data volume per single node, can improve query efficiency.

Disadvantages of database/table sharding:

1. Difficult to maintain data consistency: Nodes in database/table sharding need data synchronization and consistency maintenance, if synchronization not timely or problems occur, may affect query result accuracy.
2. Higher management cost: Database/table sharding requires multiple nodes for management and maintenance, relatively higher management cost.
3. Query efficiency may be lower: When cross-node queries needed, requires cross-node querying and data merging, may affect query efficiency.

In summary, if data volume small, query demands concentrated, high data consistency requirements, using partitioned tables may be more suitable; if data volume large, query demands dispersed, data consistency requirements not so high, using database/table sharding may be more suitable.

### Why MySQL Partitioned Tables Not Commonly Used

[Why are MySQL table partitions not very commonly used? - Zhihu](https://www.zhihu.com/question/378240599)

The reason for using partitioned tables in single MySQL instances (not distributed databases) is mainly because single table data volume too large causes indexes to be too large, thus reducing query performance.

Consider worst case of huge single table with large primary key fields, let's calculate height of main table b+ tree.

Example 1. Say single table 10 billion rows, each row averages 1000 bytes storage space, 16KB page size, then main table leaf nodes occupy about 10TB space, about 700 million pages. Assume primary key takes large space causing internal nodes each index row averaging 256 bytes, so each internal node page stores 64 index rows. Then main table b+ tree height is

`1 + ceiling(lg(64, 700000000)) = 6`

If primary key index chosen very poorly causing internal nodes each index row occupying 512 bytes, then main table b+ tree height is

`1 + ceiling(lg(32, 700000000)) = 7`

Example 2. Assume above data only has 10 million rows, then using same method can get main table b+ tree heights of 4 (256-byte primary key index) and 5 (512-byte primary key index) respectively --- height differs by only 2.

If main table in Example 1 really made 1000 partitions then each partition table would be main table of Example 2, so through partitioning, can reduce each tree search by 2 page fetches (very likely from buffer pool, otherwise system performance unusable). This difference indeed causes Example 2 query performance improvement, but difference actually not large.

Additionally, after partitioning another performance advantage is dispersing root node access, thus improving concurrent performance. However know that b+ tree traversal doesn't hold transaction locks on internal node pages, only needs to briefly hold read latch on root node page, so except for page splits (low frequency operations) traversals can execute concurrently.

In summary, I think theoretical upper limit of performance improvement from single-machine table partitioning very limited, estimated within 10%, slightly fluctuating with data and query characteristics, varying with database system implementations. Interested students can use postgresql or mysql to do experimental comparison. In different database system implementations, actual performance of partitioned tables compared to single tables varies in improvement, may not even improve, may even decrease.

Details of mysql partitioned tables as follows.

First, if many tables do partitioning, will cause very many files in mysql innodb data directory (e.g., 1000 table partitions will produce 2000 files), thus reducing operating system filesystem work efficiency. And due to mysql's limit on number of open table files (this limit can be manually modified but also limited by available OS resources) causing opened tables to be repeatedly evicted and reopened, thus reducing performance of all queries.

In early versions of mysql5.7, partitioned table implementation had poor performance, about 10% performance decrease compared to single tables with same data volume. Later in mysql5.7.19 optimizations were made, can check [this bug](https://link.zhihu.com/?target=https%3A//bugs.mysql.com/bug.php%3Fid%3D83470) at [http://bugs.mysql.com](https://link.zhihu.com/?target=http%3A//bugs.mysql.com). But even after this bug fix, partitioned tables still have about 5% performance decrease compared to single tables with same data volume (8 partitions, 100GB data volume, sysbench oltp test case).

In above example if single table occupies 10TB space, then single server node's computing resources (memory, cpu, storage) probably cannot support business operations, so most likely still need database/table sharding. If single table only has within 1TB space then completely no need for table partitioning. That is, doing table partitioning in single-node mysql instances not necessary and won't have obvious performance advantages.

For database/table sharding, using partitioned tables to implement sharding is common method used by some simple sharding middleware, also quite effective. In Kunlun distributed database, we implement database/table sharding in compute nodes, always use single tables for storage in storage nodes without partitioned tables, based on above reasons of mysql partitioned table performance decrease.

## Sorting and Grouping

For sorting and grouping order, should group first then sort. This is because grouping operations divide data by grouping keywords, then sort each group. If sorting operations done first, will cause data arrangement to change, leading to incorrect grouping results.

## Three Normal Forms

First Normal Form (1NF): Atomicity, fields cannot be further divided;
Second Normal Form (2NF): Dependency, on basis of satisfying first normal form, all non-primary key fields depend on primary key;
Third Normal Form (3NF): Transitivity, on basis of satisfying second normal form, no transitive dependencies exist between non-primary key fields.

## Others

utf8_general_ci case insensitive
utf8_general_cs case sensitive

binlog, binary, records write operations, mainly used for master-slave replication and data recovery.
redolog, to improve transaction processing performance, write log first then write disk.
undolog, MVCC failure rollback, records reverse SQL statements.
