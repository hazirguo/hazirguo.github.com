---
layout: post
title: "Linux 下系统调用的三种方法"
description: ""
category: Linux 
tags: [syscall, kernel]
---
{% include JB/setup %}

系统调用（System Call）是操作系统为在用户态运行的进程与硬件设备（如CPU、磁盘、打印机等）进行交互提供的一组接口。当用户进程需要发生系统调用时，CPU 通过软中断切换到内核态开始执行内核系统调用函数。下面介绍Linux 下三种发生系统调用的方法：

## 通过 glibc 提供的库函数
glibc 是 Linux 下使用的开源的标准 C 库，它是 GNU 发布的 libc 库，即运行时库。glibc 为程序员提供丰富的 API（Application Programming Interface），除了例如字符串处理、数学运算等用户态服务之外，最重要的是封装了操作系统提供的系统服务，即系统调用的封装。那么glibc提供的系统调用API与内核特定的系统调用之间的关系是什么呢？

* 通常情况，每个特定的系统调用对应了至少一个 glibc 封装的库函数，如系统提供的打开文件系统调用 `sys_open` 对应的是 glibc 中的 `open` 函数；
* 其次，glibc 一个单独的 API 可能调用多个系统调用，如 glibc 提供的 `printf` 函数就会调用如 `sys_open`、
`sys_mmap`、`sys_write`、`sys_close` 等等系统调用；
* 另外，多个 API 也可能只对应同一个系统调用，如glibc 下实现的 `malloc`、`calloc`、`free` 等函数用来分配和释放内存，都利用了内核的 `sys_brk` 的系统调用。
