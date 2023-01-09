
# 在 Debian QEMU 中构建 LibreOffice

基础平台：Debian sid x86
构建时间：2022-08-26 ~ 28

## 1. 准备 Debian QEMU riscv64

这里直接使用[DQIB](https://people.debian.org/~gio/dqib/)的 riscv64 镜像，下载下来后是一个叫做`artifacts.zip`的文件
注：当然，使用 debootstrap 制作 chroot 环境后再使用 virt-make-fs 将其打包成镜像，再或者是其他方法也都可以。

```shell
unzip artifacts.zip
cd artifacts
# 具体安装的依赖，以及传给 qemu 的参数，可以查看 artifacts 内的 readme.txt
sudo apt install qemu-system-misc opensbi u-boot-qemu
```

QEMU 镜像的管理员账户与密码均为`root`，普通用户的账户与密码为`debian`，注意，**LibreOffice不允许在 root 下构建**

下面是我使用的 QEMU 启动脚本

```shell
#!/bin/bash
# 启动脚本，注意修改内存与 CPU 数量
qemu-system-riscv64 -nographic -machine virt -m 8G -smp 8,sockets=1,cores=8 \
 -bios /usr/lib/riscv64-linux-gnu/opensbi/generic/fw_jump.elf \
 -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
 -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-device,rng=rng0 \
 -append "console=ttyS0 rw root=/dev/vda1" \
 -device virtio-blk-device,drive=hd0 -drive file=image.qcow2,if=none,id=hd0 \
 -device virtio-net-device,netdev=usernet -netdev user,id=usernet,hostfwd=tcp::22222-:22
```

## 2. 编译前的准备

从现在开始所有的操作均在 QEMU 内进行，请确保已经完成以下操作

1. 硬盘扩容（至少给个30G吧？）
2. 使用普通用户（ LibreOffice 不允许在 root 下构建）

```shell
# 安装依赖及基本工具
# update-initramfs 时可能会耗费非常久的时间，大约需要十来分钟
sudo apt install -y  \
    build-essential zip ccache junit4 libkrb5-dev nasm graphviz python3 python3-dev \
    qtbase5-dev libkf5coreaddons-dev libkf5i18n-dev libkf5config-dev libkf5windowsystem-dev \
    libkf5kio-dev autoconf libcups2-dev libfontconfig1-dev gperf default-jdk doxygen \
    libxslt1-dev xsltproc libxml2-utils libxrandr-dev libx11-dev libxt-dev libassuan-dev \
    bison flex libgtk-3-dev libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev firebird-dev \
    ant ant-optional libnss3-dev libavahi-client-dev automake git vim wget tar xz-utils unzip

# 下载源码（这里使用了国内镜像）
git clone --depth=1 git://go.suokunlong.cn/lo/core ./libreoffice
cd libreoffice

# external tarballs 最好提前下下来，这里使用了国内镜像
# 一共大约 3G ，请耐心等待
wget -r --level=1 -nv -nd -P "./external/tarballs" "https://go.suokunlong.cn:88/dl/libreoffice/external_tarballs/"
# --with-system-firebird
# --enable-debug
# --with-parallelism=12
vim autogen.input
ccache -M 32G
```

## 3. 编译与运行

```shell
./autogen.sh
make
# 构建完成后的运行
./instdir/program/soffice
```
