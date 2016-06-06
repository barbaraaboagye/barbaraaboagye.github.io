---
layout:     post
title:      "React Native Changed the World? or Nothing."
subtitle:   “We convince ourselves, and that allows us to convince others.”
date:       2016-05-30 10:08:00
author:     "Nickolas"
header-img: "img/RN-or-not.jpg"
---

RN是一个awesome的技术, facebook很有想法的团队创造出一项新的技术改变了native开发界. 但是RN本身又疑点重重, RN是为了解决什么问题而存在的? 在诞生了一年后, RN又解决了什么问题? 本文通过分析RN的思想, 试图透过技术, 理解动态方案.

RN(React Native)是Facebook推出的mobile开发框架, 使用javascript作为开发的主要语言, 逻辑和样式的处理由javascript完成, 渲染则使用native渲染, 支持Android和iOS两个平台. 对于native开发者, RN通过配置下发能够做到即时生效的上线. 对于web开发者, 能够使用自己熟悉的技术体系做native开发.

一年前, RN推出的时候, 惊艳移动开发业界, 大家都惊呼原来native开发还能这样做 -- 跨平台, 动态发布, 简洁的JSX语法, 各种开发工具和调试工具. 我当时也是RN的狂热推崇者, 里面很多思路在当时看来的确是引领了一代创新. 而现在, 在做了一年多native动态性的工作之后, 发现RN的背后问题重重.

# 问题1 RN是为了解决什么问题而存在的?

## 观点1 解决跨平台问题

> "React Native enables you to build world-class application experiences on native platforms using a consistent developer experience based on JavaScript and React. The focus of React Native is on developer efficiency across all the platforms you care about — learn once, write anywhere. Facebook uses React Native in multiple production apps and will continue investing in React Native."

摘自[RN官网](https://facebook.github.io/react-native/), 从这里我们可以看出, 官方是把RN作为一套跨平台的技术方案的, *learn once, write anywhere*, 也就是说RN解决了Android, iOS跨平台开发的问题. 但是别忘了, web本身就是跨平台的, HTML, CSS, javascript这些都是[universal W3C standards](https://www.w3.org/standards/).

## 观点2 解决web体验问题

有人会说, RN是为了解决web开发者在mobile上的性能体验问题. 的确, RN通过使用大量native组件做渲染, 优化了WebKit DOM渲染的性能, 通过batch update和专用线程的设计, 降低javascript和native通信带来的性能开销, 保证主线程的流畅执行. RN的组件loadtime的确比web的好, 因此体验上面, 页面加载更快, 滑动时候局部留白的时间更短, 对用户而言体验效果更好. 但也带来了很多问题.

通常web的开发是建立在W3C完备标准之上, 无论是浏览器中, 还是app中的webview中, 都遵循W3C的一套标准实现. 也就是说web开发不需要关心具体的浏览器是怎样实现的, 只要按照规范开发, 就能跑在各种各样的容器上面, 也是在此基础上, 各种各样的javascript技术蓬勃发展. 反观RN, 官方提供了一套javascript API, 但是和W3C相比, 还不成熟完备. 对于web开发很难说把native这层看为透明的. 或者说, 不改变native完全使用javascript的开发局限性很大. 很可能某个native组件的性能或者功能不满足现有业务需求, 这时就很尴尬, 需要我们深入native的组件. 一旦涉及native组件, 伴随而来的还有版本和兼容等问题. 作为开发而言, 需要熟悉web native和RN框架, 这带来的是开发效率的问题. 但随着官方不断吸纳标准的组件, 这个问题会逐渐变小, 但站在现在, 这个问题的确存在.

站在这个角度上, 如果web开发者也掌握了native开发的技能或者有app开发的支持, 那为什么不直接做成native app? RN的性能和体验和native为主的app还是有差距的. 这里总结下来, RN会提升web体验, 但有一定局限性, 可能成为后面业务发展的瓶颈.

## 观点3 解决native开发和发布效率问题

有人又会说, native app不能做动态发布, 开发效率低, 而RN可以. 在发布问题上面, [AppStore是严禁热部署动态code的](https://developer.apple.com/app-store/review/guidelines/), 因此每次发布都需要通过AppStore审核, 而近期[AppStore的审核时间已经缩短到24h以内](http://www.feng.com/iPhone/news/2016-05-11/App-Store-application-review-time-is-less-than-24-hours_646453.shtml). Android平台上面, 需要解决的是更新率的问题, 目前业界一些插件化动态发布的方案可以完成动态更新, 如手淘的Atlas, 点评的[DynamicAPK](https://github.com/CtripMobile/DynamicAPK).

开发效率上面如果Android和iOS能够用同一套javascript实现, 在熟悉了RN环境后, 的确可以大幅提升开发效率. 但对于native开发而言, 还是有一定起步成本, 而且还是前面提到的瓶颈问题, 依赖现有native组件的完备程度.

# 问题2 RN现在解决了什么问题?

按常理讲, 推出技术方案大多基于自身遇到的问题. 但[RN并没有在Facebook的主流产品中使用](https://facebook.github.io/react-native/showcase.html), 而且使用RN的App也大多是小众的App.

相比之下, Facebook的[Instant Article](https://instantarticles.fb.com/)更为抢眼, native端将pageLoad, 交互体验作为最终目标, 提高内容体验. 同时解决内容发布的几个问题, 降低发布门槛, 增加广告收益, 提供数据分析.

RN技术本身并没有什么问题, 问题是各方面都比较好的方案, 在真实业务场景下反而更难用好.

# 问题3 RN的尴尬源自哪里?

最开始看到javascript+native这种技术, 想到的是端游开发界. 很多端游开发也是使用lua+游戏引擎的技术, lua写逻辑更快, 引擎性能更好. 不同的游戏用多套脚本, 引擎可以持续复用. 这个很像RN解决问题的思路, 但为什么在RN上行不大通呢?

反复的思考后, 觉得根源是在mobile这里. mobile有什么特点? 用户时间碎片化, 单任务. 碎片化怎么理解, 根据[友盟+的数据分析报告](http://www.umeng.com/reports.html?spm=0.0.0.0.5Xi9gl&from=hp), 除了主流的资讯, SNS, 游戏类以外, 一个人使用app次数是分钟级别的, 而使用次数却会有10+次. 单任务怎么理解, 用户在使用手机的时候更难做app切换, 不会像pc中同时开启多个任务去做. 这两个核心的特点决定了, 用户较难以等待, 更容易流失. 所以pageLoad time是很重要的一个指标, 尤其是内容型的页面. 而RN同native相比在pageLoad time上是有很大差距的.

在另一边, 相比web, RN性能虽然比web好, 但灵活程度, 成熟程度远远不如web技术. 而是有web技术开发的业务一般来说对体验的要求有一定容忍. 同时还有Google等在做的专注mobile web page性能的[Accelerate Web Page](https://www.ampproject.org/)这类项目, 这样RN和web的性能很难拉开大的差距.

这样RN处在了一个不上不下的尴尬位置.

# 看清问题, 找到解法.

> "再 NB 的想法和假设，真正到了实现层面，也必须要落实到具体的用户、需求和场景这三要素上面."

前面一直在说问题, 不是说不认可RN的技术, 只是不想让大家跟着热潮去入坑, 而关键是要把相关的技术放在一起比较, 看哪个方案适合自己的业务场景.

举个例子, 在有强运营需求的基础功能上, 非常适合用RN来做运营模块. 因为运营的特性是时效性, 到达率, 而基础功能又需要保证稳定的体验. 过去的一年我的团队都在使用native为主+动态模块这种技术, native上提前做好动态模块的支持, 当营销活动点到来时, 可以用RN这种动态技术, 快速响应, 动态上线.

再举个例子, 像做Instant Article这种内容页面, 会有大量的内容, 以几种排版布局展示. 这类的业务的核心是对内容的承载能力, 体验. 像这类的业务应该针对业务的现状和近期发展, 确定适合业务的方案, 并保持对未来的可扩展性. 如果没有运营类的需求场景, pure native的实现可以完全满足需求, 设计后台接口时考虑到布局和组件的扩展性, native上能保证无痛的后续新增和修改布局, 就可以了. 简单的架构也方便后面针对性能瓶颈做持续优化.

总之, 对新技术要保持敏感, 在实际业务的技术选型上还是要保持警惕, 慎重权衡.
