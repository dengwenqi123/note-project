## Yocto 基本概念

这篇文章主要记录，yocto 基本概念、构建的运作流程和bitbake 指令用法。

## 名词解释

- BitBake: **The task executor and scheduler** used by the OpenEmbedded build system to build images. For more information on BitBake, see the BitBake User Manual.
- Task: **A unit of execution** for BitBake (e.g. do_compile, do_fetch, do_patch, and so forth).
- Recipe: A set of instructions for building packages. A recipe describes where you get source code, which patches to apply, how to configure the source, how to compile it and so on. Recipes also describe dependencies for libraries or for other recipes. **Recipes represent the logical unit of execution**, the software to build, the images to build, and use the .bb file extension.

PS: recipe 在yocto 用.bb 文件表示，而 .bbappend 用于修改.bb 文件。

- Metadata: The files that BitBake parses when building an image. In general, **Metadata includes recipes, classes, and configuration files**. In the context of the kernel ("kernel Metadata"), the term refers to the kernel config fragments and features contained in the yocto-kernel-cache Git repository.

关系：metadata 包含许多recipes。recipes 则记录着哪些tasks 可以使用。recipe再根据规范或者需求运行tasks。所以bitable 运行的时，在适当的时机调用task 中的任务来完成工作。

eg: meta layer 目录底下有许多 recipe-bop, recipe-core,etc。 每个recipe-xxx 底下有许多的recipe文档，这些文档都可以运行所需的tasks,如 do_compile、do_fetch等。

### 执行流程

看一下官方文件，架构流程如下：

![](/Users/dengwenqi/Desktop/yocto-environment-ref.png)

1. 执行source oe-init-build-env 来设置环境. 此script 会产生一个build 目录. 底下的conf 目录存放user config(local.conf) 与layers (bblayers.conf). local.conf 可以设定编译的target( kernel, machine)以及加入相关套件image 等等之类的设定. bblayers.conf 则纪录所要使用的layers.
2. 执行bitbake core-image-minimal 也就是执行core-image-minimal recipe. 此recipe 拥有产生image 相关的tasks. 但是在执行之前, bitbake 根据bblayer.conf. 把所有layers 里面的recipes 都给读进来做分析.之后才正式执行core-image-minimal. 这部分如上方图里面的Parsing of 820 .bb files complete (0 cached, 820 parsed). 1277 targets, 44 skipped, 0 masked, 0 errors.
3. bitbake 执行target recipe(core-image-minimal) 时会寻找其依赖的recipes, 并执行其所拥有的tasks (do_fetch, do_compile, etc.). 这边recipe 所拥有的tasks 可参阅[2]第九章. 基本上的流程都是先抓取source codes, 接着configure 和编译. 把编译完的档案存放于${WORKDIR}. 然后把这些档案封包成package 的格式(rpm, deb, etc.) 存放于$ {WORKDIR}/deploy-xxxx. 同样也会存放于${DEPLOY_DIR}
4. do_compile task 以及 do_package task 完毕之后, 接着就是执行 image 的部分, bitbake 会执行 do_image 相关的部分, 来产生所需要的 image 档案. 相关档案会存放于 ${DEPLOY_DIR_IMAGE}

## OpenEmbedded 的元数据

Bitbake 完全不建立工作流程，**工作流程**和它的**配置**是被**元数据定义**。

### 元数据分为两类：

- 配置文件
- 配方(recipe)文件

### 配置文件

主要是由简单变量赋值的全局性设置，设置变量全局可访问，变量可以被覆盖。**配方**(recipe)可以设置和覆盖变量，但是在配方中所做的赋值只对配方局部有效。

bitable 有多种不同类型的配置文件。文件的扩展名以.conf结尾。

- bitbake 主配置文件，bitbake.conf 包含所有默认配置项设置。
- (layer)层配置文件：layer.conf 。包含针对这个层的配方文件的路径设置和文件匹配模式(pattern)。一般位于每个层的conf目录中。
- 构建环境层配置：bblayers.conf
- 构建环境配置: local.conf。 bblayers.conf 和local.conf这两个文件都位于构建环境的conf 目录中。belayer.conf 包含构建环境所需要用到的层，以及这些层的路径。local.conf包含针对当前构建环境的设置。
- 发行版配置：<distribution-name>.conf。 包含特定发行版的策略的变量设置。该文件一般位于定义发行版的层的conf/distro目录中。
- 机器配置<machine-name>.conf。包含特定机器的配置。文件路径在bsp层的conf/machine目录。

### 配方

bitbake的配方构成构建系统的核心。它们定义了软件包的工作流，包含用于bitbake构建过程步骤的具体命令。文件是以.bb结尾。配方中包含变量和以可执行元数据形式存在的构建指令。构建指令的实质是执行过程步骤的函数。

支持bbclass类(.bbclass)。支持使用bbappend追加文件(.bbapend)扩展。**.bbappend的文件名和被追加的配方文件名同名，而且相对于层的路径也要相同。**



## bitbake 常用命令

### bitbake [recipename/target recipe:do_task ...]

执行某个recipe(or target) 所有的tasks

```bash
bitbake weston
```

### bitbake [recipename/target recipe:do_task ...] -c [cmd]

```bash
bitbake weston -c listtasks
```

#### bitbake -s 

根据设定档, 列出所有 recipe 的版本资讯. 可以搭配 grep 来找是否存在某个 recipe 以及版本.

```bash
 bitbake -s | grep linux-yocto
```

### bitbake -e [recipename/target recipe:do_task ...]

列出此 recipe 相关的环境变数资讯. 此方式可以找出 ${S}, ${WORKDIR}, 等等的资讯.

```bash
bitbake -e weston | grep ^WORKDIR=
```







