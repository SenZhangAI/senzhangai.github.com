---
layout: post
title: "exception handle CFI CFA"
description: "关于汇编中CFI(calling frame info)和CFA的作用"
keywords: 异常, CFI, CFA
category: 异常
tags: [异常, 汇编]
---

## 前言
在GAS（GNU Assembler）汇编代码中经常能看到关于cfi、cfa的伪码，例如：
补充：关于GAS语法与NASM语法区别参见[Linux 汇编器：对比 GAS 和 NASM](http://www.ibm.com/developerworks/cn/linux/l-gas-nasm.html)，写得很好

```asm
LFB0:
    .cfi_startproc
    pushl   %ebp
    .cfi_def_cfa_offset 8       # cfa偏移了8字节
    .cfi_offset 5, -8           # ebp(代号5) 向栈顶偏移了8字节
    movl    %esp, %ebp
    .cfi_def_cfa_register 5     # cfa_register不再是esp，而是ebp
    subl    $16, %esp
    movl    $0, -8(%ebp)
    movl    $0, -4(%ebp)
    jmp L2
L3:
    # 子程序执行代码，略
L2:
    cmpl    $15, -4(%ebp)
    jle L3
    movl    -8(%ebp), %eax
    # 子程序返回代码
    leave
    .cfi_restore 5              # ebp的值回复到初始状态
    .cfi_def_cfa 4, 4           # esp + 4
    ret
    .cfi_endproc
```

## CFI和CFA用途

这些有着cfi和cfa关键字的代码到底有什么用呢？
以下内容参考自：

<https://lambertyang.blog.ustc.edu.cn/>

CFI(Calling Frame Info)的作用是出现异常时stack的回滚unwind(unwind 英文有 放松，release的意思),而回滚的过程是一级级CFA往上回退，直到异常被catch。也就是说CFA定义为执行call ***的时候栈指针(Stack Pointer)所指向的地址。
eg：

```asm
pushl    %ebp
.cfi_def_cfa_offset 8   # cfa偏移了8字节
.cfi_offset 5, -8       # ebp(代号5) 向栈顶偏移了8字节
```

表示的是执行完`pushl %ebp`之后的SP与CFA偏移了8个字节长度。

```asm
movl    %esp, %ebp
.cfi_def_cfa_register 5     #cfa_register为ebp(代号5)
```

表示执行完`movl %esp，%ebp`后cfa_register不再是esp，而是ebp。

```asm
leave
.cfi_restore 5      # ebp返回
.cfi_def_cfa 4, 4   # esp + 4
```

表示执行完leave后ebp的值回复到初始状态，并CFA的计算方法是esp+4。

## CFI和CFA用途补充-stackoverflow

参见：[关于`.cfi_def_cfa_offset`的用途](http://stackoverflow.com/questions/7534420/gas-explanation-of-cfi-def-cfa-offset) 以及答案中提到的[DWARF Debugging Information Format文档第六章](http://dwarfstd.org/doc/DWARF4.pdf)

CFA(Canonical Frame Address)保存的是函数调用栈上的每个“call frame”的地址。
