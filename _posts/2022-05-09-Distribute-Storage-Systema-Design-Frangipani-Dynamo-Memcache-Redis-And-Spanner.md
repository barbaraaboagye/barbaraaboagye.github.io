---
layout:     post
title:      Distribute Storage System Design Part1 -- Frangipani & Dynamo
subtitle:   
date:       2022-05-09
author:     "Nickolas"
header-img: 
---

我们可以通过分析经典的系统, 理解分布式系统设计. 比如, Frangipani, Dynamo, Memcache, Redis Cluster, Spanner. 这些系统在[Paper Trail](https://www.the-paper-trail.org/page/reading-list/), [MIT Distributed System](https://pdos.csail.mit.edu/6.824/)中都有分析和介绍, 系统的论文也可以在[阅读清单](https://nickolashu.github.io/2022/03/27/reading-list-of-distributed-system/)中找到. 本文试图把这些系统放在一起, 对比各自不同的需求场景, 理解系统作者背后的选择权衡, 从而在面对现实的分布式问题中, 做出更好的选择.

## 1 Frangipani.  Serializable FileSystem

Frangipani是一个分布式的文件系统(1997), 为了实现多人在线编辑文件的场景. 为了简化使用场景, Frangipani不支持多人同时在线编辑同一个文件, 即独占式的写入. 

在这个场景下, 需要满足两方面的需求:

1) 每个workstation(WS)写的数据不能丢, 数据不能相互覆盖, 同时不存在写后读不到的逻辑问题.
2) workstation的读写性能很好, 就像在本地编写一样.

设计分析:

1. 低延时. 因此需要在workstation缓存一份数据, 在本地修改数据, 再提交.
2. 强一致性. WS读写独占, 相当于是串行化的隔离级别(Serializable), 通过一个Lock Server, 操作前加锁, 操作完成后释放锁.

可见, Frangipani需要一个cache coherence protocol, 详细过程如下:

```
Frangipani's coherence protocol (simplified):
  lock server (LS), with one lock per file/directory
    file  owner
    -----------
​    x     WS1
​    y     WS1
  workstation (WS) Frangipani cache:
    file/dir  lock  content
    -----------------------
​    x         busy  ...
​    y         idle  ...
  if WS holds lock,
​    busy: using data right now
​    idle: holds lock but not using the cached data right now
  workstation rules:
​    don't cache unless you hold the lock
​    acquire lock, then read from Petal(storage system)
​    write to Petal, then release lock
  coherence protocol messages:
​    request  (WS -> LS)
​    grant (LS -> WS)
​    revoke (LS -> WS)
​    release (WS -> LS)
```

一个完整的获取锁的过程, 当WS1想要读或者写时, 先向LS reqeust lock. LS 如果对应的文件没有加锁, 标记文件为占有状态, 所有者为WS1. 否则向锁的所有者WS2 grant锁, 等待WS2 写完成, 释放锁. WS1 被LS revoke. 

占有状态分为busy和idle状态, 区分WS的读和写. idle状态(只读)的锁, 不存在写一致性问题, 可以直接抢占锁.



Frangipani是一个典型的CP系统, 牺牲可用性, 达到系统的强一致性和缓存的分区. 通过一个Lock Server节点, 实现各个节点对资源的顺序读写, 达到整体的强一致性.

## 2 Dynamo 去中心化高可用的KV存储
Dynamo是Amazon设计的去中心化的高可用的Key-Value存储. 在Amazon的场景下, 

1. 大规模, 可靠性. 支持Tens of thousands of servers and network components, 兼容服务的出错异常, 和数据丢失的异常.
2. 扩展性,  highly scalable to support continuous growth. 易于扩展, 切人工运维成本低.
3. 主键存储. 简化的数据模型, 只支持主key模式的数据存储. 降低RDB带来的额外开销. 
4. 高性能, 低延迟. 

为了满足以上需求,

Available. Data is partitioned and replicated using consistent hashing. Gossip based distributed failure detection and membership protocol.
Consistency. Consistency is facilitated by object versioning, The consistency among replicas during updates is maintained by a quorum-like technique and a decentralized replica synchronization protocol.
Simplicity. Dynamo is a completely decentralized system with minimal need for manual administration. Storage nodes can be added and removed from Dynamo without requiring any manual partitioning or redistribution.
Partition Algorithm

Put:
find first node of preference list as the coordinator
coordinator generates vector lock and write locally
    write to buffer first
    writer thread periodically sync to disk
  send to N highest-ranked node
if at least W-1 respond, success
else failed

Get:
find first node of preference list as the coordinator
send to N highest-ranked node
waits for R responses before returning the result to the client.
  read from buffer first,
  then from the disk.
if all version are the same, success
else returns all the versions it deems to be causally unrelated
The divergent versions are then reconciled and the reconciled version superseding the current versions is written back.



## 3 Memcache 分布式HashTable (DHT)

Design needs: 1) ner real-time communication. 2) aggregate content on-the-fly from multi sources 3) able to access and update popular shared content 4) scale to millions qps. 

Read-heavy workload and wide fan-out.

Design principle: 

1) demand-filled lool-aside cache. Choose to delete cache data instead of update, cause of idempotent.

2) Memecache server do not communicate with each other. Try to embed the complexity of the system into a stateless client rather than in the memcache server.

   <img src="http://nickolashu.github.io/img/look-aside cache.png" alt="look-aside cache" style="zoom:50%;" />

Latency:

* Parallel request and batching: DAG represent dependcy of data.

* Client server communication:

​	get - udp for reduce latency and overhead - network failure 0.25% (80% late or drop packet) - treat as error - skip fill memcached

​	put,delete -  tcp for 可靠性

* Incast congestion: sliding window to limit incast congestion. 

​	Reason client request large number of keys, response arrive all at same time, response overwhelm rack and cluster switches.

Reducing load:

* Leases:
  * lease is a 64-bit token bound to the specific key the client originally requested.
  * stale values: happens when concurrent update to memcache get reordered. So when a key is deleted, its value is transfered to a data structure that holds recently deleted items. A get request can return a lease token or data which ic marked as stale.
  * thundering herds: request for a key's value within 10 secounds of a token being issued result in a special notification telling the client to wait a short amount of time. When the client retry the request, the data is often present in the cache.

Handle failures:

Dedicate a small set of machines(1%), to take over the responsiblities of a few failed servers, named Gutter.

When a memcached client receives no response to its get request, the client assumes the server has failed and issues the request again to the Gutter pool.

Regional Invalidation: MySQL commit log - batch deletes to McSqueal - unpack to individual memcache server

<img src="http://nickolashu.github.io/img/memcache invalidation@2x.png" alt="memcache invalidation@2x" style="zoom:50%;" />

Cold Cluster Warmup: Allow "cold cluster" retrieve data from "warm cluster" rather than a persitent storage.

​	Race conditions: cold cluster database update - simultaneously request stale data from warm cluster.

​		solution: 

​				Client:

​					when a miss id detected, 

​					client re-request the key from the warm cluster 

​					and `adds` it into the cold cluster.

​					Cold Cluster: all deletes to colud cluster issued with a two second hold-off (which reject add)

​					the failure of `add` indicates that the newer data is avaliable.

​					client refetch on the database.

​	Close time: When cold cluster's hit rate stabilizse.

ACross Region Consistency: 

​	MySQL master - replicas. rely on MySQL replication to keep replica databses up-to-date. 

<img src="http://nickolashu.github.io/img/overall architecture.png" alt="overall architecture" style="zoom:50%;" />

​	Benifits: local memcached server + local MySql provides low latency; avoids reace condition in which an invalidation arrives before the data replicated from the master db region.

​	Problem: replica may lag behind the master database. reqeust may get the stale data be

​	Solution: provide best-effort eventual consistency. Use `remote marker` to minimize the probability of reading stale data. the marker indicates that data in the local replica database are potentially stale, and the query should be redirected to the master region.

  1. befrore client update key k, set a remote marker in the region

  2. client perform write to the master db.

  3. client deletes k in the local memcache cluster.

  4. client get cache miss, check the remote marker of k, decide whether direct query to master/local databese.

     Trade consistency with speed.

​	

## 4 Redis

- Availability
master replica pattern
support async(default) and sync replication.

replica PSYNC -> master <Replication ID, offset>
   查询backlog
   计算incremental update
   or 查询不到 full sync
      background process RDB file
      buffer commands
      load RDB file, send all to replica

A replication ID basically marks a given history of the data set. Every time an instance restarts from scratch as a master, or a replica is promoted to master, a new replication ID is generated for this instance.

why a replica promoted to master needs to change its replication ID after a failover: it is possible that the old master is still working as a master because of some network partition: retaining the same replication ID would violate the fact that the same ID and same offset of any two random instances mean they have the same data set.



Redis Cluster 

Hash Slot: 

Every node is connected to every other node, The client is in theory free to send requests to all the nodes in the cluster, getting redirected if needed.

https://medium.com/opstree-technology/redis-cluster-architecture-replication-sharding-and-failover-86871e783ac0#:~:text=Redis%20Cluster%20is%20an%20active,a%20subset%20of%20those%20slots
https://redis.io/docs/manual/replication/

- Consistency
  https://www.allthingsdistributed.com/2021/11/amazon-memorydb-for-redis-speed-consistency.html

  Redis Cluster uses asynchronous replication between nodes, and **last failover wins** implicit merge function. This means that the last elected master dataset eventually replaces all the other replicas. There is always a window of time when it is possible to lose writes during partitions. 

  防止split brain: 奇数个master, 每个master两个replica.

- Persistence
https://redis.io/docs/manual/persistence/

- Performance
-- IO MultiPlexing - epoll/kqueue
Redis event library https://github.com/redis/redis/blob/99ab4236afb210dc118fad892210b9fbb369aa8e/src/ae.c
https://austingwalters.com/io-multiplexing/

-- Single Thread

-- Pipelining
https://redis.io/docs/manual/pipelining/#:~:text=Redis%20is%20a%20TCP%20server,way%2C%20for%20the%20server%20response.

https://aws.amazon.com/redis/



## 5 Spanner

<img src="http://nickolashu.github.io/img/spanner.png" alt="spanner" style="zoom:40%;" />

MVCC: no locking consistent read, with snapshot. (with no write after read casual consistency problem)

read transcation(Tr) can only read transcations before Tr;

Write transcation(Tw) will not overwrite value, but create a new value with timestamp.

MySQL local timestamp, Spanner global unique timestamp generate.

<img src="http://nickolashu.github.io/img/spanner-transcation-commit.png" alt="spanner-transcation-commit" style="zoom:40%;" />



<img src="http://nickolashu.github.io/img/spanner-true-time.png" style="zoom:40%;" />

## Reference

[Cache Consistency: Frangipani](https://pdos.csail.mit.edu/6.824/notes/l-frangipani.txt)
