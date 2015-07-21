---
layout: post
title: "汇编语言 第8、9章 定位与跳转"
description: "汇编语言笔记要点"
keywords: assembly, 汇编, 机器码, 8086
category: 汇编
tags: [汇编, 底层, 跳转]
---

## 第8章 数据处理的两个基本问题
即数据处理的地址和大小

### bx|bp 和 si|di
这4个寄存器都可以用于内存定位，不同点是：

bx默认是ds寄存器，即[bx] = ds:[bx]
bp默认是ss寄存器，即[bp] = ss:[bp]
他们只能二选一出现，即[bx+bp]错误

bx可以分割为字节而bp则不行

si|di也是二选一的。

### 处理数据长度
上述寄存器解决内存寻址问题，很简单，所以略过。

另一个问题是定位后，数据的长度。
基本思路是如果知道是byte还是word就没必要显式说明，例如： 
mov bx, ds:[0] bx是16位，所以是word
mov bl, ds:[0] bl是8位，所以是byte

如果无法判断是byte还是word则需要显示说明，例如：
mov **byte ptr** ds:[0],1
inc **word ptr** ds:[0]
需要加上关键词**byte|word ptr**显式说明。

还有些默认是访问字单元或字节单元也不需要指定，例如：
push [1000H] ,push只能用于字操作。

### div指令
div是除法指令，csapp中说过除法指令相对较慢，需要20多个时钟周期。

格式为：
div reg|内存单元

reg|内存单元 指的是除数

这里只介绍了整数除法，且将结果（包括余数和商）保存在特定位置：
（1）除数8位、被除数16位（放在AX中），结果保持：AL存放商、AH存放余数
（2）除数16位、被除数扩展为32位（DX放高位，AX放低位），结果保持：AX存放商，DX存放余数

猜测这样做的原因是工艺水平限制，8086上只有特定的寄存器实现除法功能。

### 伪指令dd
db (define byte)定义字节型数据，8位
dw (define word)定义字型数据，16位
dd (define double word)定义双字数据，32位

### dup
之前data segment中定义数据，一个一个地数据，简单粗放。
如果数据量很大，且有周期性规律其实有简化方法：
db 3 dup ('abc','ABC') 将定义'abcABCabcABCabcABC'共18个字节。
dw 200 dup (0) 则定义200个初始为0的字型数据区域。

## 第9章 转移指令的原理
修改IP或者同时修改CS、IP的指令统称为**转移指令**。

根据修改内容分类：

1. 只修改IP的转移，称为段内转移，例如 jmp ax，段内转移还分为
    * 短转移，IP修改范围-128 ~ 127
    * 近转移，IP修改范围-32768 ~ 32767
2. 修改CS和IP，称为段间转移。例如 jmp 1000:0

8086CPU的转移按照前提条件的不同分类：

* 无条件转移（如jmp）
* 条件转移
* 循环(如loop)
* 过程
* 中断

他们原理相似。

### 操作符 offset
如果在debug里jmp可以跳到指定的内存区域，
然而，如果是编译的文件，因为内存空间并未划分，
那么jmp根本找不到绝对的地址啊，

这时需要用偏移地址或者说相对地址了，
例如offset可以取得相对地址
jmp则跳到相对地址

#### offset用法举例
offset取得相对地址后，可以用于比如复制指令数据
例如如下，将s:处的`mov ax, bx`指令复制到s0：行。
由于该指令只有一个字，故只需复制一个字

```
assume cs:codesgm

codesgm segment
    s:  mov ax, bx
        ;用法为offset 标号
        mov si, offset s    ; s的偏移量为0
        mov di, offset s0   ; s0偏移量应该为2+3+3+3+3 = 14
        mov ax, cs:[si]
        mov cs:[di], ax
    s0: nop                 ;nop占用1个字节,s0偏移量应该为2+3+3+3+3 = 14
        nop
codesgm ends
end s
```


### jmp指令
jmp为无条件跳转，可以修改IP或者CS:IP

jmp要提供两个要素：
1. 转移的地址(标号|寄存器|内存单元地址)
2. 转移的距离(段间跳转|段内短跳转|段内近跳转)

jmp不同格式和用法：

```
;代码片段

    jmp short s;段内短跳转到 标号
    add ax,bx 
s:  inc ax
    ; jmp short s的机器指令为*偏移地址* 即：EB*03*,03即偏移地址
    ; 这是因为 当jmp short s进入指令缓冲起时,ip累加到02，准备执行add ax,bx
    ; 而jmp short s指令本身将IP又+3 = 05，故执行s:处的指令inc ax

    jmp short s;段内近跳转 机器码类似于EB 03 即跳转8位 -128~127

    jmp near ptr s  ;段内近跳转 机器码类似于E9 00 03 即跳转16位 -32768~32767

    jmp far ptr s   ;段间跳转 机器码类似于EA 0B01 BD0B
                    ;其中 0B 01表示IP 010BH，BD 0B 表示CS 0BBDH

    jmp ax ;jmp 寄存器，即IP = reg

    jmp word ptr ds:[0]     ; 执行后 IP = ds:[0]
    jmp dword ptr ds:[0]    ; 执行后 IP = ds:[0] CS = ds:[2]
```

然而，对于jmp+标号，实际上编译器会根据计算得到的跳转距离优化，
如果相对距离在-128~127之间，无论写成什么都会优化为 jmp short等等。
具体参见《汇编语言》附注3

### jcxz指令
为条件跳转指令，所有条件跳转指令都是短跳转 -128~127（至少对8086是这样）

格式： jcxz 标号（当cx = 0时，跳转到标号）
执行： (IP) = (IP) + 8位位移

即： 
if( (cx) = 0 ) jmp short 标号;

### loop指令
1. (cx) = (cx) -1
2. if ( (cx) != 0 ) then (IP) = (IP) + 8位位移

即：
cx--; 
if( (cx) != 0 ) jmp short 标号;


