---
layout:     post
title:      "User Growth Using Deeplink Part 2 -- Reserved Deeplink"
subtitle:   “Progress lies not in enhancing what is, but in advancing toward what will be.”
date:       2016-09-28 21:01:16
author:     "Nickolas"
header-img: "img/dl.jpg"
---

接着上面一篇文章, [user growth using deeplink](http://nickolashu.github.io/2016/08/30/deeplink-universal-link/) .

Reserved Deeplink是拉新的重要手段, 让用户安装完客户端后恢复之前的上下文, 在上下文场景下能够很好提升体验和留存效果.      
不同于deeplink, 需要app已经安装在用户的手机上. 而reserved deeplink能够引导用户下载app并还原之前的上下文.  
故名思议, 将deeplink的时机延迟到app安装之后进行. 用一张图解释reserved deeplink(copyright branch.io).  

![reserved deeplink](img/deeplink.jpg)

# Android 动态打包技术

技术上面, android和ios使用不同的方案实现, 本质都是通过一种方式在新安装的app上面关联到之前的上下文. 在Android上面, 使用的是动态打包技术.
Android的开发同学知道, Android的安装程序使用的是APK(Android application package)文件格式. 大家可能不知道, 其实这个APK文件使用的是标准的ZIP文件, 参照[Wikipedia APK](https://en.wikipedia.org/wiki/Android_application_package).
而ZIP文件中有个段叫做File comment, 可以写入不定长的数据, 而不影响ZIP解压出的内容, 参照[Wikipedia ZIP file format](https://en.wikipedia.org/wiki/Zip_(file_format)). 这样可以将上下文信息动态写入comment字段. 具体写入方式可以参见[linghaolu的实现](https://github.com/linghaolu/apkcomment). 当用户请求带有deeplink的url时, 后台将context写入到apk包中的comment字段, 再将带有上下文的apk包让用户下载安装. 程序安装完成, 第一次启动时可以通过[getPackageCodePath](https://developer.android.com/reference/android/content/ContextWrapper.html#getPackageCodePath())方法获取apk地址, 解析apk的comment段, 并做页面跳转.
通过comment字段做上下文关联的技术同样适用于标记渠道包来源, 做数据分析统计.

# iOS shared cookie

iOS上面由于应用分发渠道等限制, 技术方案会稍复杂. ios9之后[SFSafariViewController](https://developer.apple.com/reference/safariservices/sfsafariviewcontroller)和Safari共享cookie, 可以通过cookie做上下文关联. 在用户点击deeplink url时, 后台把context写入cookie, 再redirect到AppStore.
在用户安装完成, 打开app后, native上打开一个用户不可见的SFSafraiViewController, 去请求后台. 由于share cookie机制, 后台可以在这次请求中拿到之前写入cookie的数据, 解析出来返回给app. App上再做相应跳转, 即完成了reserved deeplink. 这个流程见下图.

![reserved deeplink ios](img/reserved-deeplink.png)

iOS的这个解决方案有几个问题, 首先是需要iOS9及以上, 这样才能使用SFVC;
其次用户使用的浏览器需要是Safari, 这样才能利于shared cookie机制;
最后, 由于shared cookie的机制本身是由系统控制的, 实践中经常会获取不到cookie, 造成deeplink失败.
因此, 还有一种对shared cookie的补充方案. 即ip映射.

# iOS device fingerprint

站在后台的视角上, reserved deeplink要解决的问题本质是如何关联到用户浏览器环境和新安装的app. 如果我们能拿到设备的唯一性标记, 用设备的唯一性标记做key就可以关联浏览器和app端. iOS的设备标识IDFA, 是不是可以完美解决这个问题呢?
答案是不能够..., 因为Apple出于安全性考虑, 在web上无法获取到IDFA. 其实不止IDFA, 手机web上就没有能够唯一标记设备的标识. 但是否可以使用一种算法, 使用差异性的浏览器中的多个字段, 生成一个设备的唯一标识呢? 这中办法就叫做数字指纹技术. 在pc web上, 使用设备指纹算法, 可以精确的匹配到一个设备. 但是在mobile上面, Apple做了很多安全限制, 导致很多字段获取不到, 也就无法生存唯一的设备标识.
但是在统计学上, 其实不需要完全唯一标记, 因为每天的新客不会有很多. 可以尽量区分用户, 再增加一个失效时间的维度, 就足以完成设备匹配.

经过实践, 可以使用下面几个字段组合起来作为deeplink的数字指纹:

* inner ip. 设备内网ip
* outer ip. 出口ip
* ua. ua中包含了设备型号, 操作系统等信息.
* timestamp. 时间维度可以动态调整, 时间长, 冲突的概率高, 时间短容易影响匹配. 推荐使用30分钟作为时间窗口.

使用device fingerprint技术作为shared cookie的一种补充方案, 在后台匹配不到shared cookie时计算fingerprint做匹配, 在之前的方案中增加几个环节. 如下图所示:

![reserved deeplink ios](img/reserved-deeplink-2.png)

# iOS上deeplink整合方案

看到这里, 应该对deeplink和reserved deeplink有所了解了. 可以把几种方案整合在一起, 首先将universal link的服务做成通用服务, 一个domain下面可以支持多个deeplink, 多个app. 要做到这点可以将deeplink连接编码后拼接到url中, deeplink成功后, app中做解析跳转.

![full link](img/full-link.png)
