# Say Hello to x86_64 Assembly

对于汇编还是个不折不扣的新手来说，第一部分中出现的很多概念可能不是很明白，于是我决定写更多有价值的文章。所以，让我们开始《我的汇编学习之路》的第二部分。

## 术语和概念

当我写了第一篇之后，我从不同的读者那获得很多反馈，第一篇中有些部分不明白，这就是本文以及接下来几篇从一些术语的描述开始的原因。

**寄存器**（Register）—— 寄存器是处理器内小容量的存储结构，处理器的主要功能是数据处理，处理器可以从内存中获得数据，但这是一种低速的操作，这就是为什么处理器为什么要有自己数据存储结构，称为“寄存器”。

**小端**（Little-endian）—— 我们可以假设内存是一个大的数组，它包含一个字节一个字节的数。每个地址存储了内存“数组”中的一个元素，每个元素一个字节。举例来说，我们有 4 字节数：`AA 56 AB FF`，在小端模式下最低位存放在低地址上：

```
    0 FF
    1 AB
    2 56
    3 AA
```

这里，0、1、2、3 是内存地址。

**大端**（Big-endian）—— 大端存储数据与小端相反。所有上面的字节序在大端模式下是：

```
    0 AA
    1 56
    2 AB
    3 FF
```

**系统调用**（Syscall）—— 系统调用是用户程序要求操作系统为其完成某些工作的一种方式。你可以在[这里]()找到系统调用表。

**栈**（Stack）—— 处理器有非常有限个寄存器。所以栈是一块连续的内存空间，可以通过特殊寄存器如 RSP、SS、RIP 等来寻址。在接下来的文章我会专门深入介绍栈。

**段**（Section）—— 每个汇编程序都是由段来组成的，有以下的段：

* data —— 用来声明初始化的数据或常量
* bss —— 用来声明未初始化的变量
* text —— 用来存放代码

**通用寄存器**（General-purpose register）—— 有 16 个通用寄存器：rax、rbx、rcx、rdx、rbp、rsp、rsi、rdi、r8、r9、r10、r11、r12、r13、r14、r15。

当然这不是与汇编语言有关的全部的术语和概念，如果在接下来的文章中遇到奇怪的不熟悉的词汇，我们再来解释这些词的意思。

## 数据类型

基本的数据类型有：字节（bytes）、字（words）、双字（doublewords）、四字（duadwords）以及双四字（double dualwords），它们的长度分别为：8位、2个字节、4个字节、8个字节、16个字节（128位）。

现在我们只使用整数，所以只看看它的表示。整型有两种类型：无符号和有符号。无符号整型是一个字节、字、双字、四字表示的无符号二进制数，它们能表示的范围分别为：0～255、0～65,535、0～2^32-1、0～2^64-1。有符号整型是一个字节、字、双字、四字表示的有符号的二进制数。符号位在负数的时候是置位的，在正数和0的时候是清零的。整数能表示的范围是：1个字节 -128～127，1个字 -32,768~32,767，1个双字 -2^31~2^31-1，1个四字 -2^63~2^63-1。

## 段

正如我上面提到的，每个汇编程序都是由段来组成的，它包含数据段、代码段、bss 段。我们先来看看数据段，这是主要用来定义初始化的常量。例如：

```
section .data
    num1:   equ 100
    num2:   equ 50
    msg:    db "Sum is correct", 10
```

好了，这儿差不多清楚了，三个常量名字分别为 num1、num2 和 msg，值分别是 100、50 和 "Sum is correct",10 。但是 db 、equ 又是什么呢？实际上，NASM 支持大量伪指令：

* DB, DW, DD, DQ, DT, DO, DY 和 DZ —— 用来定义初始化数据的。例如：

```
;; Initialize 4 bytes 1h, 2h, 3h, 4h
db 0x01,0x02,0x03,0x04
 
;; Initialize word to 0x12 0x34
dw    0x1234       
```

* RESB, RESW, RESD, RESQ, REST, RESO, RESY, RESZ —— 用来定义非初始化变量
* INCBIN —— 包含外部二进制文件
* EQU —— 定义常量，例如：

```
;; now one is 1
one equ 1
```

* TIMES —— 重复指令或数据（下一篇文章中描述）



## 算术操作

下面是算术操作指令的简单列表：

* ADD —— 整数加
* SUB —— 减
* MUL —— 无符号乘
* IMUL —— 有符号乘
* DIV —— 无符号除
* IDIV —— 有符号除
* INC —— 自增
* DEC —— 自减
* NEG —— 取反

本文会用到一些，其它的在接下来的文章有所覆盖。

## 控制流

通常编程语言使用 `if`、`case`、`goto` 等等来改变程序的运行顺序，当然汇编也可以。这里我们提及到一些。有一个专门用来比较两个数大小的 `cmp` 指令，它被用来接着条件判断指令来决定是否跳转。例如：

```
;; compare rax with 50
cmp rax, 50
```

`cmp` 指令仅仅比较两个数，但是对它们的值没有影响，也不会根据比较的结果执行任何东西。为了在比较之后执行操作，有条件跳转指令，可以是下面的一个：

* JE —— 如果相等
* JZ —— 如果为零
* JNE —— 如果不相等
* JNZ —— 如果不为零
* JG —— 如果第一个操作数比第二个大
* JGE —— 如果第一个操作数比第二个大或者相等
* JA —— 与 JG 指令相同，只不过比较的是无符号数
* JAE —— 与 JGE 指令相同，只不过比较的是无符号数

例如如果我们想写 C 语言中类似于 if/else 的语句：

```
if (rax != 50) {
    exit();
} else {
    right();
}
```

在汇编中是这样的：

```
;; compare rax with 50
cmp rax, 50
;; perform .exit if rax is not equal 50
jne .exit
jmp .right
```
也有一种无条件跳转的指令语法：

```
JMP LABEL
```

例如：

```
_start:
    ;; ....
    ;; do something and jump to .exit label
    ;; ....
    jmp .exit
 
.exit:
    mov    rax, 60
    mov    rdi, 0
    syscall
```

这里 `_start` 标签后有一些代码，这些代码会被执行到，汇编最后控制转向到 `.exit` 标签处，该标签后的代码开始执行。

通常无条件跳转用在循环中，例如我们有 `label` 标签，它后面有一些代码，代码执行完之后进行条件判断，如果条件不成立将跳到该段代码的起始处。循环将在后面文章中介绍。

## 示例

我们看个简单的例子：两个数相加，得到它们的和，然后与预定义的一个数进行比较，如果相等输出一些东西到屏幕上；如果不等退出。下面是例子的源代码：

```
 ;initialised data section
section .data
    ; Define constants
    num1:   equ 100
    num2:   equ 50
    ; initialize message
    msg:    db "Sum is correct\n"
 
section .text
 
    global _start
 
;; entry point
_start:
    ; set num1's value to rax
    mov rax, num1
    ; set num2's value to rbx
    mov rbx, num2
    ; get sum of rax and rbx, and store it's value in rax
    add rax, rbx
    ; compare rax and 150
    cmp rax, 150
    ; go to .exit label if rax and 150 are not equal
    jne .exit
    ; go to .rightSum label if rax and 150 are equal
    jmp .rightSum
 
; Print message that sum is correct
.rightSum:
    ;; write syscall
    mov     rax, 1
    ;; file descritor, standard output
    mov     rdi, 1
    ;; message address
    mov     rsi, msg
    ;; length of message
    mov     rdx, 15
    ;; call write syscall
    syscall
    ; exit from program
    jmp .exit
 
; exit procedure
.exit:
    ; exit syscall
    mov    rax, 60
    ; exit code
    mov    rdi, 0
    ; call exit syscall
    syscall
```

我们过一下这段代码。首先在数据段定义了三个数：num1、num2 和值为 "Sum is correct\n" 的 msg。现在看到第 14 行，这是程序的入口的地方。我们将 num1 和 num2 的值放到通用寄存器 rax 和 rbx 中，使用 add 指令相加，在 add 指令执行完之后，rax 和 rbx 相加之和保存到 rax 中，即现在 num1 和 num2 的和存放在 rax 寄存器中。

好了，我们让 num1 是 100，num2 是 50，之和是 150，用 cmp 指令比较。在比较完 rax 和 150 之后，检查比较的结果，如果 rax 和 150 不等，我们跳转到 `.exit` 处，如果相等，跳到 `.rightSum` 标签处。

接着有两个标签：`.exit` 和 `.rightSum`。首先将 rax 设置为 60，这是 exit 系统调用号，以及将 rdi 设为 0，这是退出码。然后，`.rightSUm` 相当简单，只是打印出 `Sum is corret\n`，如果你不能理解怎么工作的，看看[第一篇文章]()。

## 结论

这是 **Say Hello to x86_64 Assembly** 系列文章的第二篇，如果你有任何问题或建议，给我留言。










