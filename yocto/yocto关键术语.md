##  yocto关键术语

- S: Full path to the directory where BitBake places the unpacked sources. By default, this is a subdirectory of the working directory for the package:  S = "${WORKDIR}/${P}"
- SRC_URI: Download URI for source packages.
- SRCREV:Source revision for use with downloads from SCM.
- DEPENDS: 配方构建时依赖项
- RDEPENDS：列出软件包的运行时依赖项（即其他软件包），必须安装这些依赖项才能使生成的软件包正确运行。如果在构建过程中找不到此列表中的软件包，则会出现构建错误。



## Yocto 依赖关系

动态库依赖。yocto在do_package时会保存每个包提供的.so文件等信息，在运行时，如果检测到包A链接了包B提供的.so文件，那么会自动把B添加到A的依赖中
pc文件依赖。yocto在构建时会使用pkgconfig生成包的*.pc文件，如果包A的*.pc文件中出现了Require:字样指向包B提供的，那么会自动把B添加到A的依赖中
根据1、2两条规则，如果A依赖了B，B依赖了C，那么A会自动添加C的依赖
在IMAGE_INSTALL中的包，如果其最后生成的文件中有脚本文件指定了是由python或者perl等解释器来运行，那么yocto会自动将该解释器的recipe添加到镜像的依赖中。

首先说明，yocto中的依赖本质上是任务之间的依赖，即使是使用DEPENDS或者RDEPENDS定义的两个recipe之间的依赖关系，但实际上在yocto运行时依赖关系还是会体现在这两个recipe中的task之间，即在运行时，yocto会将recipe之间的依赖解析成task之间的依赖。

task之间的依赖关系可以分为两种：属于同一个recipe的task之间的依赖或者属于不同recipe的task之间的依赖。而属于不同recipe的task之间的依赖又可分为构建时依赖或者运行时依赖，其中构建时依赖是指在yocto构建时某个包会依赖另一个包提供的头文件、so文件或者可执行文件来完成自身的编译过程；而运行时依赖是指某个包编译生成的可执行文件在板子上运行时，需要依赖另一个包提供的so文件等。yocto对待构建时依赖和运行时依赖的区别不大，主要是将构建时依赖和运行时依赖解析成task之间的依赖时有些不同，且yocto会自动生成一些运行时依赖关系（参考前一篇博客），另外运行时依赖关系指的是两个包（PACKAGES变量中的）之间，而不是两个recipe之间。

属于同一个recipe的task之间的依赖可以用addtask来设置
属于不同recipe的task之间的依赖可以用多种方式来定义、如depends、rdepends、deptask、rdeptask和recrdeptask等，下面详细介绍每种方式。
depends用于定义不同recipe之间的task之间的构建时依赖，如do_patch[depends] = "quilt-native:do_populate_sysroot"表示本recipe的do_patch任务依赖于quilt-native的do_populate_sysroot任务。
rdepends用于定义不同recipe之间的task之间的运行时依赖，使用方式和depends类似，都只是用于告诉yocto框架让某个任务在另一个任务完成之后再运行deptask也是用于不同recipe之间的task之间的构建时依赖，不过和depends不同的是deptask可以用于批量定义依赖，如do_configure[deptask] = "do_populate_sysroot"表示本recipe的do_configure任务需要在所有包含于DEPENDS变量中的其它recipe的do_populate_sysroot任务运行之后才能运行rdeptask也是用于批量定义依赖关系，如do_package_qa[rdeptask] = "do_packagedata"，与deptask不同的是，它表示本recipe的do_package_qa任务需要在所有包含于RDEPENDS变量中的其它recipe的do_packagedata任务运行之后才能运行recrdeptask同时包含了deptask和rdeptask的功能，并且递归地寻找依赖recipe。也就是说如果A依赖了B，B依赖了C，C依赖了D，那么在A中设置do_a[recrdeptask] = do_b表示A的a任务会在B、C、D的b任务都运行完毕之后再运行。











