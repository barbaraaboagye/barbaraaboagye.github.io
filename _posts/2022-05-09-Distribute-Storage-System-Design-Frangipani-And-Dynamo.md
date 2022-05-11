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

Frangipani是一个分布式的文件系统(1997), 为了实现多人在线编辑文件的场景. 在简化使用场景下, Frangipani不支持多人同时在线编辑同一个文件, 即独占式的写入. 

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



总结: 

Frangipani是一个典型的CP系统, 牺牲可用性, 达到系统的强一致性和缓存的分区. 通过一个Lock Server节点, 实现各个节点对资源的顺序读写, 达到整体的强一致性.

## 2 Dynamo 去中心化高可用的KV存储
Dynamo是Amazon设计的去中心化的高可用的Key-Value存储. 最初被用来构建购物车应用, 后面延续到其他的应用场景, 在Amazon的业务场景下, 一个存储系统需要:

1. 大规模, 可靠性. 支持Tens of thousands of servers and network components, 在这个量级下设备和环境的问题被放大, 作为基础设施需要兼容异常, 简化上层业务.
2. 扩展性,  highly scalable to support continuous growth. 易于扩展.
3. 主键存储. 简化的数据模型, 只支持主key模式的数据存储. 降低RDB带来的额外开销. 
4. 高性能, 低延迟. 满足购物车查询和修改的延迟要求.

在扩展性上, 使用一致性Hash, 
在高可用(Available)上, 数据

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

总结  
根据Werner的描述, Dynamo在Amazon内部没有推广使用. 尽管Dynamo的设计很精妙, 但操作运维的复杂度(Operational Complexity)和系统规模带来的理解成本, 让大多数技术人员望而却步. "Developers strongly preferred simplicity to fine-grained control, they voted with their feet and adopted cloud-based AWS solutions, like Amazon S3 and Amazon SimpleDB, over Dynamo".
后续在数据结构和Cloud Base方向演进, 变成现在的DynamoDB.



## 3 Reference

[Cache Consistency: Frangipani](https://pdos.csail.mit.edu/6.824/notes/l-frangipani.txt)
[Amazon DynamoDB](https://www.allthingsdistributed.com/2012/01/amazon-dynamodb.html)D
[DynamoDB Develop Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
