
# 一些感悟

## 交叉编译中的 build 、 host 与 target

--build: 编译该软件所使用的的平台
--host: 该软件将运行的平台
--target: 该软件所要处理的目标平台

例如，我在 windows(build) 上编译了运行在 linux(host) 上处理 mips64(target) 代码的gcc。
