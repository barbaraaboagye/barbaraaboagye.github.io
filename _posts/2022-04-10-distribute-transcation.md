---
layout:     post
title:      Distribute Transcation -- 2PC & Saga Pattern
subtitle:   
date:       2022-04-10
author:     "Nickolas"
header-img: 
---

## Introduction

大家都知道, 事务通过四个特性ACID, 对逻辑的正确性提供保障. 在单机事务中, 很容易通过锁, 数据库事务等方式, 实现不同隔离级别的事务性保障. 而在分布式环境中, 目前还没有方案同时完美的做到事务性和可用性, 各个算法和Product实现,  都是在做Trade off, 这也是分布式系统理论和实践的复杂性和魅力所在. 

## Problem Description

在一个微服务系统中, 一个API会依赖下游多个服务, 每个服务有自己数据存储和处理逻辑, 整体对外需要提供一致性事务. 想象一个订单提交服务, 需要扣减库存, 扣款, 提交物流履约信息. 所有的服务, 要么全部成功, 要么全部失败.

![img](https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/e-commerce-architecture-1024x693.png?resize=573%2C388)

## 2 Phase Commit

 2PC算法最早是Gray在"Notes on Database Operating Systems" (1979)中提出来的. 每个独立的Service是一个参与者A, B, 协调者TC负责协调事务的状态. 算法的过程如下: (不考虑System failure)

> TC sends PREPARE messages to A and B.
>     A and B lock records.
>     Modifications are tentative, on a copy, only installed if commit.
>   If A is able to commit,
>   	A responds YES.
>   	then A is in "prepared" state.
>   	otherwise, A responds NO.
>   Same for B.
>   If both A and B say YES, TC sends COMMIT messages to A and B.
>   If either A or B says NO, TC sends ABORT messages.
>   A/B commit if they get a COMMIT message from the TC.
>     I.e. they copy tentative records to the real DB.
>     And release the transaction's locks on their records.
>   A/B acknowledge COMMIT message. 
>   TC gets to the end of the transaction.

![img](https://miro.medium.com/max/640/0*FBQPiHKMRmrPE_mc.png)

从ACID分析2PC:

1. 原子性. 系统最终状态由TC决定, 要么全部提交，要么全部失败;
2. 一致性. 参与者在voting阶段，lock the record执行计算，根据执行结果vote。提交阶段, 需要无条件执行TC的指令, 执行完成后release the lock. 事务状态最终由TC决定, 各个参与者保持一致;
3. 隔离性. prepare阶段, 修改是临时的, 只有commit阶段才会提交到真实记录上, 在此之前并发的其他事务无法观察到其中间状态;
4. 持久性，每个参与先记录状态(Write Ahead Log)，再vote yes，保障就算其crash事务状态也会被持久化, 和整体的一致性.



2PC为分布式事务提供了非常强的一致性, 但牺牲了可用性. 参与者需要在第一个阶段加锁, 事务完成才能释放锁, 在第二个阶段解锁. 而参与者只有在第一个阶段有权利参与vote, vote之后必须等待TC的结果才可以释放锁. 因此所有参与者加锁的时长是最长执行时间加上网络时间.`time = MAX(t1, t2, ...)+ 2RT`, 使得系统性能不高. 



在等待期间, TC或者任一参与者failover, 参与者都必须无条件等待. 使得异常情况服务的可用性不可控. 3PC是对2PC阻塞情况的一种改进.



在2PC中, 节点提交之后就必须等待TC commit或者abort, 因为每个节点不知道其他节点的状态. 而3PC增加了一轮pre-commit环节, 使得节点可以在等待timeout时, 提前同步状态, 根据状态决定是否abort, 避免了对TC的单点依赖. 

![Screenshot 2022-04-10 at 8.15.13 PM](/Users/nickolashu/Desktop/Screenshot 2022-04-10 at 8.15.13 PM.png)



但3PC也不是完美的方案, 前面说过, 节点在timeout时, 相互同步状态, 根据同步结果决定提交还是abort. 而分布式状态同步本身会有脑裂等不一致问题, 比如Paxos解决的问题. 



## Saga Pattern

我们前面看到在实际应用中, 按照2PC协议, 复杂性和性能都难以达到要求. Saga Pattern, 通过最终一致性, 提升系统性能和可用性.



SAGA Pattern理论最早是Hector Garcia Molina & Keneth Salem在1987年提出来的. 最初是为了解决LLC(Long Lived Transaction)的问题, 将长事务, 拆分成多个小的事务, 串行执行. SAGA Pattern 可以和event driven很好的结合, 降低系统的复杂性. 



回到前面订单系统的例子, 每个依赖服务可以通过生产和消费消息, 变成一个串联的系统. 每个Service内部通过本地事务保障一致性.  

![img](https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.13.39-PM.png?resize=476%2C506)

当某个服务失败时, 需要将前面执行完成的服务回滚. Order Service最终将成功或失败的结果, 返回给Client.

![img](https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-09-at-6.36.17-PM-1024x702.png?resize=567%2C389)

1. 可以看到, 整个过程是一个异步的过程, 通过监听和生产消息, 完成状态流转, 最终系统达到一致性. 
2. 每个服务节点需要维护一个状态机. 不仅Order Sevice中维护着订单状态, 支付系统, 需要检讨库存成功和失败消息, 以便回滚.
3. 每个阶段需要有回滚能力, 采用乐观的机制, 默认提交, 异常回滚.
4. Saga Pattern解决的是business logic的一致性问题, system级别的一致性问题需要底层系统解决. 比如, 消息系统 at least once, 每个服务的replicate, 保证消息一定被消费, 结果一定产生新的消息. system级别依赖的是kafka rocketMQ等可靠消息系统.
5. 由于消息可能重复, Service需要支持幂等, 消息需要有唯一的txID.
6. 由于不存在Coordinator做状态同步, 不需要lock资源, 性能更好.

Saga Pattern可以有**Events/Choreography**和**Command/Orchestration**两种实现方式, 前者如前面介绍, 后者引入一个Orchestrator, 统一处理各个服务状态, 每个服务不需要维护独立的状态机.

![Command/Orchestration flow](https://i0.wp.com/blog.couchbase.com/wp-content/uploads/2018/01/Screen-Shot-2018-01-11-at-7.40.54-PM-1024x627.png?resize=712%2C436)

在Service数量不多的时候, Events/Choreography更合适, 避免引入新的依赖. 对于复杂的状态, 可以使增加Orchestrator做集中管理, 降低Service复杂性.

## Final Thoughs

* 分布式事务问题的根源, 在于数据和服务的分布式. 是微服务系统的Side Effect. 是Monolithic升级到Micro System必须面对的问题.
* 分布式事务问题, 从数据角度理解, 是对数据做垂直拆分之后的一致性问题. 相比Paxos Raft, 解决的是Replica水平拆分的问题. 前者的成熟系统有Spanner, 后者有Zookeeper.

## References

[MIT 6.824 2PC](https://pdos.csail.mit.edu/6.824/notes/l-2pc.txt)

[saga pattern implement business transactions](https://blog.couchbase.com/saga-pattern-implement-business-transactions-using-microservices-part/)

[limits_of_saga_pattern](https://www.ufried.com/blog/limits_of_saga_pattern/)

[columbia class Lec 17: Agreement in Distributed Systems: Three-phase Commit, Paxos](https://www.cs.columbia.edu/~du/ds/assets/lectures/lecture17.pdf)

[consensus protocols three phase commit](https://www.the-paper-trail.org/post/2008-11-29-consensus-protocols-three-phase-commit/)
