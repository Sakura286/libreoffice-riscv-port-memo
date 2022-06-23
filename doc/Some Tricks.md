
# Some Tricks

## ccache

若启用ccache，确实能在修改代码（或上次中断）后的增量编译中获得很大的加速

ccache缓存默认大小是4G，据说编libreoffice得8G，做一些libreoffice开发的话可能需要32G（用这么模糊的说法是我忘了出处了），可以这样设置：

```shell
ccache -M 32G
```

## 調整并行编译

在命令行使用make时使用--jobs（或-j）参数是不起作用的，如果默认设置的并行数有误，可以在调用autogen.sh时加上--with-parallelism=<jobs_num>参数，<jobs_num>可以和CPU核心数一致

PS：在12代酷睿的大小核环境下，我的vmware优先吃小核，怎么办呢

## 调整log输出等级

```shell
make -d
make --debug[=options]
```

`https://www.gnu.org/software/make/manual/html_node/Options-Summary.html`

## 输出log到文件中

`make 2>error.log`只将错误输出到文件中
所以结合上面，可以用如下命令
`make -d >out.log 2>error.log`

`https://blog.csdn.net/weixin_37548620/article/details/105972568`

## 交叉编译中的 build 、 host 与 target

--build: 编译该软件所使用的的平台
--host: 该软件将运行的平台
--target: 该软件所要处理的目标平台

例如，我在 windows(build) 上编译了运行在 linux(host) 上处理 mips64(target) 代码的gcc。
