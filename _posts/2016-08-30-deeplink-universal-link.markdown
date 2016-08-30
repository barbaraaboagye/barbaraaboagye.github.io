---
layout:     post
title:      "user growth using deeplink."
subtitle:   ""
date:       2016-08-30 19:46:00
author:     "Nickolas"
header-img: "img/dl.jpg"
---

活跃人数是衡量app一项关键指标, dau, mau, 有了流量才能给业务发展提供养分和空间. app的流量一方面来自自身的留存, 一方面来自外部的供给, 而deeplink是外部引流的重要手段.

# 1 什么是deeplink
当有人分享一个商品给你, 发送一个链接到你的手机上, 你点击链接后直接跳转到app内对应的页面. 当你从浏览器中使用百度搜索, 点击一个搜索结果, 直接跳转到你的app的指定页面. 这些都是deeplink的使用场景, deeplink是从拦截外部请求到app内并定位到具体页面的技术.

# 2 为什么做deeplink
上面提到了流量的重要性, 而deeplink能够将外部流量引导到app内, 并提供连贯的浏览体验, 实在是引流的大杀器. 除此之外, deeplink还做高效的流量分发. 通过外部url的投放和内部拦截, 可以针对不同场景直接分发流量到具体页面, 将用户直接引导到各个垂直频道. ios的appsearch和消息都在努力做, 培养用户, 虽然目前使用量不大, 但的确是比入口堆叠更有效的流量分发方式.

# 3 如何做deeplink
deeplink分两种, 一种是用户已经安装了你的应用, 直接做链接拦截和跳转. 另一种是用户没有安装, 需要下载安装, 安装完成后再跳转到具体页面(reserved deeplink). 这两种都有不同的方案和对应的难点.

已经安装app的应用, ios8以下可以使用URLSchema做url拦截. Android上[applink](https://developer.android.com/training/app-links/index.html)可以解决跳转的问题. 而iOS9以上提供的universal link方案把体验做到了极致, 无需弹窗提示, 直接拦截跳转. 下面重点看看universal link如何做.

## 实现universal link
建议先看下[官方文档](https://developer.apple.com/library/ios/documentation/General/Conceptual/AppSearch/UniversalLinks.html)

apple为了体验和安全, universal link流程略复杂. 如官方文档提到, 需要在要拦截的http链接根路径下(或者.well-known下)提供一个apple-app-site-association文件, 文件描述了对域名的拦截规则, 格式可以参照google的https://google.com/apple-app-site-association. 这里有几点需要*特别*注意!

* aasa文件是精确到域名的, 也就是说www.google.com和map.google.com是两个aasa文件.
* aasa不能有302, response header必须是200.
* 必须是https.

接着需要在app的com.apple.developer.associated-domains文件中添加拦截的域名, 例如.

```
applinks:map.google.com
```

最后在UIApplicationDelegate中实现 _application:continueUserActivity:restorationHandler:_ 方法, 完成跳转并且 _return YES_ .参照[API文档](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIApplicationDelegate_Protocol/index.html#//apple_ref/occ/intfm/UIApplicationDelegate/application:continueUserActivity:restorationHandler:)

## 调试universal link
如果你照着上面做universal link就生效了, 那么恭喜你, 可以略过这段.

universal link的缺点是----链路太长, 不好调试. 笔者本着业界良心, 分享下之前解决universal link不生效的小技巧.

* 1 使用真机调试. 官方说模拟器可以调试, 可有时事实却不是这样, 保险起见使用真机调试.
* 2 使用抓包软件分析app安装. 每次删除重装app, 分析请求. app会在安装阶段请求aasa文件做校验. 因此如果没有发请求, 就是app entitlement写的有问题, 否则就是aasa文件问题.
* 3 调试aasa请求. 注意几个点, 路径, 域名, https, 200. 官方说的mime-type和content-type, 亲测不需要. 还有一小技巧, 可以把请求代理到本地服务直接测试, 不需要发布到线上.
* 4 使用iMessage测试. 用iMessage发送url做测试最靠谱, 其他app由于有可能定制了Safari, 会有各种诡异问题. 长按url, 看看是否有使用xxx打开选项, 因为有可能误关了universal link.

# 总结
相信完成了applink, 能够大幅提升app流量和产品体验. 下回看看如何用户没下载app时如何在安装完再跳转到具体页面.(reserved deeplink). 以及如何把这些技术串起来, 做到完美的体验.
