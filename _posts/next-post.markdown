---
layout:     post
title:      Think in technology part 2.(Draft)
subtitle:   “Do one thing every day that scares you.” -- Eleanor Roosevelt
date:       2022-02-08
author:     "Nickolas"
header-img: "img/black-and-white-man-model-25759.jpg"
---
[前一篇](https://nickolashu.github.io/2022/02/06/think-in-technology/)提到的技术人的职责, 可以总结为以下3点:
1. 技术视野和判断力(Technology Visionary)
2. 技术设计 构建 执行的**实践**能力和经验(Practical ability and experience to design, build, and run the technology)
3. 组织设计和管理(Build and manage a organisation)

从一线技术人员到CTO, 技术工作都是这几种职责的组合, 只不过时间分配占比不同. 不论是企业招聘还是个人职业发展, 都可以参照这个模型. 前一篇主要说的是技术视野和判断力. 这篇聊聊技术执行部分.

# 一 技术执行(technical execution/operation)的维度
技术执行是一种实践能力. 
完整的思考过程和维度: 
1. why价值分析 为谁而做 解决什么问题(business value/issues) ROI如何
2. what问题定义 做的什么事情(在全局/上一个层次中的位置怎样, 是一个点的问题还是方向的问题, 长期的technical strategy是什么) 目标/产出是什么 
3. how挑战分析(如何评估做的好坏 需要做到什么程度) 如何做 架构设计
# 二 架构实践分析
成功的架构经验
* 淘宝交易系统 (分布式数据库+可靠消息)
https://developer.aliyun.com/article/757221
* twitter 
https://blog.twitter.com/engineering/en_us/topics/infrastructure/2017/the-infrastructure-behind-twitter-scale
* Google Search Engine 
https://en.wikipedia.org/wiki/Google_Search 
http://sifaka.cs.uiuc.edu/~wang296/Course/IR_Fall/docs/PDFs/Search%20Engine%20Architecture.pdf
* AWS 
https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html
* FaceBook 
https://medium.com/swlh/an-introduction-to-facebooks-system-architecture-47cfcf597101
* 阿里OceanBase分布式存储架构 
https://book.douban.com/subject/25723658/

好的技术人员: [Martin Fowler](https://martinfowler.com/)

失败的架构经验 
https://www.alexhudson.com/2017/10/14/software-architecture-failing/
阿里中台架构和内部创新. 
中台架构分析.
内部创新没有能做下去的原因. 方向正确, 对核心业务的价值不明确, 缺乏技术产品商业化能力. 对大公司而言, 内部创新的好处, 机会成本, 人才培养.

1 对大公司架构经验保持警惕. 小公司不能照搬大公司的经验, 还是要看清背后的取舍和原因, 技术复杂度和业务规模需要匹配.
[Alex Hudson](https://www.alexhudson.com/about)是一个中小企业的架构顾问, 有很多中小企业失败案例的经验.  

2 对新技术保持警惕. 不要将核心业务架构不确定性高的创新技术之上. [前一篇](https://nickolashu.github.io/2022/02/06/think-in-technology/)有提到Gartner成熟度曲线, 通常新技术需要一个发展周期. 
# 三 架构经验&架构理论
1. 架构原则: (From 阿里王晶昱)
	* 服务化架构
	* 去中心化，线性扩展: HSF(Dubbo) + DRDS
		* 去中心化: 系统无单点, 系统中所有角色可单独扩缩
		* 服务能力，随着资源加入，微服务级线性的性能和容量扩展
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
2. DDD
# 四 延伸阅读
1. [大规模分布式存储系统 ](https://book.douban.com/subject/25723658/)
2. [阿里架构发展2015QCon分享](https://docs.huihoo.com/infoq/qconshanghai/2015/%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1/QCon%E4%B8%8A%E6%B5%B72015-%E6%B7%98%E5%AE%9D%E6%8A%80%E6%9C%AF%E5%8F%91%E5%B1%95%E5%8E%86%E7%A8%8B%E5%92%8C%E6%9E%B6%E6%9E%84%E7%BB%8F%E9%AA%8C%E5%88%86%E4%BA%AB-%E7%8E%8B%E6%99%B6%E6%98%B1%EF%BC%88%E6%B2%88%E8%AF%A2%EF%BC%89.pdf)





