---
title: iOS【高性能iOS应用开发】
date: 2023-08-16 13:28:24
tags:
- 图书
- 专业技术
- iOS
---

### 1 移动应用的性能

- 应用间互操作和数据共享的机制，UIActivityViewController、MutipeerConnectivity 框架。
- 单点登录(single sign-on，SSO)
- Apple 提供了下载崩溃报告的服务，iTunes Connect中。用户可选择是否开启分享崩溃数据。设置里开启，针对所有应用生效。
- CocoaLumberjack 第三方库，有内置日期记录器。DDLogError、DDLogWarn、DDLogInfo、DDLogDebug、DDLogVerbose。可以向ASL(Apple System Log，NSLog方法的默认位置)记录日志，或向文件系统记录日志。（19页代码）

### 2 内存管理

- RAM 运行内存。线程有栈空间，栈帧消耗内存，方法中的变量占用栈内存，视图层级过多导致栈溢出。
- 1G RAM 分配给一个应用的堆大小不超过512M，操作系统管理堆。大量使用图片需要关注内存平均值和峰值。对象分配在堆中。栈变量可复制到堆，堆内容也可复制到栈。
- 
