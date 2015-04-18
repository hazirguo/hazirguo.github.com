---
layout: post
title: "最小 Docker 镜像 hello-world 剖析"
description: ""
category: docker 
tags: [docker, dockerfile]
---
{% include JB/setup %} 

开始学习 Docker 的同学基本上都是按照官方的 guide 来安装，之后要测试是否已经安装成功，官方会让你 pull 一个 hello-world 示例镜像下来并运行，如下命令：

```
 guohl@ghl-MBP ⮀ ~ ⮀ docker pull hello-world
31cbccb51277: Pull complete
e45a5af57b00: Pull complete
511136ea3c5a: Already exists
hello-world:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Status: Downloaded newer image for hello-world:latest

guohl@ghl-MBP ⮀ ~ ⮀ docker run hello-world
Hello from Docker.
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (Assuming it was not already locally available.)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

For more examples and ideas, visit:
 http://docs.docker.com/userguide/
```

出现上面的输出信息表示你 Docker 安装成功。使用下面命令查看该镜像的信息，发现大小只有 910B：

```
guohl@ghl-MBP ⮀ ~ ⮀ docker images hello-world
REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
hello-world         latest              e45a5af57b00        3 months ago        910 B
```

那么该镜像运行怎么就输出上面这一串信息的呢？如果对 Docker 了解一点的话自然会去找该镜像的 Dockerfile，Dockerfile 详细描述了该镜像是如何建立的以及运行时执行的命令。从 Docker Hub 上 hello-world 的库[简介](https://registry.hub.docker.com/_/hello-world/) 找到其 [Dockerfile](https://github.com/docker-library/hello-world/blob/b7a78b7ccca62cc478919b101f3ab1334899df2b/Dockerfile) 在 github 上，简简单单的三条语句：

```
FROM scratch
ADD hello /
CMD ["/hello"]
```

* 第一条`FROM`指令是 Docker 用来指定该镜像是基于哪个基础镜像构建的，这里指定为 [`scratch`](https://registry.hub.docker.com/_/scratch/)，实际上 `scratch` 镜像是一个空镜像，用来构建基础镜像或者极小的镜像。
* 第二条`ADD`指令表示从 Dockerfile 所在目录拷贝文件到指定路径下，这里拷贝 `hello` 文件到根目录下，至于 `hello` 文件是什么稍后介绍。
* 第三条`CMD`指令用来指示当运行 `docker run` 命令运行该镜像时要执行的命令，这里执行 `/hello`，就是执行第二步拷贝到根目录下的 `hello` 文件。

`hello` 是一个可执行的二进制文件，从文章的开头可以推测该文件执行输出那一堆提示信息，那该文件怎样生成的呢？从 [hello-world 的库](https://github.com/docker-library/hello-world/tree/b7a78b7ccca62cc478919b101f3ab1334899df2b)中可以看到还有两个文件，一个是汇编文件 `hello.asm`，还有一个 `Makefile` 文件，可以猜测 `hello` 就是事先通过 `hello.asm` 编译得到的，如 `Makefile` 文件所描述的编译过程：

```
hello: hello.asm
	nasm -o $@ $<
	chmod +x hello

.PHONY: clean
clean:
	-rm -vf hello
```

该 `Makefile` 文件比较容易理解，主要是使用 nasm 汇编器将 `hello.asm` 编译生成可执行文件 `hello`。

最后再来看看 `hello.asm` 文件，这是一段 Intel x86 汇编：

```
; this is especially thanks to:
; http://blog.markloiseau.com/2012/05/tiny-64-bit-elf-executables/

BITS 64
	org	0x00400000	; Program load offset

; 64-bit ELF header
ehdr:
	;  1), 0 (ABI ver.)
	db 0x7F, "ELF", 2, 1, 1, 0       ; e_ident
	times 8 db 0                     ; reserved (zeroes)

	dw 2              ; e_type:	Executable file
	dw 0x3e           ; e_machine:	AMD64
	dd 1              ; e_version:	current version
	dq _start         ; e_entry:	program entry address (0x78)
	dq phdr - $$      ; e_phoff	program header offset (0x40)
	dq 0              ; e_shoff	no section headers
	dd 0              ; e_flags	no flags
	dw ehdrsize       ; e_ehsize:	ELF header size (0x40)
	dw phdrsize       ; e_phentsize:	program header size (0x38)
	dw 1              ; e_phnum:	one program header
	dw 0              ; e_shentsize
	dw 0              ; e_shnum
	dw 0              ; e_shstrndx

ehdrsize equ $ - ehdr

; 64-bit ELF program header
phdr:
	dd 1              ; p_type:	loadable segment
	dd 5              ; p_flags	read and execute
	dq 0              ; p_offset
	dq $$             ; p_vaddr:	start of the current section
	dq $$             ; p_paddr:	"		"
	dq filesize       ; p_filesz
	dq filesize       ; p_memsz
	dq 0x200000       ; p_align:	2^11=200000 = section alignment

; program header size
phdrsize equ $ - phdr

; Hello World!/your program here
_start:

	; sys_write(stdout, message, length)
	mov	rax, 1           ; sys_write
	mov	rdi, 1           ; stdout
	mov	rsi, message     ; message address
	mov	rdx, length      ; message string length
	syscall

	; sys_exit(return_code)
	mov	rax, 60          ; sys_exit
	mov	rdi, 0           ; return 0 (success)
	syscall

	message:
		db 'Hello from Docker.', 0x0A
		db 'This message shows that your installation appears to be working correctly.', 0x0A
		db 0x0A
		db 'To generate this message, Docker took the following steps:', 0x0A
		db ' 1. The Docker client contacted the Docker daemon.', 0x0A
		db ' 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.', 0x0A
		db '    (Assuming it was not already locally available.)', 0x0A
		db ' 3. The Docker daemon created a new container from that image which runs the', 0x0A
		db '    executable that produces the output you are currently reading.', 0x0A
		db ' 4. The Docker daemon streamed that output to the Docker client, which sent it', 0x0A
		db '    to your terminal.', 0x0A
		db 0x0A
		db 'To try something more ambitious, you can run an Ubuntu container with:', 0x0A
		db ' $ docker run -it ubuntu bash', 0x0A
		db 0x0A
		db 'For more examples and ideas, visit:', 0x0A
		db ' http://docs.docker.com/userguide/', 0x0A
	length: equ	$-message            ; message length calculation

; File size calculation
filesize equ $ - $$
```

也比较简单，就调用了两个系统调用，`sys_write` 向标准输出打印一段信息，`sys_exit` 退出程序。

至此，我们分析完了官方提供的 `hello-world` 镜像整个构建及运行过程，麻雀虽小，五脏俱全，理解最小的 Docker 镜像的工作机制对我们理解 Docker 有很大帮助。

