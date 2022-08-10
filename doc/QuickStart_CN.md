# 快速开始在 RISC-V 上踩坑

[TOC]

## 太长不看版

参照[Development/BuildingOnLinux - The Document Foundation](https://wiki.documentfoundation.org/Development/BuildingOnLinux)

```shell
# 安装依赖
apt-get install -y \
    build-essential zip ccache junit4 libkrb5-dev nasm graphviz python3 python3-dev \
    qtbase5-dev libkf5coreaddons-dev libkf5i18n-dev libkf5config-dev libkf5windowsystem-dev \
    libkf5kio-dev autoconf libcups2-dev libfontconfig1-dev gperf default-jdk doxygen \
    libxslt1-dev xsltproc libxml2-utils libxrandr-dev libx11-dev libxt-dev libassuan-dev \
    bison flex libgtk-3-dev libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev \
    ant ant-optional libnss3-dev libavahi-client-dev automake git wget

# 国外源 git clone https://gerrit.libreoffice.org/core libreoffice
git clone --depth=1 git://go.suokunlong.cn/lo/core libreoffice
cd libreoffice

wget https://raw.githubusercontent.com/Sakura286/libreoffice-riscv-port-memo/main/config/01-libreoffice-riscv64.patch
patch -p1 <01-libreoffice-riscv64.patch

# 如果下载 external tarballs 的速度太慢，可以考虑使用国内缓存
# wget -r --level=1 -nv -nd -P "./external/tarballs" "https://go.suokunlong.cn:88/dl/libreoffice/external_tarballs/"

# 可以在 autogen.input 里配置编译选项，使用 ./autogen.sh --help 来查看这些选项
# vim autogen.input
./autogen.sh
make

# make 成功后，运行程序
./instdir/program/soffice
```

## 在特定发行版上的设置

### Gentoo

If there is anyone interested in libreoffice RISC-V, it's quite easy to try under Gentoo Linux

So, under Gentoo Linux RISC-V system, you just need to run commands

```shell
mkdir -p /etc/portage/patches/app-office/libreoffice-7.3.5.2/
cd /etc/portage/patches/app-office/libreoffice-7.3.5.2/
wget -nd https://dev.gentoo.org/~dlan/riscv/patches/libreoffice-7.3.5.2/01-libreoffice-riscv64.patch
wget -nd https://dev.gentoo.org/~dlan/riscv/patches/libreoffice-7.3.5.2/02-libreoffice-boost.patch
echo "=app-office/libreoffice-7.3.5.2 ~riscv" >> /etc/portage/package.accept_keywords/libreoffice
emerge =app-office/libreoffice-7.3.5.2
```

### Debian

#### 准备 riscv 环境

如有需要在 Debian 上构建 qemu chroot 或 qemu system，请参照[基础环境的搭建](/doc/history/%E5%9F%BA%E7%A1%80%E7%8E%AF%E5%A2%83%E7%9A%84%E6%90%AD%E5%BB%BA.md)的`快速建立一个debian riscv环境`一节

#### 依赖

```shell
apt-get install -y \
    build-essential zip ccache junit4 libkrb5-dev nasm graphviz python3 python3-dev \
    qtbase5-dev libkf5coreaddons-dev libkf5i18n-dev libkf5config-dev libkf5windowsystem-dev \
    libkf5kio-dev autoconf libcups2-dev libfontconfig1-dev gperf default-jdk doxygen \
    libxslt1-dev xsltproc libxml2-utils libxrandr-dev libx11-dev libxt-dev libassuan-dev \
    bison flex libgtk-3-dev libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev \
    ant ant-optional libnss3-dev libavahi-client-dev automake git wget
```

#### 源码

之前构建成功时使用的是基于 master 分支的[69485e422c9 提交](https://go.suokunlong.cn:88/cgit/lo/core/commit/?id=69485e422c9)

#### autogen.input 参数

```shell
--disable-firebird-sdbc
--with-galleries=no
--enable-debug
# 调整并行编译任务数
--with-parallelism=4
```

部分 external tarball 会在 debian 上编译时出错，可以考虑使用对应的系统内置的软件包，然后在 autogen.input 里启用`--with-system-<package name>`选项，具体要启用什么选项，请检查`./autogen.sh --help`的输出

## 较为详细的步骤

### 1. 准备好一个 riscv 环境

### 2. 安装依赖

请参照[Development/BuildingOnLinux - The Document Foundation](https://wiki.documentfoundation.org/Development/BuildingOnLinux)

### 3. 下载源码

可以使用 [tarball] ，目前稳定版为 [7.3.5.2](http://download.documentfoundation.org/libreoffice/src/7.3.5/) ， 能下载到的最高版本是 [7.4.0.1](http://download.documentfoundation.org/libreoffice/src/7.4.0/)

如要获取最新的版本，请直接克隆官方仓库

```shell
# 国外源
# git clone https://gerrit.libreoffice.org/core libreoffice
git clone --depth=1 git://go.suokunlong.cn/lo/core libreoffice
```

### 4. 打patch

[基础的patch](/config/01-libreoffice-riscv64.patch)在主目录的 config 文件夹下

如果需要编译 7.3.5 版本，请额外使用[补丁](/config/02-libreoffice-boost.patch)

```shell
wget https://raw.githubusercontent.com/Sakura286/libreoffice-riscv-port-memo/main/config/01-libreoffice-riscv64.patch
patch -p1 <01-libreoffice-riscv64.patch
# 在 7.3.5 版本上的额外步骤
# wget https://raw.githubusercontent.com/Sakura286/libreoffice-riscv-port-memo/main/config/02-libreoffice-boost.patch
# patch -p2 <02-libreoffice-boost.patch
```

### 5. 下载 external tarballs

国外仓库：`https://dev-www.libreoffice.org/src/`
国内缓存：`https://go.suokunlong.cn:88/dl/libreoffice/external_tarballs/`

这一步不是必要的，若提示正在从 dev-www.libreoffice.org 下载各种包，但墙内速度很慢的情况下，可以用 Ctrl + C 打断，下载对应的包到 external/tarballs 下，当然也可以提前全部下好：

```shell
wget -r --level=1 -nv -nd -P "./external/tarballs" "https://go.suokunlong.cn:88/dl/libreoffice/external_tarballs/"
```

总共大约 3GB 的数据量

### 6. configure

创建或修改主目录下的`./autogen.input`文件，添加参数，具体需要参加什么参数，请检查 `./autogen.sh --help` 的输出

然后运行:

```shell
./autogen.sh
```

### 7. make

```shell
make
```

这个过程对于 4 核心的 qemu-system 来说，可能长达 1 ~ 2 天

### 8. 运行软件

```shell
./instdir/program/soffice
```

## 其他

### 关于 autogen.input

- `./autogen.sh --help`可以查看编译打包用到的参数
- 建议使用 autogen.input 记录参数，这样运行 autogen.sh 时便不再需要指定参数了
- **修改**（不是新建） autogen.input 后直接 `make` 会自动运行 autogen.sh

## 常见问题
