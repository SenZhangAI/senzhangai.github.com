---
layout: post
title: "汇编语言 第10章 CALL和RET指令"
description: "汇编语言笔记要点"
keywords: assembly, 汇编, 机器码, 8086
category: 汇编
tags: [汇编, 底层, 中断, call和return]
---

call和ret都是跳转指令，修改ip或者cs:ip

### ret和retf
ret是return的缩写
ret是return far

ret指令有2步骤：
1. (IP) = ( (ss)*16 + (sp) ) 
2. (sp) = (sp) + 2
即 pop IP

retf执行4步骤：
1. (IP) = ( (ss)*16 + (sp) ) 
2. (sp) = (sp) + 2
3. (CS) = ( (ss)*16 + (sp) ) 
4. (sp) = (sp) + 2
即：
pop IP
pop CS

## call指令
相当于：
1.push cs ; 如果的段内跳转，则不需要push cs
2.push ip
3.转移

call不能实现short转移，其他则与jmp原理相同

### call 标号
（IP 压入栈，转到标号处执行指令）

1. (sp) = (sp) - 2
2. ( (ss)*16 + (sp) ) = (IP)
3. (IP) = (IP) + 16位位移

即：
push IP
jmp near ptr 标号

该16位位移依然是相对地址偏移量

### call far ptr 标号
即：
push CS
push IP
jmp far ptr 标号

### call 寄存器
对于 call 16位reg，相当于：
push IP
jmp 16位reg

### call 内存地址

call word ptr 内存单元地址 相当于：
push IP
jmp word ptr 内存单元地址

call dword ptr 内存单元地址 相当于：
push CS
push IP
jmp dword ptr 内存单元地址

注意jmp dword ptr的内存赋值顺序，先给IP，后地址+2给CS

### call和ret配合
通过call和ret配合，实现子程序，具体形式如下

```
assume cs:code
code segment
    main:   nop
            nop
            call sub1
            nop
            mov ax, 4c00h
            int 21h

    subl:   nop
            nop
            call sub2
            nop
            ret

    sub2:   nop
            nop
            ret
code ends
end
```

可见，子程序通过call跳转实现，再由ret跳转返回，继续执行下一条指令

### mul指令
乘法指令，与本主题无关，只是书中例子用到，

该指令与除法div类似，也是用到AL或者AX为默认寄存器

其中8位乘法
例如： mul bl
意思是 al * bl

16位乘法
例如： mul byte ptr [0]
意思是 ax * ds:[0]

32位乘法
例如： mul word ptr [0]

结果8位则放在al中
结果16位放在ax中
结果32位则高16位放在dx中，低16位放在ax中

### 模块化程序设计
#### 参数和结果传递的问题
解决方案：用**寄存器**保存子程序的参数和结果是非常常见的方法

#### 批量数据传递
如果参数的个数很多，则寄存器不够用，
解决方案是：批量将参数放到内存中，然后将内存空间首地址给寄存器
（如果参数变量占用空间不一样就不好咯，书中没给这种情况的方案）

批量返回结果也可用同样方法

#### 用栈传递参数
除了寄存器传递参数，还可用**栈**来传参，也包括函数地址IP等

这项技术与高级语言编译器工作原理密切相关

这在CSAPP中的AT&T风格的汇编中多次出现，

简单来说，就是想参数和IP一起压入栈，先压入参数，后压入IP
那么读取参数将会是：
mov bx, sp
mov ax, [bx + 4]
mov bx, [bx + 6]
类似的语句

通常的顺序会是

1. 按照最后的参数先进栈（栈底）的顺序压栈
2. call 子函数，那么地址（IP）进栈
3. 子函数使用的寄存器入栈
4. 子程序pop 给之前使用的寄存器
5. sp指针指向正确的空间 （或者说把子函数的参数数据都弹出）

**注意栈内位置和压栈顺序**

#### 寄存器冲突的问题
例如main主程序中用cx作为loop次数
而恰好子程序中用到cx，则肯定会发生错误，
因此合理的解决办法是

```
子程序初始：将使用的寄存器结果先入栈

子程序主体：

子程序返回：pop，将使用过的寄存器原数据返回
            ret
```

## jmp与call、ret的补充-为什么分长跳转及短跳转
汇编中实现指令顺序变化的方式有jmp、call和ret。分为长条转和短跳转。
即便是现代CPU指令也这样划分，更确切的说划分为段内跳转和段间跳转。
也是改变IP(段内)或者CS:IP(段间)
不过可能已经把段内短跳转和段内近跳转归为一类了。
现代CPU指令为什么也这样划分，有何好处呢？

至少有一点好处是这样方便了操作系统作特权级检查。当在同一代码段内作跳转时（CS不变）那么可以假设相关代码都是同一权限。而当在长条转时改变CS，跳到不同代码段，这时候才作特权级检查。

