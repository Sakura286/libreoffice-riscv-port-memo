
# UNO 浅显理解

注：本文根据[https://wiki.openoffice.org/wiki/Uno/Article/Understanding_Uno](https://wiki.openoffice.org/wiki/Uno/Article/Understanding_Uno) 整理总结，为了与原文保持一致，文中使用OpenOffice(LibreOffice的前身)进行阐述，请注意。

UNO(Universal Network Objects)是OpenOffice的核心概念： 因为它管理着OpenOffice的所有内部对象。UNO的作用就是提供一套机制，让OpenOffice的所有对象无关平台或者执行环境而能够互相调用。例如，``OOoBasic``对象中的一个method call转换成UNO中实现的call method. UNO的另外一个特征是允许外部程序可以通过网络接口在OpenOffice上运行。

## 术语表

**API**： OpenOffice供外部用户使用的methods和attributes集合；也被称作UNO Interface Definition Language(UNO-IDL)。

**Binding：**将IDL的某些实现翻译成特定的编程语言。一个Binging也包括一些与语言相关的方法和帮助程序来处理UNO概念。

**Bridge**: 这个概念与Binging概念紧密联系：A bridge对于UNO来说就是一段可以binding到特定语言的代码段。如果想开发(扩展) Bridge的功能，需要对UNO及它本身的机制有一个清晰的了解。

**Component：**组件。根据spec组成的描述集，原则是最好彼此之间无依赖。

**IDL(***Interface Description Language***)：**common specification description language(通用规范描述语言)。这个语言被用来描述由Component定义的类型。它也根据binding的不同，以特定语言的方式进行转换。一般以".idl"为扩展名。

**Registry：**一个由component specs规则生成binary文件。有两类registry: 一个是types registries包含UNO定义的类型；另一个是services registries linking the specification to the implementation，一般以".rdb"为扩展名。

**URE(**UNO Runtime Environment**)：**允许基于其他UNO应用的子集。

# UNO基础

## Specification and Implementation

这里要注意区分自然语言定义的specs和UNO的里specs是不同的：UNO的specs是定义某个API的一套methods和properties的集合，通常由IDL语言来进行描述。

通常，一个完整的component，会包含一个UNO spec及它的实现(implementation), implementation可以是任何被UNO支持的语言。如果一门语言支持UNO，则需要创建"Bridge"连接二者。最常见的支持UNO的语言有C++, Java和python。

正如上所述，一个完整的 component call则会返回一个UNO Runtime Environment，该URE然后会使用services registeries将这个call翻译成 implementation call。一个UNO  components结构包含三部分：

* A type register: specs in binary file
* 实现(implementation)通常由共享库提供
* The services registry描述了指定类型及其实现之间的映射

## IDL first approach

UNO-IDL的主要类型有 services和interfaces。 interface定义了methods和attributes的集合而services则是导出一个interface.

interface可以多继承，serives则不支持多继承。

## Remote objects

UNO不会让client  code 直接处理object instance: 它使用proxy对象搭建一个bridge连接 implementation 和 client code language。
