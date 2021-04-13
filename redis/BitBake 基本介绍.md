## BitBake 基本介绍

基本上，BitBake是一个Python程序，它由用户创建的配置驱动，可以为用户指定的目标执行用户创建的任务，即所谓的配方（recipes）。BitBake 是一个python 写的程序(工具)，它是一个多任务引擎，可以并行执行shell和python任务，每个任务单元根据预定义的元数据来管理源码、配置、编译、打包，并最终将每个任务生成的文件集合成为系统镜像。

#### Config、tasks与recipes

通过一种 BitBake 领域特定语言写 Config、tasks 与 recipes，这种语言包含变量与可执行的 shell、python 代码。所以理论上，BitBake 可以执行代码，你也可以用 BitBake 做除构建软件之外的事情，但是并不推荐这么做。

BitBake 是一种构建软件的工具，因此有一些特殊的功能，比如可以定义依赖关系。BitBake 可以解决依赖关系，并将其任务以正确顺序运行。此外，构建软件包通常包含相同或相似的任务。比如常见的任务：下载源代码包，解压源代码，跑 configure，跑 make，或简单的输出 log。Bitbake 提供一种机制，可通过一种可配置的方式，抽象、封装和重用这个功能。

#### 安装BitBake

BitBake 可以从这里下载：https://github.com/openembedded/bitbake。选择一个版本的分支，并下载 zip。

配置环境变量：

```shell
export PATH=/path/to/bbtutor/bitbake/bin:$PATH
export PYTHONPATH=/path/to/bbtutor/bitbake/lib:$PYTHONPATH
```

### BitBake 工程布局

BitBake 工程是通过layers目录与一个build 目录组织的，layer目录中包含配置文件和meta-data。

#### Layer目录

Layer目录包含配置、任务和目标描述。通常用 meta-xxxx 命令 Layer目录。

- classes  class文件扩展名为`.bbclass`
- recipes: bitBake recipes只的是扩展名为`.bb`的文件。
- conf 配置文件，

#### Build 目录

Build目录是bitbake命令被执行的地方。

工程目录结构如下：

```shell
.
├── build
│   ├── conf
│   │   └── bblayers.conf
│   ├── local.conf
│   └── tmp
├── meta-tutorial
│   ├── classes
│   │   ├── base.bbclass
│   │   └── mybuild.bbclass
│   ├── conf
│   │   ├── bitbake.conf
│   │   └── layer.conf
│   └── recipes-tutorial
│       ├── first
│       │   └── first_0.1.bb
└── meta-two
    ├── classes
    │   └── confbuild.bbclass
    ├── conf
    │   └── layer.conf
    └── recipes-base
        ├── first
        │   └── first_0.1.bbappend

```

### 模块说明：

#### build 目录

build 是我们烹饪嵌入式系统的地方（可以想象成餐馆的大厨房），整个构建过程就是在 build 目录的 tmp 子目录完成的。build 目录的 conf 子目录里是用户的配置文件 local.conf。

#### classes 目录

这个目录放的是配方（recipes）的类文件（后缀为 .bbclass）。类文件包含了一些 bitbake 任务的定义，例如怎么配置、怎么安装。配方文件继承类文件，也就继承了这些任务的定义。例如：我们如果增加一个使用 autotool 的软件包，只要在配方文件中继承 autotools.bbclass：

```shell
inherit autotools
```

#### 层中 conf 目录

conf 目录包含编译系统的配置文件（后缀为 .conf）。bitbake 在启动时会执行 bitbake.conf，bitbake.conf 会装载用户提供的 local.conf，然后根据用户在 local.conf 中定义的硬件平台（MACHINE）和发布目标（DISTRO）装载 machine 子目录和 distro 子目录的配置文件。machine 子目录里是与硬件平台相关的配置文件，distro 子目录里是与发布目标相关的配置文件。配置文件负责设置 bitbake 内部使用的环境变量，这些变量会影响整个构建过程。



### BitBake 工作流程

运行BitBake的主要目的是生成一个东西，例如 安装包、内核、链接库、或者一个完整的Linux系统启动镜像（包括bootloader、kernel、根文件系统）。当然，你也可以通过使用bitbake命令的某些参数，只执行生成过程中的某个步骤，例如编译、获取或清除数据、或者只返回编译环境的信息。

**下面简单说一下使用BitBake生成系统镜像的执行过程。**

首先，BitBake会先搜索当前工作目录下的conf/bblayers.conf文件。该文件包含一个BBLAYERS变量，它会列出所有项目所需的layer的目录。在BBLayers 所列出的layer目录中，都会有一个conf/layer.conf文件，在这个文件中会有一个LAYERDIR 变量，它记录了该 layer 的完整路径。这些 layer.conf 文件会自动构建一些关键的变量，例如 BBPATH 和 BBFILES 。BBPATH 记录了 conf 和 classes 目录下的 configuration 和 classes 文件的位置，BBFILES 则用于定位 .bb 和 .bbappdend 文件。如果找不到 bblayers.conf 文件，BitBake 会认为用户已经在环境变量中设置了 BBPATH 和 BBFILES 。

其次，BitBake 会在 BBPATH 记录的位置中寻找 conf/bitbake.conf 文件。

### BitBake基本演示

[参考](https://a4z.gitlab.io/docs/BitBake/guide.html)

























