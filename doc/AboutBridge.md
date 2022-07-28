
# UNO bridge 移植分析

[TOC]

抄作业的话，需要选择一个与 RISC-V 相近的架构，于是选择了 MIPS64

MIPS64 有如下几个文件

- call.hxx
- call.s
- cpp2uno.cxx
- uno2cpp.cxx
- except.cxx
- share.cxx

其实不需要考虑 uno 与 cpp 之间到底发生了什么，只需要理清楚调用关系即可

我们先把上面的文件分类：

- `call.hxx` `call.s` `cpp2uno.cxx` 为 cpp to uno 部分
- `uno2cpp.cxx` 为 uno to cpp 部分
- `except.cxx` `share.cxx` 为异常处理部分

接下来分别分析这三个部分

## 异常处理

大多数架构在 linux 环境下都是使用几乎相同的`except.cxx`与`share.cxx`，所以暂且认定这部分不需要动

## cpp to uno

这部分代码运行时可以分为两部分

### 生成 codeSnippet

（注意这一段里的单复数）外部的函数调用`initializeBlock()`将一段内存区域 block 初始化为 slots，而后继续调用`addLocalFunction()`遍历传入的`type`中的每一个成员。对每一个成员，调用`codeSnippet()`生成一段机器码插入到 slots 的结尾，并将对应的 slot 指向该段机器码。

所以`codeSnippet()`做了啥？生成了一段汇编指令（机器码），并将**指令**写入内存（这里只是生成，而并非运行这些指令）。**如果运行到这段指令**，会把`privateSnippetExecutor()` `cpp_vtable_call()` 的入口地址及一些其他的信息写入特定的寄存器，并跳转到`privateSnippetExecutor()`。

### cpp_vtable_call

由于之前的工作，程序会通过 codeSnippet 跳转到 `privateSnippetExecutor()` 里。

`privateSnippetExecutor()`在`cpp2uno.cxx`里有一行`extern "C"`的声明，但是其主体是在`call.s`里的。该函数把所有的函数参数寄存器(a0~a7,f12~f19)压栈，将参数放到函数参数寄存器(a0~a5)后，调用 `cpp_vatable_call()`，从中返回后再调整返回值寄存器（v0, v1, f0, f2），恢复栈指针并返回。

`cpp2uno.cxx`中的`cpp_vatable_call()`调用了`cpp2uno_call()`。

其他的函数大都是附着在这几个主干函数上的。

### 架构相关的代码及处理举措

1. codeSnippet 因为是机器码所以无法使用伪指令，只能考虑搭配 `lui` `ori` `slli` 将 64位的立即数（函数入口地址）写入寄存器。问题在于`lui` `ori`都是符号拓展的，所以不能把所有的数据位填满（第一位如果是 1 ，那么会非常麻烦）
2. call.s 的代码需要重写，指令与寄存器需要更换为 riscv64 的，但是还有其他需要注意的地方
   1. MIPS 有延迟指令槽，所以一些分支指令附近的代码需要调整顺序
   2. MIPS 的调用规范里，把函数参数寄存器从 4 个(a0~a3)调整到了 8 个(a0~a7)，多出来的 4 个是从临时寄存器里抠出来的，所以 MIPS的临时寄存器有 t0 ~ t3, t8, t9 ，注意寄存器的对应关系（**MARK**：我不知道为什么会这样）
   3. `.set noreorder`与`.set reorder`是 MIPS 专有的
   4. 里面有大量的调试信息，然而 RISC-V 不支持把变量设置为 .uleb128 类型，为了绕过这个槛，可以选择把除了 .cfi 开头的描述栈的变化的信息之外的所有调试信息都删掉，可以参照 AArch64 那边`vtableslotcall.s`的写法
3. `cpp2uno_call()`里有对寄存器数量有要求的变量`MAX_GP_REGS`与`MAX_FP_REGS`，这两个宏在`share.hxx`里定义，不过万幸的是 RISC-V 上这两者的值也是 8 ，所以不需要动

## uno to cpp

架构相关的代码有两处

1. `MAX_GP_REGS`与`MAX_FP_REGS`，这两个不用管
2. `callVirtualMethod()`里的内联汇编，内容与`privateSnippetExecutor()`相近，需要把对应的汇编指令与寄存器改为 RISC-V 的

就是这么简单，可我还是想知道为什么
