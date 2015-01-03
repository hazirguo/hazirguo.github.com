---
layout: post
title: "写自己的第一个 Linux 内核模块"
description: ""
category: linux 
tags: [linux, kernel, module]
---
{% include JB/setup %}


什么是内核模块？内核模块是一段可以在内核需要的时候加载或者删除的代码。大多数驱动以内核模块的形式提供，当这些驱动不需要的时候，我们可以去掉这些特定的驱动，这会有效地减小内核镜像的大小。

内核模块以 .ko 为扩展名，在大多 Linux 系统中，内核模块位于 `/lib/modules/<kernel_version>/kernel/directory` 目录下。

本文将用一个最简单的 Hello World 实例来介绍如何写一个内核模块。


## 操作内核模块的工具
### 1. lsmod -- 列出所有已经加载的模块
**`lsmod`** 命令用来列出已经加载到内核中的模块，下面显示的是部分信息：

```
# lsmod
Module                  Size  Used by
ppp_deflate            12806  0 
zlib_deflate           26445  1 ppp_deflate
bsd_comp               12785  0 
..
```
  
### 2. insmod -- 插入模块到内核
**`insmod`** 命令用来插入一个新的模块到内核中：

```
# insmod /lib/modules/3.5.0-19-generic/kernel/fs/squashfs/squashfs.ko

# lsmod | grep "squash"
squashfs               35834  0
```

### 3. modinfo -- 显示模块的信息
**`modinfo`** 命令用来显示内核模块的信息：

```
# modinfo /lib/modules/3.5.0-19-generic/kernel/fs/squashfs/squashfs.ko

filename:       /lib/modules/3.5.0-19-generic/kernel/fs/squashfs/squashfs.ko
license:        GPL
author:         Phillip Lougher 
description:    squashfs 4.0, a compressed read-only filesystem
srcversion:     89B46A0667BD5F2494C4C72
depends:        
intree:         Y
vermagic:       3.5.0-19-generic SMP mod_unload modversions 686
```
### 4. rmmod -- 从内核中删除模块
**`rmmod`** 命令从内核中移除模块，你可以移除已经加载到内核中的模块：

```
# rmmod squashfs.ko
```

### 5. modprobe -- 内核增加或删除模块
**`modprobe`** 命令可以根据模块之间的依赖关系，自动选择插入或删除其它依赖的模块。

## 写一个简单的 Hello World 内核模块

### 1. 安装 Linux headers
首先你需要安装 `linux-headers-...`，根据你的发行版选择使用 apt-get 或者 yum:

```
# apt-get install build-essential linux-headers-$(uname -r)
```

### 2. Hello World 模块源代码
接下来用 C 语言创建模块的源代码 hello.c：

``` c
#include <linux/module.h>    // included for all kernel modules
#include <linux/kernel.h>    // included for KERN_INFO
#include <linux/init.h>      // included for __init and __exit macros

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Lakshmanan");
MODULE_DESCRIPTION("A Simple Hello World module");

static int __init hello_init(void)
{
    printk(KERN_INFO "Hello world!\n");
    return 0;    // Non-zero return means that the module couldn't be loaded.
}

static void __exit hello_cleanup(void)
{
    printk(KERN_INFO "Cleaning up module.\n");
}

module_init(hello_init);
module_exit(hello_cleanup);
```

**警告：** 所有内核模块将会在高特权级别的内核空间操作，所以写内核模块时你需要特别小心！

### 3. 创建编译内核模块的 Makefile
下面的 makefile 文件可以用来编译成 hello world 内核模块：

```
obj-m += hello.o

all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```
使用 make 命令来编译模块：

```
# make

make -C /lib/modules/3.5.0-19-generic/build M=/home/lakshmanan/a modules
make[1]: Entering directory `/usr/src/linux-headers-3.5.0-19-generic'
  CC [M]  /home/lakshmanan/a/hello.o
  Building modules, stage 2.
  MODPOST 1 modules
  CC      /home/lakshmanan/a/hello.mod.o
  LD [M]  /home/lakshmanan/a/hello.ko
make[1]: Leaving directory `/usr/src/linux-headers-3.5.0-19-generic'
```
上面命令会得到 hello.ko 文件，就是我们的实例内核模块。

### 4. 插入或删除示例内核模块
现在我们有了 hello.ko 文件，我们可以使用 insmod 命令将该模块插入到内核：

```
# insmod hello.ko

# dmesg | tail -1
[ 8394.731865] Hello world!

# rmmod hello.ko

# dmesg | tail -1
[ 8707.989819] Cleaning up module.
```

当模块被加载到内核中时，module_init 宏被触发，将会调用 hello_init 函数。类似的，当使用 rmmod 命令来移除模块时， module_exit 宏被触发，调用 hello_exit 函数。使用 dmesg 命令，可以查看示例内核模块的输出。

注意 printk 是定义在内核中的函数，它的用法与 IO 库函数 printf 相似，但在内核中不能使用任何库函数。

好了，你现在已经学会如何创建基本的内核模块了。


---
编译自：[http://www.thegeekstuff.com/2010/11/modprobe-command-examples/](http://www.thegeekstuff.com/2010/11/modprobe-command-examples/)
