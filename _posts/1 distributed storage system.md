1 Frangipani. serialization 一致性的 FileSystem

[Cache Consistency: Frangipani](https://pdos.csail.mit.edu/6.824/notes/l-frangipani.txt)

"cache coherence" solves the "read sees write" problem
  the goal is linearizability AND caching
  many systems use "cache coherence protocols"
    multi-core, file servers, distributed shared memory

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
​    acquire lock, then read from Petal
​    write to Petal, then release lock
  coherence protocol messages:
​    request  (WS -> LS)
​    grant (LS -> WS)
​    revoke (LS -> WS)
​    release (WS -> LS)
```



2 Redis

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

3 Dynamo
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






https://www.educative.io/courses/grokking-adv-system-design-intvw/xoEXr9614RB
