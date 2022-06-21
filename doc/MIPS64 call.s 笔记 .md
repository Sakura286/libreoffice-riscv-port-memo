
# MIPS64 call.s 笔记

```plain
/*
 * This file is part of the LibreOffice project.
 *
 * This Source Code Form is subject to the terms of the Mozilla Public
 * License, v. 2.0. If a copy of the MPL was not distributed with this
 * file, You can obtain one at http://mozilla.org/MPL/2.0/.
 *
 * This file incorporates work covered by the following license notice:
 *
 *   Licensed to the Apache Software Foundation (ASF) under one or more
 *   contributor license agreements. See the NOTICE file distributed
 *   with this work for additional information regarding copyright
 *   ownership. The ASF licenses this file to you under the Apache
 *   License, Version 2.0 (the "License"); you may not use this file
 *   except in compliance with the License. You may obtain a copy of
 *   the License at http://www.apache.org/licenses/LICENSE-2.0 .
 */
        // 存放实际代码
        .text

        // 使privateSnippetExecutor标号全局可见
        // "全局可见"的意思是可以从文件外部引用，例如cpp2uno.cxx多次引用了这个方法
        .globl  privateSnippetExecutor

// 参照https://stackoverflow.com/questions/15284947/understanding-gcc-s-output
// LFB0等只是在抛出异常后调试时会使用的一些标签
// 附：看起来如果程序里面没有写条件分支的话，那么这些标签看起来可以忽略不计
.LFB0 = .

        // CFI: Control-Flow Integrity(控制流完整性) ×
        // CFI: Call Frame Information
        // 翻译：.cfi_startproc用在每个函数的开头，这个程序应该在.eh_frame里有入口
        // 需要使用.cfi_endproc来关闭函数
        .cfi_startproc

        .cfi_personality 0x80,DW.ref.__gxx_personality_v0

        .cfi_lsda 0,.LLSDA0

        // gas手册上关于ent指令的用法只提到了SCORE、MicroBlaze、Alpha架构会用.ent
        // MIPS 64也用吗？
        // https://stackoverflow.com/questions/33062405/why-do-we-use-globl-main-in-mips-assembly-language
        // 设置.ent是在设置调试器的入口
        .ent    privateSnippetExecutor

        // 设置一个符号的类型，比如这里把priva...设置成了函数名
        // 一些语法：
        //.type <name> STT_<TYPE_IN_UPPER_CASE>
        //.type <name>,#<type>
        //.type <name>,@<type>
        //.type <name>,%<type>
        //.type <name>,"<type>"
        .type   privateSnippetExecutor, @function

privateSnippetExecutor:
        // noreorder是MIPS64特定的代码
        // https://training.mips.com/basic_mips/PDF/Assemble_Language.pdf
        .set    noreorder
        
        // daddiu 把一个寄存器的内容加上一个无符号立即数
        // 为什么是无符号数？可是这里也-160了啊
        // MIPS里栈是由高地址向低地址增长（向下增长）的，所以是-160
        // 也就是说，开辟了一个160字节的栈
        // +----------------+
        // |  last program  | 
        // +----------------+ <---- old $sp
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // +----------------+ <---- new $sp= $sp - 160
        daddiu  $sp,$sp,-160

        // CFI : Call Frame Information
        // https://garlicspace.com/2019/07/10/%E6%B1%87%E7%BC%96%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84cfi%E6%8C%87%E4%BB%A4/
        // https://www.imperialviolet.org/2017/01/18/cfi.html
        // rip 指令地址寄存器，CPU将要执行的下一条指令的地址
        // rbp 栈基地址寄存器，存放当前帧的栈底
        // rsp 栈指针寄存器，存放当前栈顶

        // CFA : Call Frame Address 标准帧地址
        // 为每个过程分配的内存区域叫做stack frame（栈帧）
        // https://stackoverflow.com/questions/7534420/gas-explanation-of-cfi-def-cfa-offset
        // https://stackoverflow.com/questions/29527623/in-assembly-code-how-cfi-directive-works
        // 在调用新程序之时应该设置CFA指向调用者执行call时的栈指针地址，也就是现在的$sp+160的位置
        // 为了确保新程序出错时能溯源到出错位置，CFA的位置在这次调用中不变，并且使用CFA来记录各个寄存器的变动记录
        // +----------------+
        // |  last program  |
        // +----------------+ <---- CFA
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // |                | 
        // +----------------+ <---- $sp
        .cfi_def_cfa_offset 160

        // sd : save double words
        // 存双字，一个字是4字节(32bit)，双字也就是8字节(64bit)，160-152=8
        // 这里在【原先】栈顶的顶部压入了返回地址，现在是这样
        // +----------------+
        // |  last program  | 
        // +----------------+ <---- CFA
        // |  return adress |
        // +----------------+ <---- sd  $ra,152($sp)
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // |                |
        // +----------------+ <---- $sp
        sd      $ra,152($sp)

        // 语法：.cfi_offset register, offset
        // 把寄存器里的值存在CFA的偏移位置处
        // 31号寄存器是$ra，存在了CFA偏移-8的位置处
        .cfi_offset 31, -8
.LEHB0 = .
        // Save the float point registers
        // sdc : Store frame pointer double
        // 也就是说要把寄存器中的数据存到内存中对应位置
        sdc1    $f12,80($sp)
        sdc1    $f13,88($sp)
        sdc1    $f14,96($sp)
        sdc1    $f15,104($sp)
        sdc1    $f16,112($sp)
        sdc1    $f17,120($sp)
        sdc1    $f18,128($sp)
        sdc1    $f19,136($sp)
        // Save the general purpose registers
        sd      $a0,16($sp)
        sd      $a1,24($sp)
        sd      $a2,32($sp)
        sd      $a3,40($sp)
        sd      $a4,48($sp)
        sd      $a5,56($sp)
        sd      $a6,64($sp)
        sd      $a7,72($sp)
        // Load arguments
        // a0=index
        // 我这里不太明白$v0和$v1之前有什么值
        move    $a0,$v0
        // a1=offset
        move    $a1,$v1
        // a2=gpregptr
        daddiu  $a2,$sp,16
        // a3=fpregptr
        daddiu  $a3,$sp,80
        // a4=ovrflw
        daddiu  $a4,$sp,160
        // Call cpp_vtable_call
        // jalr oprd1 oprd2
        // 无条件跳转到由oprd1存储的地址，并将下一条指令的地址保存到寄存器oprd2(默认是31)中
        // 所以这跳哪儿去了？怎么回来？
        jalr    $t9
        // a5=retregptr
        move    $a5,$sp

.LEHE0 = .
        // Perform return value
        // li: load immediate
        li      $v1,10
        beq     $v0,$v1,.Lfloat
        li      $v1,11
        beq     $v0,$v1,.Lfloat
        ldc1    $f0,0($sp)
        ldc1    $f2,8($sp)
        ld      $v0,0($sp)
        b       .Lfinish
        ld      $v1,8($sp)
.Lfloat:
        ldc1    $f0,0($sp)
        ldc1    $f2,8($sp)

.Lfinish:
        ld      $ra,152($sp)
        .cfi_restore 31
        jr      $ra
        daddiu  $sp,$sp,160
        .cfi_def_cfa_offset 0

        .set    reorder
        .end    privateSnippetExecutor
        // 函数到这里结束了
        .cfi_endproc
.LFE0:
        .globl  __gxx_personality_v0
        .section        .gcc_except_table,"aw",@progbits
        .align  3
.LLSDA0:
        .byte   0xff
        .byte   0x80
        .uleb128 .LLSDATT0-.LLSDATTD0
.LLSDATTD0:
        .byte   0x1
        .uleb128 .LLSDACSE0-.LLSDACSB0
.LLSDACSB0:
        .uleb128 .LEHB0-.LFB0
        .uleb128 .LEHE0-.LEHB0
        .uleb128 0
        .uleb128 0
.LLSDACSE0:
        .byte   0x7f
        .byte   0
        .align  3
        .8byte  DW.ref._ZTIi
.LLSDATT0:
        .byte   0x1
        .byte   0
        .text
        .size   privateSnippetExecutor, .-privateSnippetExecutor
        .hidden DW.ref._ZTIi
        .weak   DW.ref._ZTIi
        .section        .data.DW.ref._ZTIi,"awG",@progbits,DW.ref._ZTIi,comdat
        .align  3
        .type   DW.ref._ZTIi, @object
        .size   DW.ref._ZTIi, 8
DW.ref._ZTIi:
        .dword  _ZTIi
        .hidden DW.ref.__gxx_personality_v0
        .weak   DW.ref.__gxx_personality_v0
        .section        .data.DW.ref.__gxx_personality_v0,"awG",@progbits,DW.ref.__gxx_personality_v0,comdat
        .align  3
        .type   DW.ref.__gxx_personality_v0, @object
        .size   DW.ref.__gxx_personality_v0, 8
DW.ref.__gxx_personality_v0:
        .dword  __gxx_personality_v0
```
