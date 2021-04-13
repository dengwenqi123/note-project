## Yocto 分享

1. 如何加自己的包
2. 有项目之间依赖，如何加包
3. 编译C++ 程序
4. 增量编译
5. SSCache local-mirror
6. 如何加库

### 安装

### 基本使用



### 基本概念

- 讲解 重要文件 conf/local.conf、 conf/bitbake.conf 、conf/bblayers.conf

```bash
cat ./mina/meta/conf/bitbake.conf
```



### Yocto hellworld 添加一个软件包

https:*//github.com/tonyho/helloYocto.git*

```bash
devtool add hello-test https://github.com/tonyho/helloYocto.git
devtool add "npm://registry.npmjs.org;name=forever;version=0.15.1"

devtool build hello-test
devtool add helloyocto /home/dengwenqi/workspace/tvosSpace/helloYocto
```



### recipetool 

How do I change this config file ?

recipetool append file

```bash
recipetool edit hello-test
recipetool create https://github.com/tonyho/helloYocto.git
recipetool create -o zlib_1.2.8.bb http://zlib.net/zlib-1.2.8.tar.gz
```

why is this file on my board ?

1. Oe-pkgdata-util find-path /bin/sh
2. git grep
3. dnf whatprovides
4. dynamic install - scripts
5. IRC is your friend

![image-20210308220349217](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210308220349217.png)

### There's a script for that ...

Bitable-layers

- find/create/remove/layers

Oe-depends-dot 

- Examine output of "bitbake -g"
- 



To list bbappend files and recipe files they append to



```bash
bitbake-layers show-appends
# list currently configured layers and their priorities
bitbake-layers show-layers
bitbake-layers show-overlayed
#显示食谱所在图层
bitbake-layers show-recipes busybox
#增加图层
bitbake-layers add-layer ../sources/meta-openembedded/meta-filesystems
#
bitbake-layers remove-layer meta-filesystems
```

https://www.yoctoproject.org/docs/2.2.1/ref-manual/ref-manual.html#source-fetching-dev-environment

https://www.yoctoproject.org/docs/3.1.2/ref-manual/ref-manual.html#source-fetching-dev-environment

```bash
指定BB_STRICT_CHECKSUM = "0"在Yocto中禁用源代码的校验和检查？
```





### devtool - workspace

a separate environment(layer) in which to work on recipes, sources, patches

![image-20210309222600424](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210309222600424.png)



![image-20210309222842356](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210309222842356.png)





### Reference

1. [Yocto Project Development Tasks Manual ](http://www.yoctoproject.org/docs/current/dev-manual/dev-manual.html)
2. [BitBake User Manual](https://www.yoctoproject.org/docs/current/bitbake-user-manual/bitbake-user-manual.html)

