---
title: LevelDB Internals - Log Structured Merge Tree Storage
date: 2020-10-26 15:00:00 +0530
categories: [Database-Systems, Storage-Engine]
tags: [dbsystems, leveldb, kv-stores, lsm]
---

Hello! Welcome to the second post in the deep dive series on [LevelDB](https://github.com/google/leveldb) internals. In this post, I'll be talking about the Log Structured Merge(LSM) Tree storage mechanism used to store key value pairs persistently on the disk.

## Disclaimer
I have never worked on the LevelDB codebase. Here, I am just sharing my understanding of the implementation details which I found worth sharing. I am in no way involved in the LevelDB project as of now.

## Introduction
LSM tree is an immutable storage structure. It doesn't allow modifications to the existing data files: data files are written once to the disk and never updated in the future. In the case of updates for a data record, new records are appended to a new data file. It means that in order to read the value of a data record, multiple data files need to be read and combined. In contrast, mutable data storage structures such as B-trees do inplace updates. I'll be using B-tree as an example of a mutable data storage structure in order to compare and contrast the two mechanisms.

##### LSM Tree vs B-Tree
Mutable data storage structures are read optimized. In order to read a data file, you traverse the B-tree, locate the data block and return the value. Similarly for writes, the location of the record is determined by traversing the B-tree and the value is added/updated at the appropriate location. On the other hand, the immutable structures are write optimized. Writes can go straight to the end of the data file. It saves the effort of locating the appropriate location before writing. As a consequence of the append only structure, the read performance is hampered because now in order to read, multiple data records have to retrieved and reconciled.

Apart from optimized write performance, LSM trees and immutable structures in general greatly simplify concurrent access. Since the data files are immutable by design, there is no need to hold synchronization constructs to safeguard the data integrity. On the other hand, mutable storage structures employ locks and latches to safeguard data integrity. This concept of using immutable variables is widely employed in functional programming languages such as Erlang, Haskell, F# and many more.

Apart from the inferior read performance, LSM trees also require more disk space in comparison to the B-Tree. This means that the db system has to periodically reconcile the data blocks for a particular record and merge them into one. The newly merged data record is written in a new data file at a new location on the disk. This process is called *Compaction*. After the compaction process, the space occupied by the older data files can be reused for future storage. As you might have noticed, this means that a data record can be written multiple times to disk: once when it is written by the db user and then everytime the data block containing the record is compacted. This is an extra burden on the db system in comparison to B-Trees and is called *Write Amplification*. In order to account for the inferior read performance, LSM trees utilize a memory resident table called *Memtable*. I'll discuss it in detail in future posts.

## Components
LSM Tree Storage structure consists of the following components:

1. Log file
2. Memtable
3. Sorted String Table(SST)

##### Log file
The log file is the current data file that is under use by the db system. All of the recent updates go to this log file. In order to speed up the read operations, a copy of this log file is maintained in the memory as the Memtable.

Once the log file reaches a predefined size, it is sorted and flushed as an immutable SST on the disk. In order to do so, first a new log file is created which becomes the target for all the new writes. The old log file's contents are flushed to disk. It is kept alive as long as its contents are being flushed to disk. After the flushing is complete, the old log file is discarded and the space can be reclaimed for future use.

##### Memtable
Memtable is a memory resident table that contains all of the recent writes in memory. It greatly helps in improving the read performance of the database system. Memtables are usually implemented using data structures like [Skip list](https://en.wikipedia.org/wiki/Skip_list).

##### Sorted String Table (SST)
The SSTs are the component containing the key-value pairs that is persisted on the disk. As discussed earlier, a log file when full is converted to a SST and persisted on the disk. Once written, the SSTs are immutable and hence suitable for concurrent access without any synchronization. There are multiple SSTs in a database and their numbers grows as the amount of data stored increases. Databases such as LevelDB or [Apache Cassandra](https://cassandra.apache.org/) store SSTs in a hierarchical tree like structure. SSTs can have overlapping keys some of which will be stale data values.

In order to tackle the growing number of SSTs and reclaiming the storage occupied by stale data values, the db system periodically reconciles them through a process called *Compaction*. I'll go over the process in upcoming posts.

SSTs usually also contain some metadata about the data they store such as the key range they contain. This is useful in determining whether a key could exist in the SST or not. Often additional optimizations are also done for the same by using data structures such as Bloom filters.

## Operations
In this section, I'll discuss how the above discussed components are combined to fulfill the basic GET and SET operations on the database.

#### SET
The SET operation takes a key-value pair and writes it to the database. Following steps are taken to do that:

1. A log entry is written in the log file.
2. The value is written/updated against the key in the memtable.
3. Acknowledgement is sent to the client.

Additionally, if the log file reaches the pre-determined size, it is sorted and stored on the disk. Moreover, based on the compaction conditions, the compaction process could also get triggered in the background.

![LSM SET operation](/assets/img/lsm-tree/lsm-set.png)

#### GET
The GET operation takes a key and returns the value associated with that key from the database. In the case the key doesn't exist, a suitable error is sent.
Following steps are taken to do that:

1. The key is searched in the Memtable. If the key is found, the value is returned directly from it.
2. Otherwise, the SSTs are searched to find the most recent value stored for the given key.
3. The fetched value is returned to the client.

![LSM GET operation](/assets/img/lsm-tree/lsm-get.png)

The above steps are extremely simplistic and there is a lot of room for optimizations. For eg. Level DB stores the recently fetched SSTs in an in-memory cache called `TableCache`. In that case, the cache is checked first to see if the SST that we're looking to check from is already in the memory or not.

With this, I conclude a high level overview of LSM storage. I'll be going through the memtable and SSTs in more detail in upcoming posts.

Thanks for reading!
