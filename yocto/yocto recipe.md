## yocto recipe 

本篇文章主要是利用 yocto 的utilities 工具来产生 recipe. 和将完成的recipe 加入到image中。

### Yocto 相关工具介绍

有三个用于layer 和 recipe 相关的 utilities工具。

1. bitable-layers: 负责 新建layer 、修改bblayers.conf、查看 layer底下的recipes
2. recipetool: 用于建立一个recipe 和新增/修改 recipe.这边recipe在 yocto 即是 xx.bb 或 xx.bbappend结尾的文件。
3. devtool: 定位在开发测试上。devtool 会建立一个独立的workspace layer.让开发者在此新建/修改recipe和 patch 的source code。在不修改原有设定下和现有layers 进行整合测试。完成之后可以再通过devtool新增或更新到layer 里。

使用utilities 编写recipe 的时候，要确保所属的layer 是存在于 bblayer.conf, 不然执行 bitbake [recipe basename]  会找不到相关的recipe。

### 准备工作

以core-image-minimal recipe 为例。core-image-minimal recipe 会产生系统所需的image 相关文件。接着利用bitable-layers 建立自己的layer 和build 目录。在通过bitbake-layers 把layer 加入到belayers.conf中。

```bash
source oe-init-build-env
bitbake core-image-minimal
bitbake-layers create-layer meta-layer
bitbake-layers add-layer meta-layer
```

#### 新增recipe

以devtool 和recipetool示范如何产生recipe.由工具产生recipe的时候，若source code 来源为(git,ftp,tar,etc) 以及编译方式(came,make,etc)是可以被解析的。经工具所产生的recipe 几乎都不需要多做修改。以下说明两个范例各自的情境：

- devtool 范例：使用GIFLIB 套件。 由于GIFLIB 基于 autotool,configure 和 Makefile.devtool 可以分析并帮助我们产生无需修改的recipe.
- recipetool范例：这边以编程的方式。程序只会有header file 和 source file. 并搭配Makefile 来编译的方案。所以recipetool 产生的recipe 需要修改。

#### 1.devtool命令

devtool 产生 recipe时候，会先建立一个workspace layer,并在目录底下产生 giflib recipe. 由于source code 格式符合yocto, 产生出来的recipe 并不需要额外修改。

```bash
devtool add giflib git://git.code.sf.net/p/giflib/code

Sstate summary: Wanted 0 Found 0 Missed 0 Current 0 (0% match, 0% complete)
NOTE: No setscene tasks
NOTE: Executing Tasks
NOTE: Tasks Summary: Attempted 2 tasks of which 0 didn't need to be rerun and all succeeded.
INFO: Using default source tree path /work/baseos/build/workspace/sources/giflib
NOTE: Reconnecting to bitbake server...
NOTE: Retrying server connection (#1)...
NOTE: Reconnecting to bitbake server...
NOTE: Reconnecting to bitbake server...
NOTE: Retrying server connection (#1)...
NOTE: Retrying server connection (#1)...
NOTE: Starting bitbake server...
INFO: Using source tree as build directory since that would be the default for this recipe
INFO: Recipe /work/baseos/build/workspace/recipes/giflib/giflib_git.bb has been automatically created; further editing may be required to make it fully functional

ls . workspace/recipes/giflib/
bitbake-cookerdaemon.log  cache  conf  downloads  meta-layer  sstate-cache  tmp  workspace

workspace/recipes/giflib/:
giflib_git.bb
```

接着devtool build giflib 来进行编译，这边只有单纯进行编译的动作而已。

```bash
devtool build giflib
ls tmp/work/i586-poky-linux/giflib/5.1.4+git999-r0/
```

执行 devtool build-image core-image-minimal 把 giflib 打包成 rpm package 并加入到 core-image-minimal 的 image 里面。之后可以看到giflib \${WORKDIR} 底下多了deploy-rpms 目录。同样在tmp/deploy/rpm/i586/ 目录和 core-image-minimal 的roots 目录底下皆有giflib 的相关文件。

```bash
devtool build-image core-image-minimal
ls tmp/work/i586-poky-linux/giflib/5.1.4+git999-r0/deploy-rpms/i586/
ls tmp/deploy/rpm/i586/ | grep giflib
ls tmp/work/qemux86-poky-linux/core-image-minimal/1.0-r0/rootfs/usr/lib/
```

上述过程都沒有任何问题后. 再用 devtool finish giflib meta-layer. 把 giflib recipe 加入到自己建立的 meta-layer 底下. 以上就是经由 devtool 完成 recipe 的过程。

```bash
devtool finish giflib meta-layer
ls meta-layer/recipes-giflib/giflib/
```

#### 2.recipetool命令

以hello project  为例子。 首先建立meta-layer/recipes-myproject/myproject/files  目录，并把程序放置在该目录。程序的内容如下：

```bash
ls meta-layer/recipes-myproject/myproject/files/
```

hello.h

```c++
#ifndef HELLO_EXAMPLE_H
#define HELLO_EXAMPLE_H
/*
 * Example function
 */
extern void LibHelloWorld(void);

#endif /* HELLO_EXAMPLE_H */
```

hellolib.c 

```c++
#include <stdio.h>
#include "hello.h"

void LibHelloWorld()
{
  printf("Hello World (from a shared library!)\n");
}
```

hell.c

```c++
#include <stdlib.h>
#include "hello.h"

int main(int argc, char *argv[])
{
  printf("Hello Yocto World...\n");
  LibHelloWorld();
  return 0;
}
```

Makefile 撰写如何编译此专案的规则. 若是编译过程有发生 No GNU_HASH in the elf binary 错误. 可参阅 [6]

```makefile
SRCS := $(wildcard *.c)
OBJS := $(SRCS:.c=.o)
# hello
HELLO_OBJS := $(filter hello%.o, $(OBJS))
TARGET_HELLO := hello
# targets
TARGET :=  $(TARGET_HELLO)
# varables used in yocto
bindir ?= /usr/bin

LIBS :=

.PHONY: clean all

all: $(TARGET)

%.o : %.c
 $(CC) -c $< -o $@ ${LDFLAGS} ${LIBS}

$(TARGET_HELLO) : $(OBJS)
 $(CC) $(HELLO_OBJS) -o $@ ${LIBS} ${LDFLAGS}

install:
 install -d $(DESTDIR)$(bindir)
 install -m 755 $(TARGET_HELLO) $(DESTDIR)$(bindir)/
```

最后需要修改myproject.bb 文件，由于采用Makefile格式，只需要修改两个地方即可。

1. 修改SRC_URI 为 file://*. 由于只存在 source code 于files 目录。但是并没有指定是哪些文件。所以用 * 来表示所以文件
2. 设定 S 为 \${WORKDIR} 指定build 的目录所在位置。

```bash
LICENSE = "CLOSED"
LIC_FILES_CHKSUM = ""

# modify SRC_URI from "" to "file://*"
SRC_URI = "file://*"

# add this line.
# P.S. S => The location in the Build Directory where unpacked recipe source code resides.
S = "${WORKDIR}"

do_configure () {
}
do_compile () {
 oe_runmake
}
do_install () {
 oe_runmake install 'DESTDIR=${D}'
}
```

在执行bitbake my project 进行编译，可以看到编译成功以及所产生的相关文件。以上就是recipetool的方法。

```bash
bitbake myproject
ls tmp/work/i586-poky-linux/myproject/1.0-r0/
```

### 新增 recipe 到image

新的recipe 编写完毕后，便需要想办法加入到image 里面。而[4]有列出几种方式。

然而我的目标是不修改其他 recipe 下，把新的recipe 加入到image里，所以采用修改local.conf的方式，如下所示

- 修改local.conf

在local.conf 的 IMAGE_INSTALL 属性中添加所需的recipe。 IMAGE_INSTALL_append = "[recipe_basename]" (文件不推荐这种写法 IMAGE_INSTALL += "[recipe_basename]")

\# e.g. IMAGE_INSTALL_append = " strace"

如果想限制于特定产生 image 的 recipe

IMAGE_INSTALL_append_pn-core-image-minimal = " [recipe_basename]"
\# e.g. IMAGE_INSTALL_append_pn-core-image-minimal = " strace"

**IMAGE_INSTALL** 定义：

Specifies the packages to install into an image. The IMAGE_INSTALL variable is a mechanism for an image recipe and you should use it with care to avoid ordering issues.

以上述的两个recipes 来说，只需要在build/local.conf 加入此行

```bash
# append this line on build/local.conf
IMAGE_INSTALL_append = " giflib myproject"
```

先查看roots 底下的文件。当前是没有giflib 和 myproject. 接着 bitbake core-image-minimal. 再次查看就会看到相关的文件被安装了。

```bash
ls tmp/work/qemux86-poky-linux/core-image-minimal/1.0-r0/rootfs/usr/lib/
libkmod.so.2  libkmod.so.2.3.2  opkg
 bitbake core-image-minimal
 ls tmp/work/qemux86-poky-linux/core-image-minimal/1.0-r0/rootfs/usr/lib/
libgif.so.7  libgif.so.7.0.0  libkmod.so.2  libkmod.so.2.3.2  opkg
ls tmp/work/qemux86-poky-linux/core-image-minimal/1.0-r0/rootfs/usr/bin/
cmp         env       gifclrmp  hello           less           nc        readlink   shuf     time        unzip                wall.sysvinit
...
clear     dumpleases  gifbuild  head      last.sysvinit   mkfifo         printf    sha256sum  tftp     unlink      wall                 yes

```















### 以 devtool 产生 .bbapend

新增meta-custom-devtool-patch layer 来存放devtool 之后产生的bbappend.

```bash
cd /work/baseos/build
# 建立 meta-custom-devtool-patch layer
bitbake-layers create-layer meta-custom-devtool-patch
bitbake-layers add-layer meta-custom-devtool-patch
#建立 giflib 目录
mkdir meta-custom-devtool-patch/recipes-example/giflib
recipetool create https://github.com/webosose/activitymanager.git -o  meta-custom/recipes-example/yocto/verilator.bb
```



