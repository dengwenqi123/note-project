## Recipe(配方)

一个bitable 配方是由一组命令构成，这些命令描述了获取源码、打补丁，添加附近文件，编译，安装，和产生二进制包。构建完成之后最终得到的是一个二进制软件包，还有一些中间文件，比如库和头文件。

## 配方符号

构成一个“配方”文件的基本要素包括：

- 函数

函数提供了一系例要执行的动作。函数通常用来替代一个默认的"任务"函数，或者是完善和增强。标准的函数使用sh shell 符号，可以访问oe的变量和内部方法。

下面是一个 配方示例:

```bash
do_install() {
	autotools_do_install
  install -d ${D}${base_bindir}
  mv ${D}${bindir}/sed ${D}${base_bindir}/sed.${PN}
}
```

你自己也可以实现一个全新的函数，而不是已有函数的替代，函数在已有“任务”之间被调用。你也可以使用python来实现一个函数替代sh的方式。

### 变量的赋值和操作

变量赋值有可能是静态的文本或者包含其他变量的值。也可以给一个变量追加值。

```bash
S = "${WORKDIR}/postfix-${PV}"
PR = "r4"
CFLAGS += "-DNO_ASM"
SRC_URI_append = "file://fixup.patch;patch=1"
```

#### 关键字

Bitable 只有极少的关键字, 包含 inherit include require 和export

```bash
 export POSTCONF = "${STAGING_BINDIR}/postconf"
 inherit autoconf   
 require otherfile.inc
```

#### 条件赋值

条件赋值用来当一个变量还没有被赋值的时候赋值。通常用来给一个变量提供初始值。

```bash
# 如果VAR1 当前没有赋值 那么就赋值给 "New value".但是如果变量已经赋值的时候，结果还是原来的值。
VAR ?= "New value"
VAR1 = "Original value"
VAR1 ?= "New value"
# VAR1的值还是"Original value"
```

#### 变量值追加(有空格): +=

你可以给已经有值的变量使用符号"+="追加一个值。**注意**这个操作会在原值和追加的值中间添上空格：

```bash
SRC_URI += "file://fix-makefile.patch;patch=1" 
```

#### 变量值前加：=+

你可以给已有值的变量内容前面加上一个值，使用符号"=+"。**注意**这个操作会在原值和你追加的值中间添加空格：

```bash
VAR =+ "Starts"
```

#### 变量值追加：_append方式

你可以使用_append 方式来给一个已经存在的变量追加值。跟+=不一样的是这种方式不会在原值和 你要追加的值中间加上空格。

```bash
SRC_URI_append = " file://fix-makefile.patch;patch=1"
```

_append方式也可以用来覆盖某个值，但是仅仅是在指定目标机器和平台的时候有用：

```bash
SRC_URI_append_sh4 = " file://fix-makefile.patch;patch=1"
```

你可以把“追加符”理解为变量的自己本身。所以 += 和 =+ 符号可以和_append 一起来使用。比如：

```bash
SRC_URI_append = " file://fix-makefile.patch;patch=1"
SRC_URI_append += "file://fix-install.patch;patch=1"
```

#### 变量前加： _prepend方式

使用_prepend 在已有的变量前面加上一个值。和\_append 一样，这里\_append和=+的区别是有没有空格。

```bash
CFLAGS_prepend = "-I${S}/myincludes "
```

同样在指定机器名的时候\_prepend也是覆盖的意思。如：

```bash
CFLAGS_prepend_sh4 = " file://fix-makefile.patch;patch=1"
```

#### 使用python 的高级操作： ${@...}

为了获取更高级的功能，你可以使用python语句。比如变量替换和搜索。

python语句在声明变量之前要添加@符号。

```bash
 CXXFLAGS := "${@'${CXXFLAGS}'.replace('-frename-registers','')}"
```

### "配方"的命名： 名称，版本，和发布号

OE里配方的命名遵循一个规定。名字包括名称和版本两个部分，还有一个可选的发布号。发布号表明了这是这个包的第几次构建。发布号是包含在”配方“里的。

符号规定的”配方“名称应该这样：

```bash
<包名>_<版本>.bb
```

包名是软件包的名字（不管这个软件包是应用程序，库，模块，或者其他什么），版本部分就是版本号。所以一个典型的“配方”名应该是这样：

```bash
#表明是strace包的4.5.14版
strace_4.5.14.bb
```

发布号是在PR变量里定义的，包含在"配方"文件里。正确的格式为：

```bash
# <n>代表一个从0开始的任意整数。
r<n>
# 示例
PR = "r1"
```

如果在“配方”里没有定义PR变量那么就会使用默认值"r0".

 **注意，在任何时候你都应该使用发布号，即便是发布号0。同时发布号应该是递增的。**

当一个"配方"在被处理时，一些变量会根据“配方”自动设置，所以在任何“配方”里你都可以使用这写变量。这些变量包括：

**PN** : 包名。由“配方”的文件名所决定。bitbake处理一个“配方”的时候会自动设置。比如对于“配方” strace_4.5.14.bb对应的PN就是"strace" 。

**PV** : 包版本。有些“配方”的文件名决定。比如对于“配方”strace_4.5.14.bb对应的PV就是 "4.5.14"

**PR** : 包的发布版本号。这个是在“配方”文件里面设置的。如果没有设置默认值是"r0".

**P** : 软件包名。由包名和包版本两部分组成。

```bash
#示例 strace_4.5.14.bb,P就是"strace-4.5.14"
P = "${PN}-${PV}"
```

**PF**: 带发布号的软件包名。

```bash
PF = "${PN}-${PV}-${PR}"
```

对这里的starce_4.5.14.bb“配方”文件来说，PR是“r1”，所以PF就是"strace-4.5.14-r1"

在下面的例子中，我们指定系统包含一个附加的目录，但是我们没有直接写出包的名字，而是使用变量来完成的：

```bash
FILES_${PN} += "${sysconfdir}/myconf"
# 我们指定源码包的下载地址，使用PV变量来指定包的版本号，这样当我们升级的包的版本，重命名一个“配方”的时候就不用改动了
SRC_URI = "ftp://ftp.vim.org/pub/vim/unix/vim-${PV}.tar.bz2"
```

### 目录： 什么目录？ 放在哪？

下面是一些在“配方”里要经常使用的目录，

**工作目录：WORKDIR** : 这是一个“配方”的源码文件被解压的地方，其他普通的文件也会拷贝到这里，然后这里还会建立log目录，安装文件也在这里创建。

**源码目录：S** : 这是程序的源码目录，补丁会被应用到这里，程序的编译也在这里进行。

**目标目录：D** : 这是一个包的程序被编译出来安装的目标目录。打包程序的时候就是从这里获取文件的。

### hello world 示例

现在你已经具备了写一个基本的“配方”的知识了。我们将示例一个简单的单文件的“配方”，然后再
讲一个使用autotool管理的软件包的“配方”，看看如何为一个用使用autotool的软件包写“配方”。

```bash
mkdir packages/myhelloworld
mkdir packages/myhelloworld/files
cat > packages/myhelloworld/files/myhelloworld.c
#include <stdio.h>

int main(int argc, char** argv)
{
printf("Hello world!\n");
return 0;
}
^D
cat > packages/myhelloworld/files/README.txt
Readme file for myhelloworld.
```

现在我们为我们的“配方”创建了一个目录：packages/myhelloworld, 而且我们创建了一个 files目录来保存本地文件。我们创建了两个本地文件。一个是helloworld程序的c代码，一个是 readme。现在我们来编写“配方”。

```bash
DESCRIPTION = "My hello world program"
PR = "r0"

SRC_URI = "file://myhelloworld.c \
           file://README.txt"
# 提供一个do_compile函数,然后提供适当的命令：
do_compile() {
    ${CC} ${CFLAGS} ${LDFLAGS} ${WORKDIR}/myhelloworld.c -o myhelloworld
}
 do_install() {
	  install -m 0755 -d ${D}${bindir} ${D}${docdir}/myhelloworld
    install -m 0644 ${S}/myhelloworld ${D}${bindir}
    install -m 0644 ${WORKDIR}/README.txt ${D}${docdir}/myhelloworld
 }
```

 **注意：**
-  这里使用预定义的编译器变量，\${CC},\${CFLAGS}和\${LDFLAGS}. 这些会自动设置交叉编译这个软件的信息。
- 这里使用了\${WORKDIR}来定位文件。就像之前提到的一样，所有的文件会被拷贝的工作目录
      里，然后可以使用${WORKDIR}来提取它。

下面执行如下命令构建我们的包。

```bash
bitbake -b packages/myhelloworld/myhelloworld_0.1.bb
```

包成功构建后，会产生两个.ipkg包，可以直接给目标机器安装。一个是二进制包，一个是包含readme文档。

**需要注意**：

- 两个源码文件在tmp/work/myhelloworld-0.1-r0里，也即${WORKDIR}变量代表的工作目录。
- 在tmp/work/myhelloworld-0.1-r0/temp目录里有各种任务的日志信息。
- tmp/work/myhelloworld-0.1-r0/image是镜像目录，里面包含了要打包的文件。这就是实际上的“目标目录”。这里原本是有那两个文件的，但是在进行后面的安装任务时，这两个文件被移动到install（安装目录）了。
- 程序是在tmp/work/myhelloworld-0.1-r0/myhelloworld-0.1目录里编译的。这就是${S}变量指定的目录。
- tmp/work/myhelloworld-0.1-r0/install就是安装目录，包含了将被直接打包的文件。我们看到myhelloworld-doc包包含了/usr/share/doc/myhelloworld/README.txt单文件，myhelloworld包包含了/usr/bin/myhelloworld单个文件，然后-dev,-dbg和-local包都是空的。































