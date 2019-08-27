## 4.1 Linux内核模块简介

编译出的内核本身并不需要包含所有功能，而在某些功能需要被使用的时候，其对应的代码被动态地加载到内核中的机制被称为`模块（Module）`。其特点如下：

- 模块本身不被编译入内核映像，从而控制了内核的大小。
- 模块一旦被加载，它就和内核中的其他部分完全一样。

内核模块相关的Linux命令有 `lsmod` `rmmod` `modinfo` `modprobe`。

## 4.2 Linux内核模块程序结构

一个Linux内核模块主要由如下几个部分组成：

- `模块加载函数`

  当通过insmod或modprobe命令加载内核模块时，模块的加载函数会自动被内核执行，完成本模块的相关初始化工作。

- `模块卸载函数`

  当通过rmmod命令卸载某模块时，模块的卸载函数会自动被内核执行，完成与模块卸载函数相反的功能。

- `模块许可证声明`

  许可证声明描述内核模块的许可权限，如果不声明LICENSE，模块被加载时，将收到内核被污染(`Kernel Tainted`)的警告。

  大多数情况下，内核模块应遵循GPL兼容许可权。Linux内核模块最常见的是以`MODULE_LICENSE（“GPL v2”）`语句声明。

- `模块参数(可选)`

  模块参数是模块被加载的时候可以传递给它的值，它本身对应模块内部的全局变量。

- `模块导出符号(可选)`

  内核模块可以导出的符号（symbol，对应于函数或变量），若导出，其他模块则可以使用本模块中的变量或函数。

- `模块作者等信息声明（可选）`

## 4.3 模块加载函数

Linux内核模块加载函数一般以__init标识声明，典型的模块加载函数的形式如下面的代码所示：

```c
static int __init initialization_function(void)
{
    /* 初始化代码 */
}
module_init(initialization_function);
```

加载模块函数以“`module_init（函数名）`”的形式被指定。它返回整型值，

- 若初始化成功，应返回0

- 而在初始化失败时，应该返回错误编码。

  > 在Linux内核里，错误编码是一个接近于0的负值，在 `<linux/errno.h>` 中定义.

在Linux内核编程过程中，可以使用 `request_module(const char*fmt，…)` 函数灵活地加载其他内核模块。

在Linux中，所有标识为 `__init` 的函数如果直接编译进入内核，成为内核镜像的一部分，在连接的时候都会放在`.init.text`这个区段内。

```c
#define __init __attribute__((__section__(".init.text")))
```

此外，所有的 `__init` 函数在区段`.initcall.init`中还保存了一份函数指针，在初始化时内核会通过这些函数指针调用这些`__init` 函数，并在初始化完成后，释放init区段（包括.init.text、.initcall.init等）的内存。

除了函数以外，数据也可以被定义为 `__initdata`，对于只是初始化阶段需要的数据，内核在初始化完后，也可以释放它们占用的内存。

## 4.4 模块卸载函数

Linux内核模块卸载函数一般以 `__exit` 标识声明，典型的模块卸载函数的形式如代码：

```c
static void __exit cleanup_function(void)
{
    /* 释放代码 */
}
module_exit(cleanup_function);
```

模块卸载函数在模块卸载的时候执行，而不返回任何值，且必须以“`module_exit（函数名）`”的形式来指定。

通常来说，模块卸载函数要完成与模块加载函数相反的功能。

- 若模块加载函数注册了 XXX，则模块卸载函数应该注销 XXX。
- 若模块加载函数动态申请了内存，则模块卸载函数应释放该内存。
- 若模块加载函数申请了硬件资源（中断、DMA 通道、I/O 端口和 I/O 内存等）的占用，则模块卸载函数应释放这些硬件资源。
-  若模块加载函数开启了硬件，则卸载函数中一般要关闭之。

除了函数以外，只是退出阶段采用的数据也可以用 `__exitdata` 来形容。

实际上，`__init`、`__initdata`、`__exitdata` 和 `__exit` 都是宏，其定义分别为： 

```c
#define __init __attribute__((__section__(".init.text")))

#ifdef MODULE
#define __exit __attribute__((__section__(".exit.text")))
#else
#define __exit __attribute_used__attribute_((__section__(".exit.text")))
#endif

#define __initdata __attribute__((__section__(".init.data")))
#define __exitdata __attribute__((__section__(".exit.data")))
```

## 4.5 模块参数

我们可以用“`module_param（参数名，参数类型，参数读/写权限）`”为模块定义一个参数。

例如下列代码定义了1个整型参数和1个字符指针参数：

```c
static char *book_name = "dissecting Linux Device Driver";
module_param(book_name, charp, S_IRUGO);
static int book_num = 4000;
module_param(book_num, int, S_IRUGO);
```

在装载内核模块时，用户可以向模块传递参数，形式为“`insmode（或modprobe）模块名 参数名=参数值`”。如果不传递，参数将使用模块内定义的缺省值。

> 如果模块被内置，就无法insmod了，但是 `bootloader`  可以通过在bootargs里设置“`模块名.参数名=值`”的形式给该内置的模块传递参数。

参数类型可以是 `byte、short、ushort、int、uint、long、ulong、charp（字符指针）、bool或invbool（布尔的反）`，在模块被编译时会将module_param中声明的类型与变量定义的类型进行比较，判断是否一致。

除此之外，模块也可以拥有参数数组，形式为“`module_param_array（数组名，数组类型，数组长，参数读/写权限）`” 。

模块被加载后，在 `/sys/module/` 目录下将出现以此模块名命名的目录。当“参数读/写权限”为 0时，表示此参数不存在 sysfs 文件系统下对应的文件节点，如果此模块存在“参数读/写权限”不为 0的命令行参数，在此模块的目录下还将出现 parameters 目录，包含一系列以参数名命名的文件节点，这些文件的权限值就是传入 module_param()的“参数读/写权限”，而文件的内容为参数的值。 

## 4.6 导出符号

Linux的 `/proc/kallsyms` 文件对应着内核符号表，它记录了符号以及符号所在的内存地址。

模块可以使用如下宏导出符号到内核符号表中：

```c
EXPORT_SYMBOL(符号名);
EXPORT_SYMBOL_GPL(符号名);
```

导出的符号可以被其他模块使用，只需使用前声明一下即可。

```c
#include <linux/init.h>
#include <linux/module.h>

int add_integar(int a, int b)
{
return a + b;
}
EXPORT_SYMBOL_GPL(add_integar);

int sub_integar(int a, int b)
{
return a - b;
}
EXPORT_SYMBOL_GPL(sub_integar);

MODULE_LICENSE("GPL v2");
```

## 4.7 模块声明与描述

在Linux内核模块中，我们可以用MODULE_AUTHOR、MODULE_DESCRIPTION、MODULE_VERSION、MODULE_DEVICE_TABLE、MODULE_ALIAS分别声明模块的作者、描述、版本、设备表和别名，例如：

```c
MODULE_AUTHOR(author);
MODULE_DESCRIPTION(description);
MODULE_VERSION(version_string);
MODULE_DEVICE_TABLE(table_info);
MODULE_ALIAS(alternate_name);
```

对于USB、PCI等设备驱动，通常会创建一个MODULE_DEVICE_TABLE，以表明该驱动模块所支持的设备:

```c
/* table of devices that work with this driver */
static struct usb_device_id skel_table [] = {
    { USB_DEVICE(USB_SKEL_VENDOR_ID,USB_SKEL_PRODUCT_ID) },
    { } /* terminating enttry */
};
MODULE_DEVICE_TABLE (usb, skel_table);
```

## 4.8 模块的使用计数

Linux 2.6以后的内核提供了模块计数管理接口 `try_module_get（&module）`和 `module_put（&module）`

```c
//函数用于增加模块使用计数；若返回为0，表示调用失败，希望使用的模块没有被加载或正在被卸载中。
int try_module_get(struct module *module);

//该函数用于减少模块使用计数。
void module_put(struct module *module);
```

Linux 2.6以后的内核为不同类型的设备定义了`struct module* owner`域，用来指向管理此设备的模块。

- 当开始使用某个设备时，内核使用 `try_module_get（dev->owner）`去增加管理此设备的owner模块的使用计数；
- 当不再使用此设备时，内核使用  `module_put（dev->owner）`减少对管理此设备的管理模块的使用计数。

这样，当设备在使用时，管理此设备的模块将不能被卸载。只有当设备不再被使用时，模块才允许被卸载。

在Linux 2.6以后的内核下，对于设备驱动而言，很少需要亲自调用try_module_get（）与 module_put（）。

## 4.9 模块的编译

Linux内核模块可以通过如下命令进行编译:

```bash
make -C 内核源码绝对路径 M=模块源码文件所在的绝对路径 modules  

#此命令的解释：
#1）进入到内核源码目录
#2）读取内核源码目录的Makefile
#3）执行内核源码Makefile中的 modules 目标
#4）根据 modules 目标的命令编译M所指向的文件路径 Makefile指定 的 C 文件
```

因此有如下的 编译模块所需的 Makefile 文件模板：

```Makefile
KVERS := $(shell uname -r)
#若定义了HC_KSD变量, 则以HC_KSD的取值替换默认的linux内核代码路径(/lib/modules/$(KVERS)/build)
KSRCDIR = /lib/modules/$(KVERS)/build
ifdef HC_KSD
KSRCDIR := $(HC_KSD)
endif

MAKE = make #代码默认不进行交叉编译
#若定义了HC_CTP变量, 则使用【aarch64-linux-gnu-】交叉编译工具链交叉编译到arm64上
ifdef HC_CTP
MAKE := make CROSS_COMPILE=aarch64-linux-gnu- ARCH=arm64
endif

# WHAT_YOU_NEED_TO_CHANGED
#指定驱动模块的名字，注意：驱动模块的名字千万不能和本文件夹内的任何源文件同名！
MOD_NAME := hello
#指定驱动模块的所有依赖源文件
MOD_SRCS := hc_hello.c

#最终由XXX-objs链接生成$(MOD_NAME).o ，再生成$(MOD_NAME).ko
$(MOD_NAME)-objs := $(MOD_SRCS:.c=.o)
obj-m += $(MOD_NAME).o

# 模块编译额外FLAG, 放开后将以【完全不优化】模式编译模块
# EXTRA_CFLAGS = -g -O0

build: modules
modules clean:
	$(MAKE) -C $(KSRCDIR) M=$(CURDIR) $@
help:
	@echo -e '\n[HC_KSD=/path/to/kernel/srcdir] [HC_CTP=1] make [{build|clean}]\n'
```

该Makefile文件应该与源代码hello.c位于同一目录，开启其中的EXTRA_CFLAGS=-g-O0，可以得到包含调试信息的hello.ko模块.

**`内核用EXPORT_SYMBOL_GPL（）导出的符号是不可以被非GPL模块引用的`**。

是一般而言，产品在启动过程中应该加载模块，在嵌入式产品 Linux 的启动过程中，加载企业自己的模块的最简单的方法是修改启动过程的 rc 脚本，增加 insmod /.../xxx.ko 这样的命令。用 busybox 做出的文件系统，通常修改 etc/init.d/rcS 文件。 

