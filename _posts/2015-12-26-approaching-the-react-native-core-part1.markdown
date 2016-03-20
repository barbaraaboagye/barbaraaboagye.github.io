---
layout:     post
title:      "Approaching the React Native Core (1) ---- from javascript to native view."
subtitle:   ""
date:       2015-12-26 16:15:15
author:     "Nickolas Hu"
header-img: "img/post-bg-rnfs.jpg"
---

## 介绍(闲扯)
React Native是javascript和native混合编程的一套框架, 是由facebook开源出来的, 通过javascrip和React提供跨平台的native开发支持. 之前有[一篇RN的简单介绍](http://nickolashu.github.io/2015/10/12/react-native-at-first-sight/), 由于目前网络上对RN深入的分析较少(只有几篇内部的开发分享很赞, 列在了参考中), 其他大多是偏于主观感受. 因此试图整理一篇偏干的文章, 分析RN到底做了什么, 如有纰漏欢迎讨论.

本文不是入门教程的文章, 喜欢看入门操作教程的朋友们可以直接去[RN官方站点](https://facebook.github.io/react-native/docs/getting-started.html), 介绍的很详细, 而且在不断更新. 因为我对iOS更熟悉, 所以分析是由iOS的视角切入的, 各位javascript朋友请不吝赐教. 使用的RN版本是目前最新的v0.17.0-rc版本.

## 正文

![performance log](img/rn-execution.png "Performance Log")

从RN运行的Performance记录, 可以看到一次RN执行的几个主要环节. 获取js -> 执行js -> 初始化nativeModule -> 配置nativeModule. 从`RCTPerformanceLogger.h`中可以看到几个环节的定义, 可以通过跟踪这几个定义看到整个过程.

### 1 获取js

第一次加载时(和修改js后), js文件会在node server上编译完成, 编译的过程主要是使用babel interpreter生成一个JSC能够执行的文件. 获取的过程就是通过外部传入的url请求获取js文件. 实现在`RCTJavaScriptLoader`文件中, 简化后的代码如下, 支持从本地和http服务两种方式获取js.

    {% highlight objective-c %}
    // 从scriptURL获取js脚本, 支持从本地加载资源和http资源两种, 通过URL schema区分.
    + (void)loadBundleAtURL:(NSURL *)scriptURL onComplete:(RCTSourceLoadBlock)onComplete
    {
      // Load local script file
      if (scriptURL.fileURL) {
        NSString *filePath = scriptURL.path;
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
          NSError *error = nil;
          NSData *source = [NSData dataWithContentsOfFile:filePath
                                                  options:NSDataReadingMappedIfSafe
                                                    error:&error];
          onComplete(error, source);
        });
        return;
      }

      // Load remote script file
      NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithURL:scriptURL completionHandler:
                                    ^(NSData *data, NSURLResponse *response, NSError *error) {
        onComplete(nil, data);
      }];

      [task resume];
    }
    {% endhighlight %}

### 2 执行js

这部分是RN的核心流程. 介绍执行之前需要了解下ios是如何调用js的. iOS7以后, Apple推出了JavaScriptCore framework, 封装了[WebKit的js引擎](http://trac.webkit.org/wiki/JavaScriptCore), 提供了iOS上执行js的能力. JSC上有几个关键概念.

* JSVirtualMachine
* JSContext
* JSValue

`JSVirtualMachine`封装了js的虚拟机, 处理内存管理和gc. 如果需要支持多线程的js执行, 可以创建多个`JSVirtualMachine`.
`JSContext`是javascript的运行环境, 用来执行js和保存变量, 相对于web开发中的window对象. 一个`JSVirtualMachine`可以对应多个`JSContext`, 不同的`JSContext`间可以通信, 但`JSVirtualMachine`之间是完全隔离的.
`JSValue`是`JSContext`中的对象的封装. `JSContext`中可以创建任意个`JSValue`.
详细的JSC使用知识超越了本文的范围, 想了解的朋友可以参考其他的资料[OC](https://infinum.co/the-capsized-eight/articles/running-javascript-in-an-ios-application-with-javascriptcore), [swift](http://nshipster.com/javascriptcore/)

好了, 看看怎么做的执行部分. 概括来讲, 首先会执行前面拿到的js, 执行完会发出一个`RCTJavaScriptDidLoadNotification`消息. 接着会调用js的`@"AppRegistry.runApplication"`方法, 这个方法会返回一个结果给到native, native根据这个结果做渲染.下面详细看看各个过程.

#### 1 执行js 调用栈如下
| `RCTContextExecutor` `executeApplicationScript:sourceURL:onComplete:` 执行js代码
| -- | `enqueueApplicationScript:url:onComplete`
| -- | -- | `RCTBatchedBridge` `executeSourceCode:`

RN的js的执行是在`RCTContextExecutor`中做的, 方法如下. RN中执行js的操作都是在一个独立的javascript线程中处理的. 后面也可以看到, RN中有3个主要的线程.

* com.facebook.React.JavaScript js代码运行的线程
* com.apple.main-thread 主线程, 所有UIKit相关处理
* com.facebook.React.ShadowQueue 布局相关处理

因此在监控性能的时候, RN中除了主线程的fps, 还可以看到JavaScript线程的fps, 这两部分都会影响整体的性能情况.

    {% highlight objective-c linenos %}
    // 简化过的执行方法 RCTContextExecutor
    - (void)executeApplicationScript:(NSData *)script
                           sourceURL:(NSURL *)sourceURL
                          onComplete:(RCTJavaScriptCompleteBlock)onComplete
    {
        __weak RCTContextExecutor *weakSelf = self;
        [self executeBlockOnJavaScriptQueue:^{
            RCTContextExecutor *strongSelf = weakSelf;

            // JSStringCreateWithUTF8CString expects a null terminated C string
            NSMutableData *nullTerminatedScript = [NSMutableData dataWithCapacity:script.length + 1];

            [nullTerminatedScript appendData:script];
            [nullTerminatedScript appendBytes:"" length:1];

            JSValueRef jsError = NULL;
            JSStringRef execJSString = JSStringCreateWithUTF8CString(nullTerminatedScript.bytes);
            JSStringRef jsURL = JSStringCreateWithCFString((__bridge CFStringRef)sourceURL.absoluteString);
            // 前两个参数是js执行的context和执行的代码, 后面的参数都是错误处理用的
            JSValueRef result = JSEvaluateScript(strongSelf->_context.ctx, execJSString, NULL, jsURL, 0, &jsError);
            JSStringRelease(jsURL);
            JSStringRelease(execJSString);

            onComplete(error);

        }];
    }
    {% endhighlight %}

2 执行完成后通知js app启动

调用栈大致如下, 省略了中间几个方法.
| `RCTBatchedBridge` `executeSourceCode:` 发送完成消息
|... |
| -- | -- | `RCTBatchedBridge` `- (void)enqueueJSCall:args:` 执行`@"AppRegistry.runApplication"`方法

3 返回结果的格式
在 _executeJSCall的 `@"AppRegistry.runApplication"`方法时断点可以获取返回值



## External links
* Tadeu Zagallo, FB RN团队的开发分享的原理 http://tadeuzagallo.com/blog/react-native-bridge/
* RN社区翻译的中文文档 http://reactnative.cn/docs/getting-started.html
* 为什么Browser的响应速度比native慢 http://stackoverflow.com/questions/10731934/why-html-web-ui-response-slower-than-native-ui#

## 回收站
可以发现执行完成的结果`result`并没有返回, 看堆栈上调用这个方法的, 是一个`RCTBridge`对象, 在执行完成后做了一次`flushQueue`操作.

    {% highlight objective-c linenos %}
    - (void)enqueueApplicationScript:(NSData *)script
                                 url:(NSURL *)url
                          onComplete:(RCTJavaScriptCompleteBlock)onComplete
    {    
      // 上面的js的执行方法
      [_javaScriptExecutor executeApplicationScript:script sourceURL:url onComplete:^(NSError *scriptLoadError) {
        [_javaScriptExecutor flushedQueue:^(id json, NSError *error)
         {
           [self handleBuffer:json batchEnded:YES];
           onComplete(error);
         }];
      }];
    }
    {% endhighlight %}

native调用js都是通过下面这个方法调用的, 调用`method`方法, 返回JSON对象(如果有的话).js方法都是是挂在`__fbBatchedBridge`这个module下面的, 因此先获取module, 再获取方法.

    {% highlight objective-c linenos %}
    - (void)_executeJSCall:(NSString *)method
                 arguments:(NSArray *)arguments
                  callback:(RCTJavaScriptCallback)onComplete
    {
      __weak RCTContextExecutor *weakSelf = self;
      [self executeBlockOnJavaScriptQueue:^{
        RCTContextExecutor *strongSelf = weakSelf;
        NSError *error;
        NSString *argsString = (arguments.count == 1) ? RCTJSONStringify(arguments[0], &error) : RCTJSONStringify(arguments, &error);

        JSValueRef errorJSRef = NULL;
        JSValueRef resultJSRef = NULL;
        JSGlobalContextRef contextJSRef = JSContextGetGlobalContext(strongSelf->_context.ctx);
        JSObjectRef globalObjectJSRef = JSContextGetGlobalObject(strongSelf->_context.ctx);

        // 1 get the BatchedBridge object
        JSStringRef moduleNameJSStringRef = JSStringCreateWithUTF8CString("__fbBatchedBridge");
        JSValueRef moduleJSRef = JSObjectGetProperty(contextJSRef, globalObjectJSRef, moduleNameJSStringRef, &errorJSRef);
        JSStringRelease(moduleNameJSStringRef);

        if (moduleJSRef != NULL && errorJSRef == NULL && !JSValueIsUndefined(contextJSRef, moduleJSRef)) {

          // 2 从BatchedBridge获取方法的引用
          JSStringRef methodNameJSStringRef = JSStringCreateWithCFString((__bridge CFStringRef)method);
          JSValueRef methodJSRef = JSObjectGetProperty(contextJSRef, (JSObjectRef)moduleJSRef, methodNameJSStringRef, &errorJSRef);
          JSStringRelease(methodNameJSStringRef);

          // 3 调用js函数
          // 这里省略了多参数的处理逻辑, 感兴趣的朋友看代码吧
          resultJSRef = JSObjectCallAsFunction(contextJSRef, (JSObjectRef)methodJSRef, (JSObjectRef)moduleJSRef, 0, NULL, &errorJSRef);
        }

        // 4 序列化返回值
        id objcValue;
        if (!JSValueIsNull(contextJSRef, resultJSRef)) {
          JSStringRef jsJSONString = JSValueCreateJSONString(contextJSRef, resultJSRef, 0, nil);
          if (jsJSONString) {
            NSString *objcJSONString = (__bridge_transfer NSString *)JSStringCopyCFString(kCFAllocatorDefault, jsJSONString);
            JSStringRelease(jsJSONString);

            // string -> JSON
            objcValue = RCTJSONParse(objcJSONString, NULL);
          }
        }
        // objcValue就是js的返回值, 一个json对象
        onComplete(objcValue, nil);
      }];
    }
    {% endhighlight %}


(`RCTBatchedBridge`的`executeSourceCode:`方法中) 发app加载完成消息
