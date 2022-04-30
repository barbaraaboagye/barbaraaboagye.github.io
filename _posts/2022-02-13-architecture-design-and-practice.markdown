---
layout:     post
title:      Architecture design and practice analysis. 
subtitle:   “Form ever follows function.” -- Louis H. Sullivan 1896
date:       2022-02-13
author:     "Nickolas"
header-img: "img/essense.jpg"
---

本文讨论如何构建技术执行能力. 不论是企业招聘还是个人职业发展, 都可以参照这个能力模型.

先是回顾技术人员的3个能力象限. 再从技术视角如何思考和判断问题. 到架构设计的原则. 并通过交易和营销的成功实践分析, 理解什么是好的架构.



[前一篇](https://nickolashu.github.io/2022/02/06/think-in-technology/)提到的技术人的职责, 可以总结为以下3点:

1. 技术视野和判断力(Technology Visionary)

2. 技术设计执行的**实践**能力和经验(Practical ability and experience to design, build, and run the technology)

3. 组织设计和管理(Build and manage a organisation)

   

从一线技术人员到CTO, 技术工作都是这几种职责的组合, 只不过时间分配占比不同. 不论是企业招聘还是个人职业发展, 都可以参照这个模型. 前面主要说的是技术视野和判断力. 这篇聊聊技术执行部分.



## 一 技术视角问题思考
技术的背景的人, 容易陷入如何解决问题的思维定式. 一个自顶向下的完整技术思考维度应该包括:

1. 价值分析(Why). 为谁而做, 解决什么问题(business value/issues), ROI如何.  价值分析决定了要不要做, 什么时间做, 投入多少资源.

2. 问题定义(What). 事情在全局/上一个层次中的位置怎样, 是一个点的问题还是方向的问题, 长期的technical strategy是什么, 目标/产出是什么. 这部分还是在一个抽象层次上帮助理解问题, 确定目标.

3. 挑战分析&架构设计(How). 针对业务的目前, 发展诉求和约束, 做可扩展的架构设计. 同时考虑需要做到什么程度, 如何评估做的好坏.



## 二 架构理论

[前一篇](https://nickolashu.github.io/2022/02/06/think-in-technology/)有讲到, 架构指建筑/软件设计和构建的艺术或实践(the art or practice of designing and constructing buildings), 是成功和失败经验的总结.

1. ### 架构原则

   阿里架构师王晶昱总结的架构原则. 使用架构原则指导技术选型和架构的完备设计. 

   * 服务化架构
   * 去中心化，线性扩展: HSF(Dubbo) + DRDS
     * 去中心化: 系统无单点, 系统中所有角色可单独扩缩
     * 服务能力，随着资源加入，微服务以及线性的性能和容量扩展
   * 异步化，最终一致
     * 流程异步化, 去锁, 并行
     * 系统应用尽量无状态化
     * 确保系统最终一致
   * 使用成熟组件: **越下层的系统，越需要稳定**
     * 长期实际生产环境中证明过的可靠成熟组件
     * 用户量翻倍，系统构建难度也会翻倍
   * 自动化, 高可靠
     * 数据化监控运维. 系统运行状态, 异常可观测.
     * 任何节点和链路故障情况，能够自动检测, 高效处理
     
       

2. ### DDD 领域驱动设计

   Quote from [Martin Fowler](https://martinfowler.com/bliki/DomainDrivenDesign.html):
   > Domain-Driven Design is an approach to software development that centers the development on programming a domain model that has a rich understanding of the processes and rules of a domain.  The approach is particularly suited to complex domains, where a lot of often-messy logic needs to be organized.

   * 围绕Domain Model理解和设计系统. 有助于控制复杂系统的大型的复杂度, 同时也会增加新人理解门槛.
   * DDD是对复杂现实业务的领域抽象, 像导购的领域, 大数据或其他技术系统不适合使用.
   * 在Domain基础, Java应用可以使用[Presentation Domain Data 分层设计](https://martinfowler.com/bliki/PresentationDomainDataLayering.html) 做为应用分层设计.

## 三 交易营销架构实践分析

通过分析成功的经典的架构经验, 来理解如何做业务分析和架构设计.



### 1 交易系统 

#### 1.1 [微信群红包系统](https://www.infoq.cn/article/2017hongbao-weixin/?pid=2900) 

针对微信红包业务特点, 做需求分析:

* 不能接受超卖问题. 资金要求强一致性.  

* 和秒杀商品类似, 访问具有集中性, 不存在跨红包访问. 并且单红包读写规模有限, 群红包受群大小限制, 一般单红包读写量几百以内. 红包总量和总访问qps高, 2017年除夕红包读写qps76万，收发微信红包 142 亿个. 

* 拆红包且具有时效性.  (24h未领取退回).



针对业务特点. 对应的设计: 
* 红包数据直接在DB管理, 保障资金强一致性. 

* 由于请求针对红包具有集中性, 用红包id做分库, 提升整体负载能力. 由于单红包访问规模有限, 接入层按照红包id的hash分发请求, 使同一红包的请求落在同一机器上, 保障数据库连接资源可以被服用, 同时方便水平扩展. 为了降低数据库事务压力, 对同一红包写请求做FIFO本地的排队, 将并发问题转换为串行问题.

* 冷热数据访问量差距明显, 可以做冷热分离. 用日期值做分表. 

  > 分库表规则像 db_xx.t_y_dd 设计，其中，xx/y 是红包 ID 的 hash 值后三位，dd 的取值范围在 01~31，代表一个月天数最多 31 天



#### 1.2 [电商交易系统](https://developer.aliyun.com/article/757221) 

电商交易业务特点:
* 一个交易订单, 由买家 商家 商品构成. 一笔消费交易由买家发起, 对不同商家订单拆分为子订单. 订单查询分为买家查询和商家查询. 订单读写具有天然隔离, 不会跨用户或跨商家.
* 淘宝的交易叫做[信保交易](https://activity.alibaba.com/page/tradeassurance.html), 交易流程从下单开始, 到买家确认收货, 给商家打款结束.
* 单日交易量大, 交易峰值大. 2021 天猫双十一交易量5403亿元, 按笔单价100元估算有54亿笔订单. [2020年双十一订单创建峰值58.3万/秒](https://www.thepaper.cn/newsDetail_forward_9932469).
* 交易查询有时效性近期订单查询量远大于历史(90天)订单. 
* 交易对库存有一致性依赖, 少卖或超卖对商家都会有损失. 商品交易存在明显马太效应, 存在明显热点商品. 



对应的一些设计:
* 高可用. 一方面垂直扩展, 库存库, 订单库分为买家库和商家库. 一方面做水平扩展,  订单库分别用买家id和商家id做分库分表拆分, 库存库用商品id拆分. 另一方面, 主备, 去单点写(双写, 多写和failover).

* 领域设计. 交易系统需要对订单流转状态做管理. 订单流程结合交易营销玩法, 涉及到营销价格计算, 库存减扣, 履约, 资金支付等领域. 适合使用DDD对各域拆分设计, 保证业务持续演进过程中, 整体的复杂度上有更好的局部性.

* 冷热分离. 在线和历史库存对事务以及数据量要求不同, 使用不同的存储方案, [在线MySQL, 历史HBase](https://developer.aliyun.com/article/757221).

* 库存高并发. 类似于秒杀的热点商品。
1) 读缓存. 本地缓存+集中式缓存降低高并发读, 数据库事务保障数据一致性. 
2) 热点商品识别和优化.  
3) 使用队列对商品做顺序写. 有几种选择, 单机队列(类似于微信红包的设计), 分布式队列, MySQL队列. 
4) 库拆分, 讲热点库存拆分成多个桶，多个桶加和总库存, 用户流量分散到多个桶库存中, 降低热点库压力.
5) MySQL事务优化. 
6) 离线任务对账. 
具体参照[君山的文章](https://www.toutiao.com/i6260281405876470273/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io&wid=1644746473876).

  ![img](https://p6.toutiaoimg.com/origin/33f0001674c419fdbe4?from=pc)

总结下交易系统主要设计. 数据库垂直和水平拆分, 库表拆分维度和值设计, 避免跨表访问; 利用多级缓存, 降低库读压力; 通过队列方式, 将随机请求和锁变为顺序请求和并发请求, 降低数据库写入效率.



### 2 营销系统

#### 抢红包活动, [微信摇一摇红包](https://www.infoq.cn/article/1-billion-bonus-from-the-clouds) ,双十一红包雨

业务特点:
* 红包场次不多, 资金池规模大.
* 红包预算不能超发.
* 瞬时访问量大, 比如春晚微信摇一摇红包. 天猫双十一红包雨.

设计:
* 大的资金池可以拆分成多个小资金池, 对用户访问做打散, 转化为前面热点商品库存问题.
* 通过限流方式, 上游保护下游, 各级对自身做限流保护(接入层, 应用, 数据库). 限流越靠前, 整体系统资源占用越少.


## 四 总结&延伸阅读

本文尝试分析和总结技术执行的思考维度, 架构设计的原则和交易营销的经典架构. 由于时间关系, 营销招选投架构, 搜索, SNS的经典架构留到下一篇分析. 



延伸阅读:

1. [大规模分布式存储系统 ](https://book.douban.com/subject/25723658/)
2. [阿里架构发展2015QCon分享](https://docs.huihoo.com/infoq/qconshanghai/2015/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1/QCon%E4%B8%8A%E6%B5%B72015-%E6%B7%98%E5%AE%9D%E6%8A%80%E6%9C%AF%E5%8F%91%E5%B1%95%E5%8E%86%E7%A8%8B%E5%92%8C%E6%9E%B6%E6%9E%84%E7%BB%8F%E9%AA%8C%E5%88%86%E4%BA%AB-%E7%8E%8B%E6%99%B6%E6%98%B1%EF%BC%88%E6%B2%88%E8%AF%A2%EF%BC%89.pdf)
2.  [Martin Fowler](https://martinfowler.com/)
2.  [Revolutionizing Money Movements at Scale with Strong Data Consistency](https://eng.uber.com/money-scale-strong-data/) (TODO)
