---
layout:     post
title:      Reading List of Distributed System
subtitle:   
date:       2022-03-27
author:     "Nickolas"
header-img: 
---

分布式问题在不同的性能(Avaliable), 一致性时间约束(Consistency)下, 有不同的理论和实践方案, 是在CAP中设计取舍, 本身有足够复杂又有趣问题. 分布式理论在1970-2000年有非常快速的发展. 比如CAP, 2PC, Paxos.



当下的互联网是构建在精妙设计的分布式系统之上的, 比如GFS, MapReduce, Spanner, Zookeeper, Cassandra, BigTable, Dynamo, Kafka. 这些系统在2000-2013年诞生. 当下这些分布式系统技术已经相对成熟, 虽然现在已经很少有机会去设计这些系统, 但熟悉其原理和取舍, 对于商业系统架构设计非常重要. 



所以梳理了领域中的经典理论和系统, 以及近年对理论的新的解读. 整理出来, 抽时间慢慢阅读消化, 希望对大家有帮助.



## Consensus Theory

2PC. Gray described 2PC in ["Notes on Database Operating Systems" (1979)](http://research.microsoft.com/~Gray/papers/DBOS.pdf). 在[Consensus Protocols: Two-Phase Commit](https://www.the-paper-trail.org/post/2008-11-27-consensus-protocols-two-phase-commit/)(2008)这篇文章中有更straight forward的介绍. Thought works这篇文章https://martinfowler.com/articles/patterns-of-distributed-systems/two-phase-commit.html(2022)有更新的理解和详细的解释.

3PC. [NonBlocking Commit Protocols" (1981)](http://www.cs.cornell.edu/courses/cs614/2004sp/papers/Ske81.pdf)

[Paxos Made Simple](http://research.microsoft.com/en-us/um/people/lamport/pubs/paxos-simple.pdf), a more terse readable Paxos paper by Lamport himself. Shorter and more easier compared to the original.

[Raft Consensus Algorithm](https://raftconsensus.github.io/) An alternative to Paxos for distributed consensus, that is much simpler to understand. Do checkout an [interesting visualization of raft](http://thesecretlivesofdata.com/raft/)

Replicated Log



## Concurrency Theory

[The C10K problem](http://www.kegel.com/c10k.html)

[A High-Speed Load-Balancer Design with Guaranteed Per-Connection-Consistency](https://www.usenix.org/system/files/nsdi20-paper-barbette.pdf) - Barbette et. al., NSDI 2020



## Distributed System

[Kafka: a Distributed Messaging System for Log Processing](http://notes.stephenholiday.com/Kafka.pdf)

[Bigtable: A Distributed Storage System for Structured Data](http://static.googleusercontent.com/media/research.google.com/en//archive/bigtable-osdi06.pdf)

[Dynamo: Amazon's Highly Available Key Value Store](http://bnrg.eecs.berkeley.edu/~randy/Courses/CS294.F07/Dynamo.pdf) Paraphrasing @fogus from their [blog](http://blog.fogus.me/2011/09/08/10-technical-papers-every-programmer-should-read-at-least-twice/), it is very rare for a paper describing an active production system to influence the state of active research in any industry; this is one of those seminal distributed systems paper that solves the problem of a highly available and fault tolerant database in an elegant way, later paving the way for systems like Cassandra, and many other AP systems using a consistent hashing.

[The Google File System](http://static.googleusercontent.com/external_content/untrusted_dlcp/research.google.com/en/us/archive/gfs-sosp2003.pdf)

Spark

Mesos



## Other List & Resource

[Awesone Distributed Systems](https://github.com/theanalyst/awesome-distributed-systems)

[Readings in distributed systems](http://christophermeiklejohn.com/distributed/systems/2013/07/12/readings-in-distributed-systems.html)

[MIT 6.824 Distributed Systems](https://pdos.csail.mit.edu/6.824/)

[Google Research](https://research.google/pubs/?area=distributed-systems-and-parallel-computing)

[Read List From Paper Trail](https://www.the-paper-trail.org/page/reading-list/)
