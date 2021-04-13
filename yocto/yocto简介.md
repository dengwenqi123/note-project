## Yocto Project概述

Yocto 是一个开源协作项目，可帮助开发人员为嵌入式产生创建基于Linux的定制化系统，而不管硬件架构如何。该项目提供了一套灵活的工具和空间，全球的嵌入式开发人员可以共享技术，软件堆栈，配置和最佳实践，可用于为嵌入式设备创建定制的`Linux`映像。

### Yocto Project 组成开发元素

1. 一组集成工具，使嵌入式Linux成功运行，包括自动构建和测试工具，板级支持和许可证合规流程，以及基于Linux的自定义嵌入式操作系统的组件信息
2. 参考嵌入式发行版(称为Poky)
3. OpenEmbedded构建系统，与OpenEmbedded Project共同维护

`Yocto Project`，`Poky`和`OpenEmbedded`互相之间的关系如下图所示：

![image-20201228173736884](/Users/dengwenqi/Desktop/image-20201228173736884.png)

Poky 参考嵌入式操作系统，实际上是一个有效的构建示例，它将构建一个小型嵌入式操作系统，使用的是它内置的构建系统(BitBake构建引擎和OpenEmbedded-Core核心构建系统元数据)。

### 层(layer)模型-定制的关键

Yocto Project 有一个嵌入式和IoT Linux 创建的开发模型，它将其与其他简单的构建系统区分开来。它被称为层(layer)模型：本质上，层是组织进文件和目录结构的元数据(配置文件，菜谱等)集合。

使用不同的图层在逻辑上分隔构建中的信息。例如，您可以拥有BSP层，GUI层，发行版配置，中间件或应用程序。将整个构建放在一个层中会限制并使未来的定制和重用变得复杂。

### Yocto Project 术语

| 术语                  | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| 追加文件(append file) | 追加文件扩展现有的菜谱。BitBake逐字追加该文件内容到对应菜谱，追加文件使用bbapend后缀 |
| BitBake               | 作为`OpenEmbedded`构建系统中的构建引擎，`BitBake`是任务执行器和调度器。**类似于我们熟悉的`Make`构建引擎** |
| 板级支持包（`BSP`）   | 板级支持包中的二进制，代码和其他支持数据使一个特定的操作系统运行在特定的目标硬件系统上 |
| 类(class)             | 类提供封装和基本继承机制，是可以在多个菜谱中使用的元数据文件。BitBake 类文件使用bbclass后缀 |
| 配置文件（`conf`）    | 包含变量的全局定义，用户定义的变量和硬件配置信息的文件。     |
| 镜像                  | 用于加载到设备上的Linux 发行版的二进制形式。通常包括bootloader,内核和根文件系统 |
| 层(layer)             | 是组织进目录和文件的元数据(配置文件，菜谱等)集合。层准许您合并相关元数据以自定义构建和扩展，并隔离多个体系结构构建的信息 |
| 元数据（`metadata`）  | 元数据包括类，菜谱，配置文件和引用构建指令本身的其他信息，以及用于控制构建内容并影响构建方式的数据。`OpenEmbedded Core`是一组重要的经过验证的元数据 |
| OpenEmbedded Core     | `oe-core`是由基础菜谱，类和配置文件组成的元数据              |
| 菜谱（`recipe`）      | 最常见的元数据形式，**类似于我们熟悉的Makefile文件**。菜谱将包含用于构建包的设置和任务（指令）的列表，然后用于构建二进制映像。菜谱描述了获取源代码的位置以及要应用的补丁。菜谱描述了库或其他配方的依赖关系，以及配置和编译选项。菜谱使用`bb`文件后缀 |



一些很有用的Bitbake构建调试命令：

| 命令                                         | 描述                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| bitbake <recipe_name/target_name>            | 构建指定目标                                                 |
| bitbake <packet> -c <task_name>              | 执行指定的任务                                               |
| bitbake-layers show-layers                   | 显示图层                                                     |
| bitbake-layers show-recipes “<package_name>” | 显示指定包对应的食谱                                         |
| bitbake -e \| grep -w DISTRO_FEATURES        | 输出BitBake所分析到的元数据文件、变量、功能等等，然后从中查找DISTRO_FEATURES变量值 |
| bitbake -e \| grep -w MACHINEOVERRIDES       | 输出BitBake所分析到的元数据文件、变量、功能等等，然后从中查找MACHINEOVERRIDES变量值 |
| bitbake -f <target>                          | 强制执行操作，即使不需要                                     |
| bitbake -v <target>                          | 显示详细输出                                                 |
| bitbake -g <target>                          | 以graphviz格式输出软件包之间的依赖关系树，最终会在构建目录下生成下面连个DOT语言描述文件:task-depends.dot : 描述任务级之间的依赖， recipe-depends.dot : 包与包之间的依赖 |
| bitbake -g -u taskexp <target>               | 显示构建目标的包依赖关系，此命令将打开一个UI窗口，因此它必须在主机（虚拟或本机）内部的控制台上执行 |
| bitbake -DDD <target>                        | 显示更多调试信息                                             |
| bitbake <package> -c devshell                | 打开一个新的shell，其中已为软件包定义了必要的系统值          |
| bitbake <package> -c listtasks               | 列出软件包的所有任务                                         |
| bitbake virtual/kernel -c menuconfig         | 交互式内核配置                                               |
| bitbake <image> -c fetchall                  | 获取特定镜像的源码                                           |
| bitbake -s \| grep <pkg>                     | 检查当前Yocto设置中是否存在某些软件包，并且显示对应食谱版本信息 |
| bitbake -s                                   | --show-versions   Show current and preferred versions of all recipes. |
| bitbake -c listtasks recipe_name             | 查看某个recipe 提供哪些tasks.                                |
| bitbake -c your-task recipe-name             | 只运行 recipe 中的某个 task                                  |
| Yocto 查找软件包的源地址                     | Bitable -e 软件包名称 \| grep ^SRC_URI                       |





## BitBake 介绍

BitBake Recipes，由文件扩展名.bb表示，是最基本的元数据文件。这些菜谱文件为BitBake提供一下内容：

- 有关包的描述信息(作者，主页，许可证等)
- 菜谱的版本
- 现有依赖项(构建和运行时依赖项)
- 源代码所在的位置以及如何获取它
- 源代码是否需要任何补丁，在何处找到它们以及如何应用它们
- 如何配置和编译源代码
- 在目标机器上安装包或创建的包的位置

### 追加文件

附加文件，这些文件具有 .bbappend文件扩展名，扩展或覆盖现有配方文件中的信息。

BitBake希望每个附加文件都有一个相应的配方文件。此外，附加文件和相应的配方文件必须使用相同的根文件名。文件名只能在使用的文件类型后缀(例如：formfactor_0.0.bb`和 `formfactor_0.0.bbappend)中有所不同。

命名附加文件时，可以使用“%”通配符来匹配配方名称。例如：假设你在一个名为如下的追加文件：

```bash
busybox_1.21%.bbappend
```

该追加文件将匹配任何版本的配方。因此，append文件将匹配以下配方名称：busybox_1.21.x.bb

```bash
busybox_1.21.1.bb
busybox_1.21.2.bb
busybox_1.21.3.bb
```

## 依赖图

在之前的章节，我们看到了程序包如何使用其配方中的DEPENDS 和 RDEPENDS 变量声明对其他程序包的直接构建时和运行时依赖性。能够很容易的知道这种做法，会导致冗长而复杂的依赖关系链。

当然，BitBake 必须能够解析这些依赖关系链，以正确的顺序构建软件包。使用其依赖项解析器，BitBake还可以创建依赖关系图以进行分析和调试。

BitBake 使用DOT 纯文本图形描述语言创建依赖关系图。DOT提供了一种简单的方式来描述具有节点和边的无向图和有向图，其语言和语言可以被人类和计算机程序读取。

Graphviz1软件包中的程序以及许多其他程序都可以读取DOT文件并将其呈现为图形表示形式。

给目标创建依赖图，调用BitBake -g 或 --graphviz选项。

ask-depends.dot  任务之间的依赖关系

package-depends.dot 运行时的目标依赖

pn-depends.dot 构建时的依赖

```bash
bitbake -g <target>
```

给目标创建如下依赖文件：

- pn-buildlist: 该文件不是DOT文件，而是包含从目标以相反顺序构建的软件包列表。
- pn-depends.dot: 在有向图中包含程序包依赖关系，该有向图首先声明节点，然后声明边。
- package-depends.dot：本质上与pn-depends.dot相同，但是在节点之后声明节点的边缘。该文件可能更易于人阅读，因为它会将以节点结尾的边缘与该节点组合在一起。
- Task-depends.dot: 声明任务级别上的依赖关系。

BitBake还提供了内置的用户界面，用于程序包的依赖关系，依赖浏览器。 您可以使用以下方式启动依赖项浏览器

```bash
bitbake -g -u taskexp <target>
```

依赖关系浏览器使你可以在图形用户界面中分析构建时和运行时依赖关系以及反向依赖关系。

#### Yocto 中删除文件系统镜像中的软件工具

在build/conf/local.conf中增加或更改以下命令来删除软件工具

```bash
PACKAGE_EXECLUDE = "package_name pakcage_name ..."
```

### Package vs Recipe

recipe包含构建系统用来创建软件包(package)的指令。recipe和package是前端与构建过程结果之间的区别。[参考](https://docs.yoctoproject.org/what-i-wish-id-known.html)

如前所述，构建系统采用recipe并根据配方(recipe)说明构建软件包(package)。产生的软件包与构建的配方有关，

### Yocto Linux 模块编译目录在哪？

我们可以通过如下命令查看，某个特定模块编译目录：

```bash
bitbake -e nodejs | grep ^S=
```

### Yocto 如何确定一个包的名字

使用如下命令来确定:

```bash
bitbake -s | grep XXX
```

其中XXX为包的关键字，例如linux或者boot,这样就可以看到所有带有关键字的包了。



### bitbake调试

```bash
$ bitbake -DDD <target>
```

### Events

```bash
addhandler myclass_eventhandler
python myclass_eventhandler() {
         from bb.event import getName
         from bb import data
         print("The name of the Event is %s" % getName(e))
         print("The file we run for is %s" % data.getVar('FILE', e.data, True))
     }
```

bitBkae 准许在配方和类文件中安装事件处理程序。在操作过程中的某些点会触发事件，例如针对给定的.bb的操作开始，给定任务开始、任务失败、任务成功等。主要目的是使构建失败时能像电子邮件一样通知用户构建失败。

```bash
addtask fetch
addtask unpack after do_fetch
addtask configure after do_patch
addtask compile after do_configure
addtask install after do_compile
addtask build after do_populate_sysroot
addtask cleansstate after do_clean
addtask cleanall after do_cleansstate

fetch unpack 
patch configure compile install 
populate_sysroot build
clean cleansstate cleanall

```



![image-20210308215525069](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210308215525069.png)





### recipetool









































