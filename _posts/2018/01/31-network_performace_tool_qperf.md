---
layout: post
title: "网络性能测试工具 qperf 简介"
description: ""
category: 网络
tags: [bash, Linux, command, network]
---

网络的性能有很多指标，但带宽和延迟是重要的两个指标，Linux 下有很多工具可以用来度量网络的性能，这里首先介绍一款工具——**qperf**。

## 简介

qperf 可以用来测试两个节点之间的带宽（bandwidth）和延迟（latency），不仅仅可以用来测试 TCP/IP 协议的性能指标，还可以用来测试 RDMA 传输的指标。使用方法是：一个节点运行 qperf 作为服务端，另一个节点则运行 qperf 作为客户端，与服务端建立连接之后打流，获取带宽和延迟等数据。

## 安装

在 CentOS 发型版的 Linux 操作系统中安装比较简单，使用官方 yum 源，调用命令 `yum install qperf` 即可安装。

## 使用

如前面介绍，qperf 带不同的参数可以同时作为服务端和客户端：

* **服务端**：

典型的用法不带任何参数，直接运行 `qperf` 命令即启动服务端进程，等待客户端的连接，默认监听端口为 19765。如果想指定监听端口可以增加参数 `--listen_port xxx`，此时要保证客户端也要指定该端口号建立连接。

* **客户端**：

客户端命令格式稍微复杂一些，如下：

``` bash
qperf SERVERNODE [OPTIONS] TESTS
```

其中，

1. `SERVERNODE` 为服务端的地址
2. `TESTS` 为需要测试的指标，使用帮助命令 `qperf --help tests` 可以查看到 qperf 支持的所有测量指标，可以一条命令中带多个测试项，这里介绍常用的有：

    * `tcp_bw` —— TCP流带宽
    * `tcp_lat` —— TCP流延迟
    * `udp_bw` —— UDP流带宽
    * `udp_lat` —— UDP流延迟
    * `conf` —— 显示两端主机配置

3. `OPTIONS` 是可选字段，使用帮助命令 `qperf --help options` 可以查看所有支持的可选参数，这里介绍常用的参数：

    * `--time/-t` —— 测试持续的时间，默认为 2s
    * `--msg_size/-m` —— 设置报文的大小，默认测带宽是为 64KB，测延迟是为 1B
    * `--listen_port/-lp` —— 设置与服务端建立连接的端口号，默认为 19765
    * `--verbose/-v` —— 提供更多输出的信息，可以更多尝试一下 `-vc`、`-vs`、`-vt`、`-vu` 等等

例如：

``` bash
# qperf 192.168.0.8 -t 10 -vvu tcp_lat udp_lat conf
tcp_lat:
    latency   =  57.2 us
    msg_size  =     1 bytes
    time      =    10 sec
    timeout   =     5 sec
udp_lat:
    latency   =  10 sec
    msg_size  =   1 bytes
    time      =  10 sec
    timeout   =   5 sec
conf:
    loc_node   =  performance-north-south-000-0003
    loc_cpu    =  8 Cores: Intel Xeon Gold 6161 @ 2.20GHz
    loc_os     =  Linux 3.10.0-514.10.2.el7.x86_64
    loc_qperf  =  0.4.9
    rem_node   =  performance-north-south-000-0001.novalocal
    rem_cpu    =  8 Cores: Intel Xeon Gold 6161 @ 2.20GHz
    rem_os     =  Linux 3.10.0-514.10.2.el7.x86_64
    rem_qperf  =  0.4.9
```

## 进阶使用方法

qperf 有个比较酷的功能可以循环 loop 遍历测试，这对于摸底网络性能找到最优参数非常有帮助，利用的是其中一个 OPTIONS 参数，使用说明可以参数帮助文档：

> --loop Var:Init:Last:Incr (-oo)
    Run a test multiple times sequencing through a series of values.  Var
    is the loop variable; Init is the initial value; Last is the value it
    must not exceed and Incr is the increment.  It is useful to set the
    --verbose_used (-vu) option in conjunction with this option.

通常可以调优的 `Var` 设为 `msg_size`，看下下面的命令输出结果就一目了然了：

```
# qperf 192.168.0.8 -oo msg_size:1:64K:*2 -vu tcp_bw tcp_lat
tcp_bw:
    bw        =  2.17 MB/sec
    msg_size  =     1 bytes
tcp_bw:
    bw        =  4.13 MB/sec
    msg_size  =     2 bytes
tcp_bw:
    bw        =  7.82 MB/sec
    msg_size  =     4 bytes
tcp_bw:
    bw        =  14.3 MB/sec
    msg_size  =     8 bytes
tcp_bw:
    bw        =  25.8 MB/sec
    msg_size  =    16 bytes
tcp_bw:
    bw        =  42.4 MB/sec
    msg_size  =    32 bytes
tcp_bw:
    bw        =  63.4 MB/sec
    msg_size  =    64 bytes
tcp_bw:
    bw        =  83.3 MB/sec
    msg_size  =   128 bytes
tcp_bw:
    bw        =  388 MB/sec
    msg_size  =  256 bytes
tcp_bw:
    bw        =  816 MB/sec
    msg_size  =  512 bytes
tcp_bw:
    bw        =  941 MB/sec
    msg_size  =    1 KiB (1,024)
tcp_bw:
    bw        =  967 MB/sec
    msg_size  =    2 KiB (2,048)
tcp_bw:
    bw        =  980 MB/sec
    msg_size  =    4 KiB (4,096)
tcp_bw:
    bw        =  1.01 GB/sec
    msg_size  =     8 KiB (8,192)
tcp_bw:
    bw        =  950 MB/sec
    msg_size  =   16 KiB (16,384)
tcp_bw:
    bw        =  1.01 GB/sec
    msg_size  =    32 KiB (32,768)
tcp_bw:
    bw        =  986 MB/sec
    msg_size  =   64 KiB (65,536)
tcp_lat:
    latency   =  61.5 us
    msg_size  =     1 bytes
tcp_lat:
    latency   =  58.2 us
    msg_size  =     2 bytes
tcp_lat:
    latency   =  57.5 us
    msg_size  =     4 bytes
tcp_lat:
    latency   =  57.6 us
    msg_size  =     8 bytes
tcp_lat:
    latency   =  57.9 us
    msg_size  =    16 bytes
tcp_lat:
    latency   =  61.7 us
    msg_size  =    32 bytes
tcp_lat:
    latency   =  58.6 us
    msg_size  =    64 bytes
tcp_lat:
    latency   =  58.3 us
    msg_size  =   128 bytes
tcp_lat:
    latency   =  58.8 us
    msg_size  =   256 bytes
tcp_lat:
    latency   =   60 us
    msg_size  =  512 bytes
tcp_lat:
    latency   =  61.8 us
    msg_size  =     1 KiB (1,024)
tcp_lat:
    latency   =  72.8 us
    msg_size  =     2 KiB (2,048)
tcp_lat:
    latency   =  75.6 us
    msg_size  =     4 KiB (4,096)
tcp_lat:
    latency   =  103 us
    msg_size  =    8 KiB (8,192)
tcp_lat:
    latency   =  132 us
    msg_size  =   16 KiB (16,384)
tcp_lat:
    latency   =  163 us
    msg_size  =   32 KiB (32,768)
tcp_lat:
    latency   =  313 us
    msg_size  =   64 KiB (65,536)
```

至此，简单介绍了一下 **qperf** 这款网络性能测试工具的用法，后面有机会再看下其他性能测试工具，例如 **netperf**、**iperf** 等等，并比较各自的优缺点。
