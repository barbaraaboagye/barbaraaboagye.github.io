---
layout:     post
title:      "Several Q&A about ARC iOS developer should take care of."
subtitle:   "There are no two words in the English language more harmful than good job."
date:       2014-08-24 12:00:00
author:     "Nickolas Hu"
header-img: "img/post-bg-04.jpg"
---

#### What is ARC?
ARC是"Automatic Reference Counting"的缩写,是内存管理的一种方式. LLVM编译器通过引用计数管理内存, 代替人工引用计数(MRC). 

#### Why ARC?
相比shared_ptr这种MRC的方式, ARC降低了人犯错的可能. 除了引用计数管理内存, 还有通过GC的方式, 最典型的是JVM, 相比之下GC的优点是更不容易造成内存泄露, 开发人员无需指定变量的持有情况, 也无需关心循环引用的问题. 而缺点是GC在执行时往往需要暂停其他线程的执行(stop the world), 会影响ui响应效率.   
apple工程师很善于通过编译期来做优化, 降低运行时的开销(Swift).

#### What ARC does not do?
ARC不是什么高级的AI魔法, 在内存模型上, ARC和MRC/plain code是一样的, 实际上每一个.o文件都可以设置是否使用arc, (XCode4以上默认开启, 可以设置每个.m的compiler flag="-fno-objc-arc"关闭ARC). 这也说明了ARC是在compile时期做的, 在linking时可以混用. 不过Apple的工程师通过非常优秀的编译期Clang(也叫做LLVM), 降低了自动retain release带来的次数, 使得遍出来的代码和人工RC差异不大.[1][1]  
ARC不会处理非NSObject的对象, 也就是说plain值, core foundion object不能通过ARC管理内存, ARC是依赖NSObject实现的.


#### Reference
[1](http://arstechnica.com/apple/2011/07/mac-os-x-10-7/10/#arc)
