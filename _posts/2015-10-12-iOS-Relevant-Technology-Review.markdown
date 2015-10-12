---
layout:     post
title:      "iOS Relevant Technology Review ---- SPDY"
subtitle:   "Change is the only constant."
date:       2015-10-12 20:27:00
author:     "Nickolas Hu"
header-img: "img/post-bg-03.jpg"
---

###设计概述

SPDY的目标是降低页面加载时间。通过实现prioritizing和multiplexing页面资源的传输，一个client只需要一个连接。TLS加密在SPDY实现中几乎无处不在，传输头做了gzip或DEFLATE压缩(相对于HTTP，HTTP头是明文传输)。此外，服务端可以示意或者主动push而不是等待每个资源的请求。

SPDY依赖SSL/TLS，不支持直接操作TCP。对于SSL的要求是出于安全以及避免通过代理对话的不一致问题。[1][[1]]


###相对于HTTP

HTTP的瓶颈之一是依赖多个链接实现并发。这造成了很多问题，比如，增加额外的round trip以建立链接，slow-start延迟，用于避免同一服务器打开太多链接而限制客户端的链接配给问题。HTTP pipelining有所帮助，但是仅达到了部分multiplexing(多路复用)。并且，pipelining由于中间干扰被证明是现有浏览器中不可用的。

SPDY没有代替HTTP，它只是改变了HTTP请求和回应在网络中发送的方式。
SPDY增加了一个framing layer，使得并发的streaming可以在一个TCP连接上进行(或任意的可靠的传输层stream)[3]。这意味着所有现有的HTTP服务端应用可以直接使用。

SPDY实际上是HTTP和HTTPS协议的通道。当通过SPDY请求时，HTTP请求被处理、tokenized、简化和压缩。例如，每个SPDY endPoint记录前几次发出的请求头，从而避免发送重复的请求头，只要头部未改变，而必须发送的请求头会被压缩。[2][2]

总结一下，SPDY在HTTP之上提供了四个实现：
- 请求多路复用：一个SPDY连接可以支持不限个数的并发请求。
- 优先请求：客户端可以控制请求优先级，避免网络信道被非关键次元占用而使得高优先级请求等待。
- 压缩头：单个页面可能请求50~100个子资源，HTTP头的数据量是很可观的。
- Server push流：支持服务端推送数据到客户端而无需客户端请求
SPDY师徒保留HTTP现有的语义。所有的特性例如cookies，ETags，Vary headers，Content-Encoding 协商等工作同HTTP一样，SPDY只是替换了写入数据到网络的方式。[3][3]

###相对于http/2
比较http/2 http1.1-14 SPDY. HTTP/2在HTTP头压缩上效果最好, 因为使用了针对性优化的HPACK 协议. Response Size上面SPDY会更小, 因为SPDY会为了安全做数据填充.[4][4]

[1]:  http://en.wikipedia.org/wiki/SPDY#Design
[2]:  http://en.wikipedia.org/wiki/SPDY#Relation_to_HTTP
[3]:  http://www.chromium.org/spdy/spdy-protocol/spdy-protocol-draft3-1
[4]:  https://blog.httpwatch.com/2015/01/16/a-simple-performance-comparison-of-https-spdy-and-http2/comment-page-1/

