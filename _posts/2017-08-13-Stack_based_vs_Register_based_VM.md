---
layout: post
title: Stack based vs. Register based VM
date: 2017-08-13
categories: 技术
tags: vm Stack-based Register-based
---

VM使“一处编译 到处运行"成为可能。

VM应该实现点啥?

- 把源语言编译成VM指定的字节码
- 表示指令和操作数(operands)的数据结构
- 函数调用堆栈
- 指令指针
- 一个虚拟 ‘CPU’ – 指令调度器
  - 获取下一条指令
  - 解码操作数
  - 实现指令

实现虚拟机有两种主要方式：基于堆栈的(Stack based)和基于寄存器的(Register based)。广泛使用的是的Stack based类型vm，如Java虚拟机。lua VM从5.0开始使用Register based模式，应该是第一个被业界大规模使用的后端语言Register based VM；Android VM Dalvik也是Register based类型的。两者区别在于存储、检索操作数及计算结果的机制。

## 1. Stack based

操作时存储在栈中，操作过程为 pop操作数→处理→push结果入栈

举个例子，Stack based VM是通过如下流程执行加法的：

```
POP 20
POP 7
ADD 20, 7, result
PUSH result
```

![stack_based](http://jinluzhang.github.io/assets/posts_img/2017_08_13_Stack_based_vs_Register_based_VM/Figure_1_stack_based.png)

因为需要push和pop，执行加法操作需要四行指令。

Stack based的一个优点是操作数由SP隐含寻址，VM不需要明确知道操作数地址。所有的算术运算和逻辑运算都是通过push和pop操作数及结果来进行的。

缺点是内存copy复制较多，指令较多，这些对效率都有影响。

## 2. Register based

操作数存储于CPU寄存器，所以不存在push和pop操作，但是指令需要包含操作数的寄存器地址。

仍以加法操作为例：

```
ADD R1, R2, R3 ;        # Add contents of R1 and R2, store result in R3
```

![register_based](http://jinluzhang.github.io/assets/posts_img/2017_08_13_Stack_based_vs_Register_based_VM/Figure_register_based.png)

由于没有push和pop操作，所以指令只有一条；但是我们需要额外指定操作数地址R1 R2 R3。

优点在于：

- 节省了入栈、弹栈开销，执行速度更快
- 使一些在Stack based VM中不能进行的优化成为可能：比如当有相同子表达式存在时，可以把计算结果暂存在寄存器中，节省了重复计算的开销

缺点在于平均指令长度变长了，字节码size比较大；在代码生成阶段对寄存器进行分配，实现复杂

## 3. Lua VM

lopcodes.h           /Users/zhangjinlu/env/origin_source/lua-5.3.0/src/lopcodes.h

 所有指令都是无符号整数，每条指令的前6位是opcode （共40种）

 指令格式有如下4种:

enum OpMode {iABC, iABx, iAsBx, iAx};

![OpMode](http://jinluzhang.github.io/assets/posts_img/2017_08_13_Stack_based_vs_Register_based_VM/Figure_3_lua_op.png)

 'A' : 8 bits

 'B' : 9 bits

 'C' : 9 bits

 'Ax' : 26 bits ('A', 'B', and 'C' together)

 'Bx' : 18 bits ('B' and 'C' together)

 'sBx' : signed Bx

它们用来存放 寄存器id、常量表id、upvalue id

![op-1](http://jinluzhang.github.io/assets/posts_img/2017_08_13_Stack_based_vs_Register_based_VM/Figure_4_a.png)

![op-2](http://jinluzhang.github.io/assets/posts_img/2017_08_13_Stack_based_vs_Register_based_VM/Figure_4_b.png)

 

## 相关资料

https://markfaction.wordpress.com/2012/07/15/stack-based-vs-register-based-virtual-machine-architecture-and-the-dalvik-vm/

http://opensourceforu.com/2011/06/virtual-machines-for-abstraction-dalvik-vm/

http://sunxiunan.com/archives/2151

http://simohayha.iteye.com/blog/517748