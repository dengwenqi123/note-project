## Bitbake基本使用

> Bitable 是一种类似于make 的构建工具，一个通用的任务执行引擎，它允许shell和python任务在处理复杂任务件依赖时高效、并行地运行。主要用于 OpenEmbedded 和Yocto工程构建Linux 发行版。本篇文章主要是介绍 bitbake 一些基本功能。

### 什么是bitbake 

bitbake 是一个python程序，它由用户创建的配置驱动，可以为用户指定目标执行用户创建的任务，即所谓的配方(recpie)。

#### config、 tasks 与 recipes

Bitable 是一种构建工具，可以定义依赖关系。同时bitbake 可以解决这些依赖关系，并将其任务以正确顺序运行。此外，构建软件包通常包含相同的任务：下载源代码、解压源代码、运行configure、make 和简单log输出。bitbake 提供一种机制，可以通过配置的方式，抽象、封装和重用这些功能。

### bitbake 使用示例

bitbake工程通过layers目录与一个build目录组织。layer目录包含配置、任务和目标描述。常用 meta-'something'命令layer目录。build目录是bitbake命令被执行的地方。在这里，bitbake期望能找到其初始配置文件，并将其生成的所有文件放在该目录。

最小工程目录结构如下：

```bash
bbTutorial/
├── build
│ ├── bitbake.lock
│ └── conf
│ └── bblayers.conf
└── meta-tutorial
├── classes
│ └── base.bbclass
└── conf
├── bitbake.conf
└── layer.conf
```

#### bitbake主要配置文件

bblayers.conf base.bbclass bitbake.conf layer.conf 

```bash
vim build/conf/bblayers.conf
BBPATH := "${TOPDIR}"
BBFILES ?= ""
BBLAYERS = "/path/to/meta-tutorial"
```

每个layer需要一个 conf/layer.conf 文件。

```bash
BBPATH .= ":${LAYERDIR}"
BBFILES += ""
```

**注意事项**：

- 添加当前目录到BBPATH,TOPDIR 被 BitBake 设置为当前工作目录。
- 初始设置BBFILES 变量为空，Recipes在后面会添加。
- 添加我们 meta-tutorial 的路径到 BBLAYERS 变量。当bitbake开始执行时，它会搜索所有给定的layer目录，以便获得其他配置。
- LAYERDIR 是 bitbake传给其所加载Layer的变量。我们添加该路径到BBPATH变量。
- BBFILES 告诉 bitbake recipe 在哪。

#### class/base.bbclass

一个*.bbclass 文件包含共享功能，我们的base.bbclass 包含一些我们log 函数，以及一个build 任务。

#### bitbake搜索路径

### bitbake示例





## Reference

[Bitbake User Manual](https://docs.yoctoproject.org/bitbake/)

https://www.yoctoproject.org/docs/3.1.2/bitbake-user-manual/bitbake-user-manual.html

































