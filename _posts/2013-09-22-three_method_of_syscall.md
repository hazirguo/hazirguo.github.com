---
layout: post
title: "Linux 下系统调用的三种方法"
description: ""
category: Linux 
tags: [syscall, kernel]
---
{% include JB/setup %}

系统调用（System Call）是操作系统为在用户态运行的进程与硬件设备（如CPU、磁盘、打印机等）进行交互提供的一组接口。当用户进程需要发生系统调用时，CPU 通过软中断切换到内核态开始执行内核系统调用函数。下面介绍Linux 下三种发生系统调用的方法：

