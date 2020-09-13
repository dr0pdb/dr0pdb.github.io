---
title: LevelDB Internals - A Deep Dive
date: 2020-09-11 23:30:00 +0530
categories: [Database-Systems, Storage-Engine]
tags: [dbsystems, leveldb, kv-stores]
pin: true
---

Hello! Welcome to the deep dive series on the [LevelDB](https://github.com/google/leveldb) internals. LevelDB is an embedded key-value library written in C++ and released open source by Google.

Since it's open source, I had the chance to go through the codebase of the library. In this blog series, I'll be discussing some of the internal implementation details that I found interesting.

## Disclaimer
I have never worked on the LevelDB codebase. Here I am just sharing my understanding of the implementation details which I found worth sharing. I am in no way involved in the LevelDB project as of now.

## Motivation
I am planning to build my own toy distributed database from scratch. The first step in building the database is to build a storage engine. Having gone through some of the text books such as the `Database Internals` by Alex Petrov, I decided to read some of the existing storage engines to understand some of the implementation details. LevelDB is the first one that I looked at. I felt it would be better if I shared my learnings through articles as it might be helpful for fellow learners.

## Overview
LevelDB is an key-value storage library. It doesn't have builtin client-server support although it's easy to wrap the functionality around the library. The keys and values both can be arbitrary byte arrays. It stores the data sorted by key in a persistent way on disk. It also supports snapshots which enables the users to guarantee a consistent view of data despite the future changes. LevelDB will keep the snapshot untill it's explicitly released by the user process.

Internally LevelDB uses **Log Structured Merge**(LSM) storage mechanism to store the data on disk. It also maintains a memtable and a log file which gets all the latest changes. Once the log file reaches a pre-determined size, it's sorted and flushed to the disk as an immutable *Sorted String Table*(SST). 

Writes are usually very fast since they involve adding/updating an entry in the memtable and addition of an entry in the log file. On the other hand, the speed of a read operation depends on whether the data is already stored in the memtable or not. If the data is present in the memtable, it is swiftly returned to the user. In the other not so ideal case, the data has to be read from the SSTs stored on the disk. Because of the immutable property of the SSTs, multiple SSTs have to be read and the final value of the data has to be calculated after applying all the logged updates. I'll be discussing the LSM storage mechanism in detail in a separate article.

Since the stored data is immutable, there are usually redundant copies of the same data which increases the read amplification. Hence the db system needs to discard them in order to improve the read latency as well as reclaim the disk space. This process is called *Compaction*. I'll be going over them in detail in one of the future articles.

The DB system only allows one process to interact with a particular database at a time. The process could have multiple threads and the db system uses synchronization primitives such as Mutexes and Condition variables to ensure data consistency.

With this, I conclude a high level overview of Leveldb. I'll be going through them in much detail in the future articles. The next article will describe the public API and the Log structured Merge(LSM) storage mechanism used in LevelDB in detail.

Thanks for reading!
