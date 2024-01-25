
# libreoffice-riscv-port-memo

Some memo taken when strive for porting LiberOffice to RISC-V

- 快速开始 -> [快速开始在 RISC-V 上踩坑](/doc/QuickStart_CN.md)
- 常用链接 -> [一些资料及链接](/doc/Reference.md)
- 新鲜资料 -> [Scaffolding](/doc/Scaffolding.md)
- 获取补丁 -> [01-libreoffice-riscv64.patch](/config/01-libreoffice-riscv64.patch)

更多内容，请查看 doc 及 config 文件夹

## ATTENTION!

Currently `-gsplit-dwarf` of gcc-riscv64 is broken, which will cause 'no symbol xxx in current context' when using gdb to debug. Please add `--disable-split-debug` to `autogen.input` to avoid this.
