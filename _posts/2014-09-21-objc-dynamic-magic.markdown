---
layout:     post
title:      "Objective-C Dynamic Magic"
subtitle:   "The Devil is in the detail"
date:       2014-09-21 12:00:00
author:     "Nickolas"
header-img: "img/objc-dynamic-magic.jpg"
---

## Intro
动态性是相对于编译、链接期而言的，在运行时决议，是objective-c强大的特性。许多面向对象的语言都有动态性特点，比如java、c++，而objective-c是基于c实现的，在动态性上也有很多相似之处，但objective-c也有自己明显的特点。Objective-C是一个语言扩展的集合，足够有意义认为是一个不同的语言。他是严格的C的超集。平时我们经常使用动态性，比如向下转型之前会使用[isKindOfClass:](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Protocols/NSObject_Protocol/Reference/NSObject.html#//apple_ref/occ/intfm/NSObject/isKindOfClass:)判断对象是否为指定类型，或者使用[respondsToSelector:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Protocols/NSObject_Protocol/index.html#//apple_ref/occ/intfm/NSObject/respondsToSelector:)判断对象是否实现了某个方法，了解动态性可以让我们知道这些方法背后的原理。除此之外，还可以了解到一些objective-c最佳实践的原因，比如为什么objective-c的函数调用不需要判空。动态性更是基础库的利器，能做出很多强大的功能，也正是因为它过于强大并有些复杂官方是不推荐在app开发中过度使用动态性编程，下面我们一起来了解下。

## 基础结构
首先我们要了解下NSObject类的结构。在NSObject.h中可以看到NSObject的定义。

    {% highlight objective-c linenos %}
    @interface NSObject <NSObject> {
       Class isa;
    } 
    typedef struct objc_class *Class;
    {% endhighlight %}

可以看出NSObject只有一个成员变量isa，而Class是一个指向objc_class的指针。换句话说，NSObject对象就是指向objc class定义的指针。
而objc_class的定义如下：

    {% highlight objective-c linenos %}
    struct objc_class {
        Class isa;
        Class super_class;
        const char *name;                                        
        long version;                                            
        long info;                                     
        long instance_size;                           
        struct objc_ivar_list *ivars;                  
        struct objc_method_list **methodLists;                   
        struct objc_cache *cache;                   
        struct objc_protocol_list *protocols;
    };
    {% endhighlight %}
objc_class的定义可以在[apple运行时开源代码](http://www.opensource.apple.com/source/objc4/objc4-208/runtime/objc-class.h)中看到。这里我们看到objc class定义了运行时需要的所有信息，其中包括，指向父类定义的指针，支持哪些protocols，成员方法以及成员变量。

其中比较有趣的是objc_class的第一个变量也是一个Class对象，前部分的内存布局和NSObject一样，这意味着Class类型本身也是一个对象，所有objc对象的操作同样适用于objc Class，比如发送消息对于Class和对象是一致的。这样增加了一致性，减少了区分Class和对象的特殊逻辑。

Class的isa对象指向的是metaclass的Class，也其中包含了类的静态成员和方法。也就是说objc的class包含了实例class和metaclass。那么metaclass的isa指向什么呢？是nil还是metametaclass？其实metaclass的isa指向自身，这样整体上一致性增强了。

继承NSObject的对象，内存布局上成员变量会附加到isa后面，基类在继承类前面。布局类似于c++的内存布局。

Objective-c classobject关系可以参照下图。
![Alt text](http://images.cnblogs.com/cnblogs_com/studentdeng/201110/201110011842504595.png  "classobject关系.")

简单解释下此图。实线表示的是继承关系，比如子类class继承于父类class，父类class继承于root class。虚线表示的是isa关系，isa是Objective-C的指针，指向的是描述自己的对象，isa对象中的描述信息包括方法、实例对象、空间大小、布局等，比如实例对象的isa指向的是class对象，描述了实例对象支持的方法、实例对象；而class对象的isa指向的是metaclass，描述了class方法、class对象。当给实例对象发消息时，objc_msgSend()会首先在其isa对象(class对象 T)中查看其方法表，类似C++中vtable。再查看其T的父类的方法表，如果有的话。当给class对象发消息时同样的会查看其isa对象，即metaclass的方法表，再查看metaclass的父类的方法表。

而所有metaclass的isa都是指向Root metaclass的，包括Root metaclass自身，但这不重要，因为不会有人调用metaclass方法。重要的是其superclass，Root meta class的superclass是Root class。因为这样每个metaclass都是继承自Root class的，可以继承Root class的方法。而实际上所有的class object要么是Root class的实例，要么是Root class的subclass。这样整个体系就完整和优雅的。

还有一点有意思的，当我们调用class对象的class方法时，[ClassA class]，实际上是等同于调用[ClassA self]，而不是返回其isa对象，这是违反meta schema，这是apple的设计人员在实践中对meta理论做出的妥协，隐藏了meta class，使得整个设计更实用而不是那么的meta。

## 消息机制 
以上就是objc的对象布局，下面看下消息机制。由于objc是基于c的，当我们在@implementation...@end中间实现函数的时候，编译器其实会把函数转换成c函数。转换的方式就是会增加两个额外的参数，self和_cmd，同时去掉“[]-”这些字符。这样objc方法就变成了c方法，如果我们有办法拿到对应的函数指针，我们可以直接用c的方式去调用。

在objc的术语中，实际的c的函数被叫做IMP。c方法其实在objc中是存在的，如果你nm一个objc的dylib，可以看到_开头的c方法。那么[]的objc函数是如何执行的呢？

其实编译器把[]方法转换成objc_msgSend()方法，objc_msgSend()的前两个参数是，消息的接收者，和一个selector，也就是一个方法名。

理论上可以简单地认为一个selector就是一个c string。实际上它的内存模型和c string一样，是以NUL结尾的char* 指针。唯一的区别是，objc运行时保证，每个selector会只有唯一一个实例，即在整个内存地址空间中，每个方法都有独一无二的实例。这样做的原因是，假如使用c string，那么在调用函数时需要使用类似于strcmp这种方法来判断函数调用，这种char by char的对比方式实在慢的可笑。而当每个函数的地址都唯一时，可以简单地用指针比较来比较函数，这样快的多。因此selector的类型是SEL，而不是char*，而我们可以使用sel_registerName()函数将一个c string转化为selector。

继续来看objc_msgSend()方法，它是一个varargs函数，除了前两个参数以外，剩下的都是函数原有的参数。在objc运行时库中可以看到它的定义。

    {% highlight objective-c linenos %}
    id objc_msgSend(id receiver, SEL name, arguments...)
     {
        IMP function = class_getMethodImplementation(receiver->isa, name);
        return function(arguments);
     }
    IMP class_getMethodImplementation(Class cls, SEL name);
    {% endhighlight %}
    
理论上这个方法就是这样，因为编译过程中会做各种性能的优化。在objc的运行时中有个方法叫做class_getMethodImplementation()，它可以通过查照class的函数表，找到指定的selector然后返回。这样得到了一个IMP，而IMP实际上就是一个函数指针，可以像C方法一样调用。因此，objc_msgSend()做的就是获取消息接收者的class object通过isa指针，并且找到selector对应的IMP。这样我们就做到了消息发送，实际上没什么black magic。

## Dynamic & Reflective 动态性和反射
objc的内存模型和消息机制使得class object包含了类方法的所有信息，并且这些信息可以通过objc运行时api获取到，可以说objc是支持反射（自省）的。比如我们可以在运行时获取指定类型的继承结构，所有方法，所有protocol。

运行时API还可以修改class object，而消息发送是由一个C方法实现的，因此objc的动态性非常强。比如我们可以在运行时增加方法实现，甚至将预定义的方法实现换成我们自己的。objc的消息机制设计了多个处理函数的时机，如果对象接收到的消息无法识别，运行时会调用forwardInvocation:(resolveInstanceMethod:)，这里可以决定如何处理消息，甚至可以将消息交给另一个对象处理。

这个特性使得objc同其他动态语言和脚本语言的动态性一样强，如Perl，Ruby，Python，PHP，Javascript等，而objc的不同是会编译为native code。（性能比JIT engines要强），并且获得了与其他脚本语言一样的动态性。相比之下，C++几乎没有introspection能力（除了RTTI）和dynamism。而Java具有反射和动态性，但是没有类似于-forwardInvocation:方法。

That's all.
    
## Reference && Further Reading
- [understanding objective c runtime](http://cocoasamurai.blogspot.com/2010/01/understanding-objective-c-runtime.html)
- [10 ios questions](http://onevcat.com/2013/04/ios-interview/)
- [Official runtime programming guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)
- [objective-c internals](http://algorithm.com.au/downloads/talks/objective-c-internals/objective-c-internals.pdf)
