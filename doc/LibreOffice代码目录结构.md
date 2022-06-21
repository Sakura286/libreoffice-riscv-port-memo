
# LibreOffice 代码目录结构

**sal:**system abstract layer  包括rtl, osl 和sal. 读取LibreOffice系统配置文件的入口在sal/rtl/bootstrap.cxx，因为该文件会读取系统的一些信息，如ARCH、 OS等，可能在移植时从这里开始分析。

**salhelper**： a small C++ library which offers additional runtime library functionality

**cppu**： UNO core library

**cppuhelper:**C++ UNO Runtime Engine is implemented in here.
