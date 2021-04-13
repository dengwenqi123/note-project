> xxxx.bb 文件是用来描述一个食谱(recipe)的。因为xxx.bb 文件会因为 patch codes、新增 tasks 等因素而修改。这篇文章简单介绍一下xxx.bbappend 文件，以不修改xxx.bb 文件的情况下扩充需求。本文会以 patch code 为例子，并以devtool 工具和手动的方式讲解。

### bbapend简介

.bbappend 是扩展.bb的方式，由于.bbapend 是用来新增tasks、patch code、新增/修改操作等等。其名称必须跟所要修改的**食谱名称相同**，而版本号则依据需求编写。

#### 示例

```bash
bb file名称： gitlib_5.1.4.bb
bbappend名称： gitlib_5.1.4.bbappend 或 gitlib_%.bbappend
% 表示任意版本
```

我认为，有如下情况可以使用.bbappend：

1. 当recipe 已经存在yocto 的基本layer之中，不应该随便修改。eg: u-boot_2017.09.bb 存在于yocto的meta layer
2. 某些layers 会共用此recipe的情况。eg: meta-raspberrypi 和 meta-ti 分别各自使用自己的 xserver-xf86-config_0.1.bbappend 来对 yocto meta layer 的 xserver-xf86-config_0.1.bb 做扩充修改.

### bbappend 使用示例

本示例以giflib作为测试库。先下载giflib-5.1.4.tar.bz2。接着使用 bitable-layer建立 meta-custom layer,在通过recipe tool 把产生的giflib_5.1.4.bb 存放在此layer.。

```bash
source mina/oe-init-build-env
# 自动进入build目录后
mkdir 3rd-pkgs; cd 3rd-pkgs/
 wget https://downloads.sourceforge.net/giflib/giflib-5.1.4.tar.bz2
cd /work/baseos/build
# 建立 meta-custom,并加入yocto 环境
bitbake-layers create-layer meta-custom
bitbake-layers add-layer meta-custom
# 进入生成的 giflib 目录
cd meta-custom/recipes-example
rm -rf example/
mkdir giflib 
# 通过recipetool 产生 bb文件
cd /work/baseos/build
recipetool create -o 3rd-pkgs/meta-custom/recipes-example/giflib/giflib_5.1.4.bb 3rd-pkgs/giflib-5.1.4.tar.bz2
```

#### 以devtool 产生 .bbappend

新增meta-custom-devtool-patch layer 来存放 devtool 之后产生的bbappend。

```bash
bitbake-layers create-layer meta-custom-devtool-patch
bitbake-layers add-layer meta-custom-devtool-patch
# 建立 giflib 目录
mkdir meta-custom-devtool-patch/recipes-example/giflib
```

使用devtool 修改source code 会产生 workspace layer.并复制一份 giflib code 至 workspace/source。

这边针对 util/giffix.c 修改。在main 函数里面添加printf("devtool hello !!\n")；

```bash
# 进入build 目录
devtool modify giflib
NOTE: Tasks Summary: Attempted 93 tasks of which 90 didn't need to be rerun and all succeeded.
INFO: Source tree extracted to /work/baseos/build/workspace/sources/giflib
INFO: Recipe giflib now set up to build from /work/baseos/build/workspace/sources/giflib

###### 修改 /work/baseos/build/workspace/sources/giflib/util/giffix.c  #########
cd /work/baseos/build/workspace/sources/giflib && vim util/giffix.c

int main(int argc, char **argv)
{
    printf("hello !!! (devtool)\n");      ///< 新增此行
    int i, j, NumFiles, ExtCode, Row, Col, Width, Height, ErrorCode,
    DarkestColor = 0, ColorIntens = 10000;

```

修改完毕之后，至source code 目录下进行git add 和 git commit. 最后在devtool finish 即可产生bbappend.

```bash
git add util/giffix.c
 git config --global user.email "you@xiaomi.com"
 git config --global user.name "xxx"
git commit -m "use devtool to create bbappend"
devtool finish giflib meta-custom-devtool-patch
ls meta-custom-devtool-patch/recipes-example/giflib/*
meta-custom-devtool-patch/recipes-example/giflib/giflib_%.bbappend

meta-custom-devtool-patch/recipes-example/giflib/giflib:
0001-use-devtool-to-create-bbappend.patch
```

#### 手动编写 .bbappend

新增 meta-custom-manual-patch layer 來存放自己建立的 bbappend 和 patch.

因此在 recipes-example 底下建立 giflib 和 giflib/giflib. 其中 giflib/giflib 是 yocto 预设会搜寻的目录
可以修改 FILEPATHS(常用于 .bb) or FILESEXTRAPATHS(常用于 .bbappend) 来符合自己的需求.

```bash
bitbake-layers create-layer meta-custom-manual-patch
bitbake-layers add-layer meta-custom-manual-patch
mkdir -p meta-custom-manual-patch/recipes-example/giflib/giflib

```

这边在 3rd-pkgs 手动复制一份 giflib, 并于 util/giffix.c 中添加 printf("hellow !! (manual)\n");.
接着在两个 giflib 的目录下, 使用 diff 产生出 patch. 把 patch 放置 meta-custom-manual-patch layer

```bash
# 进入build 目录
cd 3rd-pkgs && tar xf giflib-5.1.4.tar.bz2
cp -fa giflib-5.1.4 giflib-5.1.4_modify
diff -Naur giflib-5.1.4/util/giffix.c giflib-5.1.4_modify/util/giffix.c  > 00_giflib_util_giffix.patch
# 把 patch 放置 meta-custom-manual-patch
cd ..
mv 3rd-pkgs/00_giflib_util_giffix.patch meta-custom-manual-patch/recipes-example/giflib/giflib/
```

**注意**: 使用 diff 记得在 source code 的上一层, 不然 yocto 用 pacth 标示文件路径的时候会找不到对应文件.
以下为 00_giflib_util_giffix.patch 里的文件路径

在 meta-custom-manual-patch 新增 giflib_5.1.4.bbappend. 填写以下参数, 便完成以手动建立 bbappend.

```bash
# 进入build 目录
# 建立 giflib_5.1.4.bbappend
vim meta-custom-manual-patch/recipes-example/giflib/giflib_5.1.4.bbappend

############# 以下为giflib_5.1.4.bbappend 內容################
# 告诉 bitbake 把 recipe 目录加入
FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}:"
# 告诉 patch 名称
SRC_URI_append = "\
    file://00_giflib_util_giffix.patch \
"
```

#### 验证

可在tmp/work/i586-poky-linux/giflib/5.1.4-r0 查看是否有相关的patch. 以及于tmp/work/i586-poky-linux/giflib/5.1.4-r0/giflib-5.1. 4/util/giffix.c 看source code 是否有被修改

```bash
ls tmp/work/i586-poky-linux/giflib/5.1.4-r0/
head -n 38 tmp/work/i586-poky-linux/giflib/5.1.4-r0/giflib-5.1.4/util/giffix.c  | tail -n 4
```













