# 部分测试的修复及定位记录

## CppUnitTests

### testExternalRefFunctions

**错误表现**：在 ucalc_formula.cxx:7178~7181 中，在对 LibreOffice 所能支持的最大范围的单元格进行求和后，使用 GetError() 函数无法返回 538 (Formula::MatrixSize) 错误，而是返回 503 (#NUM!) 错误

**错因**：LibreOffice Calc 进行公式计算时，返回的错误值是使用一个 NaN 的 double 类型存储的，例如 0x7ff800000000021a ，错误值为 0x21a ，即 538 。该值包括在一个 KahanSum 类型中，在 get 时候需要过一遍 kahan 算法
在 sc/inc/kahan.hxx:59 时，x86 的特性使得两个 NaN 在进行相加的时候取后面一个的值，所以这个非法的 double 值正好保留下来了，而 rv 在 fadd.d 两个 NaN 值时会将结果置为无穷大（0x7ff8000000000000），返回的错误值随即丢失

附加资料
[1] [KahanSum 引入 interpr6.cxx](https://git.libreoffice.org/core/+/fd4df675dfe1012448285134082f61a0c03a7d15%5E%21/#F6)
