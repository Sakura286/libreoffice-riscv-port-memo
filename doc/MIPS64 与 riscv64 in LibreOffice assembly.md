
# MIPS64 与 RISCV64 in LibreOffice assembly

[mips64 assembly online ] [https://edumips64.readthedocs.io/en/latest/instructions.html](https://edumips64.readthedocs.io/en/latest/instructions.html)

## .set noreorder/reorder

默认汇编器处在reorder的模式下，该模式允许汇编器对指令进行重新排序，以避免流水线堵塞并获得更好的性能，在这种模式下，是不允许在代码中插入 nop指令的。反之，在noreorder模式下，指令的顺序不会被改变也不会对代码进行任何优化。这样做的优点是程序员可以完全控制代码的执行顺序，缺点是必须手工对指令排序，并在分支和加载指令的延迟槽中填上有用的指令或nop指令．比如：

.set noreorder

lw   t0, 0(a0)

nop        #加载指令延迟槽

sub  t0, 1

bne  t0, zero, loop

nop        #分支指令延迟槽

.set reorder

原文链接：[https://blog.csdn.net/adaptiver/article/details/6760220](https://blog.csdn.net/adaptiver/article/details/6760220)

## DADDIU rt, rs, immediate

Executes the sum between 64-bits register rs and the immediate value, putting the result in rt. No integer overflow occurs under any circumstances.

对应ADDW(riscv64)?

## SD rt, offset(base)

Stores the content of register rt in the memory cell specified by offset and base, treating it as a double word.    

sd	$ra,152($sp) （riscv  也是 sd存储double word）

1. sdc1	$f12,80($sp) 
 Store double-word To Float Point, Save the float point registers.

(fsd riscv)

## move	$a0,$v0 [$v0是return value, $a0是函数参数 ]
##  b: Unconditional PC-relative (though relatively short-range) branch.

## 小概念

#### 汇编器指令（assembler directive）

就是以.开头的那些指令，既有架构特定的，也有GNU特定的

```plain
RISC-V
https://github.com/riscv-non-isa/riscv-asm-manual/blob/master/riscv-asm.md#:~:text=directive%20is%20used.-,Pseudo%20Ops,-Both%20the%20RISC
MIPS
https://www.d.umn.edu/~gshute/mips/directives.html
GNU汇编（gas）
https://sourceware.org/binutils/docs-2.38/
```

需要解释的汇编器指令

### 伪指令（pseudo instruction）

指的是为了简化书写而使用的“不正规”的汇编指令，例如mv实际上是addi加0的操作

国内有把汇编器指令称呼为“伪指令”的，但是在查阅英文时无法复现