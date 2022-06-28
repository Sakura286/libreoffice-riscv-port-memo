
FROM riscv64/debian:unstable

LABEL maintainer="vimer, Sakura286"

RUN apt-get update -y && apt-get install ca-certificates -y

RUN echo "deb  http://mirror.iscas.ac.cn/debian-ports unstable main" > /etc/apt/sources.list

# automake and libassuan-dev added
RUN apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive \
    apt-get install -y --no-install-recommends \
    build-essential zip ccache junit4 libkrb5-dev nasm graphviz python3 python3-dev \
    qtbase5-dev libkf5coreaddons-dev libkf5i18n-dev libkf5config-dev libkf5windowsystem-dev \
    libkf5kio-dev autoconf libcups2-dev libfontconfig1-dev gperf default-jdk doxygen \
    libxslt1-dev xsltproc libxml2-utils libxrandr-dev libx11-dev libxt-dev libassuan-dev \
    bison flex libgtk-3-dev libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev \
    ant ant-optional libnss3-dev libavahi-client-dev automake git vim wget tar xz-utils unzip \
    && rm -rf /var/lib/apt/lists/*

# WORKDIR /root/
#ENV LO_VERSION="7.3.4.2"
#RUN wget http://download.documentfoundation.org/libreoffice/src/${LO_VERSION%.*}/libreoffice-${LO_VERSION}.tar.xz \
#    && tar -xf libreoffice-${LO_VERSION}.tar.xz \
#    && wget -r --level=1 -nv -nd -P "./libreoffice-7.3.4.2/external/tarballs" "https://go.suokunlong.cn:88/dl/libreoffice/external_tarballs/" \
#    && rm libreoffice-${LO_VERSION}.tar.xz
#WORKDIR /root/libreoffice-${LO_VERSION}

RUN cd /root/ \
    && git clone git://go.suokunlong.cn/lo/core ./libreoffice \

RUN cd /root/libreoffice \
    && git reset --hard 69485e422c90 \
    && wget -r --level=1 -nv -nd -P "./external/tarballs" "https://go.suokunlong.cn:88/dl/libreoffice/external_tarballs/" \
    && wget https://raw.githubusercontent.com/Sakura286/libreoffice-riscv-port-memo/main/config/0001-stage.patch \
    && patch -p1 <0001-stage.patch \
    && wget https://raw.githubusercontent.com/Sakura286/libreoffice-riscv-port-memo/main/config/autogen.input \
    && ccache -M 32G
