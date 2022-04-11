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

Redis internals https://redis.io/docs/reference/internals/

https://medium.com/opstree-technology/redis-cluster-architecture-replication-sharding-and-failover-86871e783ac0#:~:text=Redis%20Cluster%20is%20an%20active,a%20subset%20of%20those%20slots.

https://aws.amazon.com/redis/