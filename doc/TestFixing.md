# 部分测试的修复及定位记录

## CppUnitTests

### testExternalRefFunctions

**错误表现**：在 ucalc_formula.cxx:7178~7181 中，在对 LibreOffice 所能支持的最大范围的单元格进行求和后，使用 GetError() 函数无法返回 538 (Formula::MatrixSize) 错误，而是返回 503 (#NUM!) 错误

**错因**：LibreOffice Calc 进行公式计算时，返回的错误值是使用一个 NaN 的 double 类型存储的，例如 0x7ff800000000021a ，错误值为 0x21a ，即 538 。该值包括在一个 KahanSum 类型中，在 get 时候需要过一遍 kahan 算法
在 sc/inc/kahan.hxx:59 时，x86 的特性使得两个 NaN 在进行相加的时候取后面一个的值，所以这个非法的 double 值正好保留下来了，而 rv 在 fadd.d 两个 NaN 值时会将结果置为无穷大（0x7ff8000000000000），返回的错误值随即丢失

**补充**：这种通过 NaN 浮点数未使用的部分来传递(propagation)有效信息(payload)的方式叫做 NaN Boxing (NaN Propagation)，目前 x86、arm、mips、PowerPC、S390x等架构均支持，而 RISC-V 没有做相关的支持。困った。

#### 附加资料

[1] [KahanSum 引入 interpr6.cxx](https://git.libreoffice.org/core/+/fd4df675dfe1012448285134082f61a0c03a7d15%5E%21/#F6)
[2] [IEEE-754 2008, Ch. 6.2.3 NaN propagation](http://www.dsc.ufcg.edu.br/~cnum/modulos/Modulo2/IEEE754_2008.pdf)
[3] [RISC-V Spec, Ch.8.3 NaN Generation and Propagation](https://riscv.org/wp-content/uploads/2017/05/riscv-spec-v2.2.pdf)
[4] [GetErrCode() doesn't work as expected on RISC-V due to floating-point NaN payload not being propagated - Bugzilla](https://bugs.documentfoundation.org/show_bug.cgi?id=152943)
[5] [RISC-V's canonical NaN](https://groups.google.com/a/groups.riscv.org/g/isa-dev/c/g79dHlV4B_k)
[6] [NaN payload propagation - unresolved issues](https://grouper.ieee.org/groups/msc/ANSI_IEEE-Std-754-2019/background/nan-propagation.pdf)
