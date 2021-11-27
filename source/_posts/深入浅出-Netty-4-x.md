---
title: 深入浅出 Netty 4.x
tags: Netty
categories: 技术
abbrlink: 44829
date: 2015-11-11 22:33:31
---
**什么是Netty？**

Netty最初是由JBOSS提供的一个基于NIO的Java开源框架。Netty提供了异步的、基于事件驱动的网络通信框架，用以快速开发高性能、高可靠性的网络服务器和客户端程序dsf。后来Netty项目从JBOSS独立出来，其现在的官网是：http://netty.io/
<!--more-->
**哪里有Netty的中文入门文档？**

极客学院贡献了Netty官方文档的汉化版本，地址在：[Netty 4.x 用户指南](http://wiki.jikexueyuan.com/project/netty-4-user-guide/)

**这篇Blog将带来什么？**

本文并非是一篇入门文档，而是讲解一些Netty的工作原理和应用场景。

**正文**

Netty是典型的[Reactor](http://dl2.iteye.com/upload/attachment/0035/6239/d5ffb4de-ba22-380d-9017-baf5aefcb85d.pdf)模式，直译为反应器模式，实际上就是一种异步处理模型。这里的异步是相对于传统的sockt通信模型而言的。

这里有一篇博文通俗易懂地描述了Reactor模型，推荐看下：[《Reactor模式，或者叫反应器模式》](http://daimojingdeyu.iteye.com/blog/828696)

# Netty 重要概念

> EventLoopGroup

*Special EventExecutorGroup which allows registering Channels that get processed for later selection during the event loop.*

一种特别的EventExecutorGroup（事件执行器组），它可以注册到在事件循环中选择Channel。


> ServerBootstrap

ServerBottstrap是继承自AbstractBootstrap。官方文档中并未对前者有任何描述，因此我们只能查看其父类AbstractBootstrap 的说明。

AbstractBootstrap 是能够更便捷地创建一个Channel的工具类。它通过链式调用的方式让复杂的Channel配置变得更简单。
而如果当前实例并不是一个ServerBottstrap的话，我们可以通过bind()方法来方便地创建无连接的信息传输，例如：UDP。（这段不是很懂）



> ChannelFuture

在Netty中的所有的I/O操作都是异步执行的，这就意味着任何一个I/O操作会立刻返回，不保证在调用结束的时候操作会执行完成。

那么怎样才能知道请求执行的结果呢？

Netty会返回一个ChannelFuture的实例，通过这个实例可以获取当前I/O操作的结果或者状态。

一个ChannelFuture实例的结果，要么是complete，要么是uncomplete。

当一次I/O操作开始时，一个新的Future对象（ChannelFuture继承自Future）即被创建了。
此时，这个新的Future对象初始化标记为uncomplete，也就是说既没有成功，也没有失败，或者被取消操作，因为当前的I/O操作还没有结束。

而一旦当前I/O操作结束的时候，不管结果是成功、失败或者取消，Future都会被标记为Complete，并将成功或失败信息填充到该Future中。


> 未完待续
