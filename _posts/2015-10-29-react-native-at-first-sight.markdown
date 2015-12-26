---
layout:     post
title:      "React Native at first sight "
subtitle:   "Change is the only constant."
date:       2015-10-12 20:27:00
author:     "Nickolas Hu"
header-img: "img/post-bg-rnfs.jpg"
---

## what is React Native?
跟据官方的描述, React Native是一套使用 React 构建 Native app 的编程框架.  推出不久便引发了广泛关注, 这也得益于 JavaScript 开放而活跃的技术社区和 React Native 完备的技术体系支持. 本文试图从概括的介绍 React Native.

React Native 主要是一套 Runtime, 包括一套 React.js 库使得开发可以用 React 的方式开发, 以及一套 Native 的Ployfill 弥补webkit 渲染&&交互能力的不足.

javascript runtime运行在 JavaScriptCore 上, JavaScriptCore 是WebKit的 JavaScript 引擎, 也就是 Safari 中的 JavaScript 引擎, Apple 把他叫做 Nitro. JSC 自从2008年诞生, 至今做了很多优化, 成熟的执行引擎保证了运行效率. RN 本身没有和 js 引擎绑定, 选择 JavaScriptCore 的原因是 iOS 对 JSC 的天然支持.

## how React Native works?
React Native 通过执行javascript, 计算出布局 样式 事件处理的信息, 保存到一个类似于DOM的数据结构(ReactElement tree)中. 通过pure native组件渲染出UI. 事件处理上也是首先native先处理, 再根据javascript的事件监听调用响应的callback. 是将javascript的逻辑能力和native的性能的折中整合.

在组件开发和复用方面, ReactNative 采用了 RequireJS 对组件的组织方式. RequireJS 在 node 中广泛使用, 同 npm 结合让组件的重用变得简单. 一个文件就是一个组件, 组件可以发布到 npm 仓库, 通过配置描述组件名称和版本, 其他开发可以直接引用现有的组件. 同 cocoapods 和 maven 类似, 解决了 js 代码依赖管理和协作的问题.

在组件开发上面, UI 开发方式上面沿用了 React 的优秀设计,  React.js 在 javascript 界掀起了一场热潮, 将 面向DOM 的开发方式变为以面向组件的开发. 这倒是和 native 的开发模式很像, 先设计静态组件样式, 再处理交互流程, 抽离可变状态. 但是 React 加上 javascript 的编程能力是 Java 和 OC 不能比的... React 利用 Babel 支持了 ES6的语法, 并为了简化 DOM 的编写设计了 JSX, 这些大大简化了开发量, 并使得开发变得有趣. React 也采用了依赖反转的设计, DOM 重绘, 组件的生命周期, 都是由React.js控制, 开发只需要关心UI 和交互设计, 可以参考 iOS 中的各种生命周期的 delegate 理解.

Native和javascript的通信是通过native调用javascript实现的, 调用只能是单向的, 总是从native到js. 这套机制目前已经比较成熟,  但RN通过实现对象传递和batch处理js回调的方式解决了双向通信和性能的问题, 对于开发者就像能够做到双向的调用一样. 在 Native 部分, RN 提供了 javascirpt 和 Native 通信时对象传递的方案, 并通过单向调用返回注册callback的方式, 让native在恰当的时机回调js方法, 并且通过让Native 组件运行在独立线程的方式和batch调用callback的方式优化整体的性能. 

RN 官方提供了很多 Native 的组件, 并且由于其开放性, 网上有很多 react-parts 这些开源的组件库, 开发者可以用他们做出精细的交互设计.

## Disadvantage
loadtime问题, 由于RN是以javascript为主体的设计, 页面渲染需要先执行一次javascript render调用. 目前经过一些优化后(framework抽离复用, js资源cache)在iPhone5上面测试简单的页面渲染, js执行的时间要200ms左右, 这个时间主要是在第一次js执行的开销. 这个时间对于用户是可以感知的, 而且会随着js的复杂增长.

内存开销. 由于依赖了js引擎, 在内存占用上会明显高于pure native, 这对于android的低端机而言问题尤其严重.

功耗, 由于额外的js的执行频繁的通信需要的序列化操作都会造成功耗的上升, 虽然没有亲测过具体的差距, 但功耗也是值得关注的问题之一.

虽然存在这些问题, 但动态能力对于native开发而言的诱惑太大, native开发还是需要根据要解决的问题选择合适的技术. 对于js开发而言, 使用自己熟悉的环境能够大幅提升在客户端的体验, 是非常棒的技术.

## Reference
https://facebook.github.io/react-native/
http://trac.webkit.org/wiki/JavaScriptCore
http://ariya.ofilabs.com/2012/06/nitro-javascriptcore-and-jit.html
http://jlongster.com/Removing-User-Interface-Complexity,-or-Why-React-is-Awesome
https://news.ycombinator.com/item?id=8964935
http://tadeuzagallo.com/blog/react-native-bridge/


