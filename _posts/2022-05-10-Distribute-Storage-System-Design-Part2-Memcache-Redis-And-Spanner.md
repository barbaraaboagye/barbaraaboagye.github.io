---
layout:     post
title:      Distribute Storage System Design Part2 -- Memcache Redis And Spanner
subtitle:   
date:       2022-05-10
author:     "Nickolas"
header-img: 
---

## 1 Memcache 分布式HashTable (DHT)

Design needs: 1) ner real-time communication. 2) aggregate content on-the-fly from multi sources 3) able to access and update popular shared content 4) scale to millions qps. 

Read-heavy workload and wide fan-out.

Design principle: 

1. demand-filled lool-aside cache. Choose to delete cache data instead of update, cause of idempotent.

2. Memecache server do not communicate with each other. Try to embed the complexity of the system into a stateless client rather than in the memcache server.

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

## 2 Redis

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



## 3 Spanner

<img src="http://nickolashu.github.io/img/spanner.png" alt="spanner" style="zoom:40%;" />

MVCC: no locking consistent read, with snapshot. (with no write after read casual consistency problem)

read transcation(Tr) can only read transcations before Tr;

Write transcation(Tw) will not overwrite value, but create a new value with timestamp.

MySQL local timestamp, Spanner global unique timestamp generate.

<img src="http://nickolashu.github.io/img/spanner-transcation-commit.png" alt="spanner-transcation-commit" style="zoom:40%;" />



<img src="http://nickolashu.github.io/img/spanner-true-time.png" style="zoom:40%;" />