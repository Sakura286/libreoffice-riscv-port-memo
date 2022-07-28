# 快速开始在 RISC-V 上踩坑

## 基本步骤

### 1. 准备好一个 riscv环境

如有需要在 Debian 上构建 qemu chroot 或 qemu system，请参照[基础环境的搭建]()的`快速建立一个debian riscv环境`一节

### 2. 下载源码

请参照[一些资料及链接]()的`源码`一节

目前稳定版为 [7.3.5.2](http://download.documentfoundation.org/libreoffice/src/7.3.5/) ， tarball 能下载到的最高版本是 [7.4.0.1](http://download.documentfoundation.org/libreoffice/src/7.4.0/) ，如要获取最新的版本，请克隆官方仓库

https://wiki.documentfoundation.org/Development/BuildingOnLinux)

### 3. 打patch

[基础的patch]()在主目录的 config 文件夹下

### 4. 安装依赖

请参照[Development/BuildingOnLinux - The Document Foundation Wiki](

### 5. 下载 external tarballs

请参照[一些资料及链接]()的`源码`一节，将 tarball 下载到

### 5. 开始编译

创建或修改主目录下的`./autogen.input`文件，添加参数

make！

## 其他

### 关于 autogen.input

- `./autogen.sh --help`可以查看编译打包用到的参数
- 建议使用 autogen.input 记录参数，这样运行 autogen.sh 时便不再需要指定参数了
- **修改**（不是新建） autogen.input 后直接 `make` 会自动运行 autogen.sh

### 在 Debian sid riscv 上编译的历史设置

#### 源代码

基于 master 分支的 [69485e422c9 提交](https://go.suokunlong.cn:88/cgit/lo/core/commit/?id=69485e422c9)

#### autogen.input 参数

```shell
--disable-firebird-sdbc
--with-galleries=no
--enable-debug
# 调整并行编译任务数
--with-parallelism=4
# external tarball 里的 orcus 会在当前（7.26）的提交中编译出错，
# 所以手动安装了 liborcus-dev ，加了这个选项
# --with-system-orcus
```

#### 依赖

```shell
apt-get install -y \
    build-essential zip ccache junit4 libkrb5-dev nasm graphviz python3 python3-dev \
    qtbase5-dev libkf5coreaddons-dev libkf5i18n-dev libkf5config-dev libkf5windowsystem-dev \
    libkf5kio-dev autoconf libcups2-dev libfontconfig1-dev gperf default-jdk doxygen \
    libxslt1-dev xsltproc libxml2-utils libxrandr-dev libx11-dev libxt-dev libassuan-dev \
    bison flex libgtk-3-dev libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev \
    ant ant-optional libnss3-dev libavahi-client-dev automake tar xz-utils unzip
```

