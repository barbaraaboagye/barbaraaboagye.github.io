---
layout:     post
title:      "React Native at first sight "
subtitle:   "Change is the only constant."
date:       2015-10-12 20:27:00
author:     "Nickolas Hu"
header-img: "img/post-bg-03.jpg"
---

## what is React Native?
跟据官方的描述, React Native是一套使用 React 构建 Native app 的编程框架.  虽然推出不久变引发了广泛关注, 得益于 JavaScript 开放而活跃的技术体系, React Native 有完备的技术体系. 本文试图从概括的角度解释 React Native 的技术体系和设计思路, 对于细节解释的可能不完全准确.

React Native 主要是一套 Runtime, 包括一套 React.js 库使得开发可以用 React 的方式开发, 以及一套 Native 的Ployfill 弥补webkit 渲染&&交互能力的不足.

javascript runtime运行在 JavaScriptCore 上, JavaScriptCore 是WebKit的 JavaScript 引擎, 也就是 Safari 中的 JavaScript 引擎, Apple 把他叫做 Nitro. JSC 自从2008年诞生, 至今做了很多优化, 成熟的执行引擎保证了运行效率. RN 本身没有和 js 引擎绑定, 选择 JavaScriptCore 的原因是 iOS 对 JSC 的天然支持.

## how React Native works?
ReactNative 采用了 RequireJS 对组件的组织方式. RequireJS 在 node 中广泛使用, 同 npm 结合让组件的重用变得简单. 一个文件就是一个组件, 组件可以发布到 npm 仓库, 通过配置描述组件名称和版本, 其他开发可以直接引用现有的组件. 同 cocoapods 和 maven 类似, 解决了 js 代码依赖管理和协作的问题.

UI 开发方式上面沿用了 React 的优秀设计,  React.js 在 javascript 界掀起了一场热潮, 将 面向DOM 的开发方式变为以面向组件的开发. 这倒是和 native 的开发模式很像, 先设计静态组件样式, 再处理交互流程, 抽离可变状态. 但是 React 加上 javascript 的编程能力是 Java 和 OC 不能比的... React 利用 Babel 支持了 ES6的语法, 并为了简化 DOM 的编写设计了 JSX, 这些大大简化了开发量, 并使得开发变得有趣. React 也采用了依赖反转的设计, DOM 重绘, 组件的生命周期, 都是由React.js控制, 开发只需要关心UI 和交互设计, 可以参考 iOS 中的各种生命周期的 delegate 理解.

React 中最重要的一个 delegate 是 render 方法, 它返回一个 ReactElement tree. 在 ReactNative 中这个 ReactElement tree 交给 native 组件做渲染. 

在 Native 部分, RN 提供了 javascirpt 和 Native 通信时对象传递的方案, 并通过让Native 组件运行在独立线程的方式优化整体的性能. 由于 javascript 单线程和虚拟机的限制, cpu 密集型的调用和精致的交互动画设计应该放在 native 端实现. RN 官方提供了很多 Native 的组件, 并且由于其开放性, 网上有很多 react-parts 这些开源的组件库, 开发者可以用他们做出精细的交互设计.

## React Native Key Features
1 dynamic
2 components
3 tools
4 open source

## Reference
https://facebook.github.io/react-native/
http://trac.webkit.org/wiki/JavaScriptCore
http://ariya.ofilabs.com/2012/06/nitro-javascriptcore-and-jit.html
http://jlongster.com/Removing-User-Interface-Complexity,-or-Why-React-is-Awesome
https://news.ycombinator.com/item?id=8964935
http://tadeuzagallo.com/blog/react-native-bridge/


