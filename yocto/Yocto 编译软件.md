## Yocto 编译软件

> Yocto 提供了许多 fetch code 的方式[3],大多数都是采用抓取外网方式。我们一般作为开发，需要以本机上的source code 作为来源进行编译。这篇文章主要介绍如何让yocto 编译本地的source code。

### 使用本机 source code 

这边介绍两种方式：

1. **git fetcher**: 基于 git fetcher,仅需要修改SRC_URI 参数，就可以使用本机上的source code 。
2. **external source**: 基于externalsrc class: 有两种方式：
   1. 直接在recipe 中新增 external source 。
   2. 在 local.conf 新增特定recipe 的external source。

### 注意

本文示例是基于 tvos_base_image镜像中构建baseos为参考。

### 准备工作

在build 目录下新建3rd-pkgs。 把下载的giflib 程序放入该目录：

```bash
# 初始化环境变量
source mina/oe-init-build-env
# 进入构建目录
cd /work/baseos/build/
mkdir -p 3rd-pkgs && cd 3rd-pkgs
# 下载库文件
wget https://downloads.sourceforge.net/giflib/giflib-5.1.4.tar.bz2
```

新增 meta-test 层，并在meta-layer 底下新增 giflib 目录，存放之后的 bb files。

```bash
cd /work/baseos/build/
bitbake-layers create-layer meta-test
bitbake-layers add-layer meta-test
mkdir meta-test/recipes-example/giflib
```

### git fetcher

使用git fetcher 有两个重点：

1.  SRC_URI 的 protocol 参数需要改为 file。
2. S 的路径要改为 S="\${WORKDIR}/git"。

2的原因可以参考[4]。设置 source code 目录名称会根据来源而修改。当为tar 之类的压缩文件，名称通常为 S="\${WORKDIR}/\${BPN}-${PV}"。当为git的话，则为 S="\${WORKDIR}/git"。

#### 示例

首先进入 3rd-pkgs 目录解压 gitflib, 并进入giflib的目录。新建一个git 仓库，提交之后记录commit id 。

```bash
#解压缩
cd /work/baseos/build/3rd-pkgs
tar jxvf giflib-5.1.4.tar.bz2
#新建git 仓库
cd giflib-5.1.4
git init .
git add .
git commit "add package"
git log -1
commit 8a4a8ab1b64efab82b7f48eaebc3d9acea827779 (HEAD -> master)
Author: dengwenqi <dengwenqi@xiaomi.com>
Date:   Tue Mar 9 08:21:23 2021 +0000

    add package
    
```

在 meta-test/recipes-example/giflib 新增 giflib-git_5.1.4.bb,文件并设置如下内容，重点在 SRC_URI 和 S的数值。 SRCREV 则填写刚刚输出的commit id即可。

```bash
LICENSE = "Unknown"
LIC_FILES_CHKSUM = "file://COPYING;md5=ae11c61b04b2917be39b11f78d71519a"

S = "${WORKDIR}/git"
SRC_URI = "git:///work/baseos/build/3rd-pkgs/giflib-5.1.4;protocol=file;branch=master"
#SRC_URI = "git:///media/disk1-1/Github/poky/build/3rd-pkgs/giflib-5.1.4;protocol=file;branch=master"
SRCREV = "8a4a8ab1b64efab82b7f48eaebc3d9acea827779"

EXTRA_OECONF = ""

inherit autotools
```

执行bitbake giflib-git 看编译结果

```bash
cd /work/baseos/build
bitbake giflib-git
```

### external source 

这边有修改recipe 的方式和 修改 local.conf 两种方式的示例。

**注意**：记得一定要 inherit externalsrc 不管是全局(local.conf) 或者局部(.bb or .bbappend)的方式。

#### 1.recipe 方式

只需要在recipe 设定 external source，就可以使用external source code。

示例：

先在 3rd-pgks 目录底下拷贝giflib-5.1.4名为giflib-5.1.4_ext. 作为source 来源。

```bash
cd /work/baseos/build/3rd-pkgs
cp -r giflib-5.1.4 giflib-5.1.4_ext
```

在 meta-test/recipes-example/giflib 新增 giflib-ext_5.1.4.bb. 设定如下内容，主要记得在bb file 加上 `inherit externalsrc`、**EXTERNALSRC** 和 **EXTERNALSRC_BUILD** 三个参数。

```bash
LICENSE = "Unknown"
LIC_FILES_CHKSUM = "file://COPYING;md5=ae11c61b04b2917be39b11f78d71519a"

EXTRA_OECONF = ""
#
inherit autotools

inherit externalsrc

EXTERNALSRC = "/work/baseos/build/3rd-pkgs/giflib-5.1.4_ext/"
EXTERNALSRC_BUILD = "${B}"
```

进行编译

```bash
bitbake giflib-ext
```

#### 2. local.conf 方式

直接在local.conf 修改，这边以giflib-git_5.1.4.bb  为例，强制使用 libunistring 为 external source code。

示例：

首先下载libunistring-0.9.9.tar.xz于3rd-pgks目录。 解压缩后，把giflib-5-1.4目录里的COPYING复制至libunistring-0.9.9。 主要是因为giflib-git_5.1.4.bb的LIC_FILES_CHKSUM有针对性COPYING进行验证

```bash
cp -fa giflib-5.1.4/COPYING libunistring-0.9.9  
```

接着在 conf/local.conf 最底下 加入 **INHERIT** 和 **EXTERNALSRC_pn-recipe-name**.

这边临时使用giflib-git作为示范。 凸显基于外部的libunistring程序代码进行编译。

```bash
INHERIT += "externalsrc"
EXTERNALSRC_pn-giflib-git = "/work/baseos/build/3rd-pkgs/libunistring-0.9.9"
```

进行编译，可以看到从giflib的程序转为libunistring的结果

```bash
#编译giflib-git，但实际是编译libunistring
bitbake giflib-git
#看到giflib-git内容变成libunistring
ls tmp/work/i586/giflib-git/5.1.4-r0/image/usr/lib/
libunistring.so  libunistring.so.2  libunistring.so.2.1.0
```

### 总结

上述两种方式都可以使用本机上的 source code 为来源，但是这两种还是有些差异。

- git fetcher：只适合对源代码进行编译，不适合重复修改。 原因是每当您修改源代码，则必须提交。 再接着修改配方的SRCREV参数
- exteranl src：只适合对源代码进行修改与编译。 而在使用中有遇到的问题：
  - 有些地方无法与yocto工具进行搭配使用。 例如 透过devtool产生patch之类。
  - 有些食谱可以替换外部源代码。 使用其中一个方法不通时候，需要尝试另一个方式。

### Reference

1. [Yocto Project Development Tasks Manual: Building Software from an External Source](https://www.yoctoproject.org/docs/latest/dev-manual/dev-manual.html#building-software-from-an-external-source)
2. [externalsrc-example_0.1.bb](https://github.com/beck-ipc/meta-at-chip/blob/master/recipes-examples/externalsrc-example/externalsrc-example_0.1.bb)
3. [BitBake User Manual: Fetchers](https://www.yoctoproject.org/docs/latest/bitbake-user-manual/bitbake-user-manual.html#bb-fetchers)
4. [Yocto Project Reference Manual : Var-S](ttps://www.yoctoproject.org/docs/2.4.2/ref-manual/ref-manual.html#var-S)

























