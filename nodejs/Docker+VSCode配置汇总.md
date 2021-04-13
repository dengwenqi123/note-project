# Docker+VSCode配置汇总

## 序

之所以要写这么一篇介绍的文章，主要是因为作为一个打杂的算法工程师，在工作中需要使用远程服务器进行开发，然而远程服务器系统版更新缓慢，总有些代码跑不起来，让调包的我很难办。

另外工作中难免遇到共用开发机器的情况，如果把现在的算法工作称为炼丹，那服务器就是丹房，如果大家共用一个丹炉那总是会有配置不同的问题，因此最好的解决方式就是每个人用自己的丹炉。

基于以上原因，我整理了这么一篇配置，大体方案是基于 Docker + VSCode 配置属于个人的开发环境，还会涉及 VSCode 扩展等。

## 声明

首先要说明的是，由于个人才疏学浅，本文介绍的方案并不是理论上最优的方案，而且一些配置项也并不是必须或并不是适合所有人的，因此如果有同学想要参考的话建议首先完整阅读下各部分，有了整体的概念之后再根据自身需要参考配置。

其次本文各部分都参考了大量文档文章，我尽量给出了相关的参考链接，在此感谢原作者们。对于给出了具体参考链接的情况本文就不会对具体内容多做描述，还请看官查看相关参考文章。不论是本文还是相关参考资料都存在错误的可能，因此还需要阅读的各位根据自己遇到的情况进行调整，**官方文档永远比二手博客文章更靠谱**。

最后感谢各位（如果有）看官给本文贡献了点击量和浏览量…

## Docker

首先要说 Docker 是什么

> Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的 Linux或Windows 机器上，也可以实现虚拟化

以上是我抄的百科词条，总体意思看来是做了个内核容器虚拟化，也就是在真实系统的基础上又虚拟化了容器出来。类比的话可以类似于常用的虚拟机，但又不一样，容器开销上会小很多。这种虚拟容器适合用来制作可移植镜像发布应用，而我这里要用它来当做虚拟环境用，不知道算不算是一种浪费。

提到了Docker就要说到它的底层 LXC ，关于 LXC 和 Docker 的关系可以看看 《[请问docker与lxc是什么关系，有什么区别](https://www.zhihu.com/question/268288911)》。因为 LXC 是 Docker 的底层，所以也有人用 LXC 来达到共用机器隔离的目的，可以参考[《为实验室建立公用GPU服务器》](https://zhuanlan.zhihu.com/p/25710517)。不过本文还是介绍使用 Docker 的方法，就不多介绍 LXC 了。

知道了Docker 是什么了之后就要使用 Docker，需要说明的是最好参考 [docker-docs](https://link.zhihu.com/?target=https%3A//docs.docker.com/) （注：大家都看过廖雪峰老师的Python等系列博客吧，他在自己的博客里也表示“[docker就看官方文档](https://link.zhihu.com/?target=https%3A//www.liaoxuefeng.com/discuss/1012626196787840/1252876670204256)”），一些中文资料如《[菜鸟教程-Docker 教程](https://link.zhihu.com/?target=https%3A//www.runoob.com/docker/docker-tutorial.html)》因为更新不及时可能会存在些问题，这里我是踩过坑的。

## 安装 Docker

以 CentOS7 + 社区版Docker 为例，CentOS7 中 Docker 的安装可以参考《[Get Docker Engine - Community for CentOS](https://link.zhihu.com/?target=https%3A//docs.docker.com/install/linux/docker-ce/centos/)》

首先 Docker 对系统环境有些要求，参考文档中 《OS requirements 》部分，具体的说就是 CentOS7 以及 centos-extras repository 。

系统符合要求后可以卸载已有老版本 docker，参考 《Uninstall old versions》 部分，没装过的可以跳过了。

接下来的安装部分提供了三种安装方式，分别是常用的由repository安装、使用 RPM package 手动安装以及测试脚本安装，参考文档 《Install Docker Engine - Community》部分，以下简单介绍下常用的安装方式：

第一步 SET UP THE REPOSITORY：

- - 安装依赖包：`sudo yum install -y yum-utils device-mapper-persistent-data lvm2`
  - 设置稳定源：`sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`

第二步就是安装 docker：`sudo yum install docker-ce docker-ce-cli containerd.io`，安装后docker并未自动启动

第三步就是启动 docker：`sudo systemctl start docker`

最后可以测试下docker 是否安装成功：`sudo docker run hello-world`，如果能够正确显示一些信息则安装成功
安装完之后如果想卸载了可以参考 《Uninstall old versions》部分卸载 docker，但是卸载时候不会自动删除镜像和容器，需要记得手动删除。

## 安装NVIDIA-Docker

作为一个算法工程师，总归有用上 GPU 的时候，为了在 Linux 上启用 GPU 支持，需要安装 [nvidia-docke](https://link.zhihu.com/?target=https%3A//github.com/NVIDIA/nvidia-docker)。

> The NVIDIA Container Toolkit allows users to build and run GPU accelerated Docker containers.

要安装 nvidia-docker 首先要确保安装了 [NVIDIA driver](https://link.zhihu.com/?target=https%3A//github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions%23how-do-i-install-the-nvidia-driver) 和 Docker 19.03，其中驱动的链接已经给出了，没安装的自己安装下。

CentOS7下的 nvidia-docker 的安装可以参考《[CentOS 7 (docker-ce), RHEL 7.4/7.5 (docker-ce), Amazon Linux 1/2](https://link.zhihu.com/?target=https%3A//github.com/NVIDIA/nvidia-docker%23centos-7-docker-ce-rhel-7475-docker-ce-amazon-linux-12)》，使用说明可以参考[《Usage》](https://link.zhihu.com/?target=https%3A//github.com/NVIDIA/nvidia-docker%23usage)。

需要注意的是在 Usage 里已经使用 `--gpus` 参数，而后续的 TensorFlows 的 docker 中文教程中还在使用已废弃 `--runtime=nvidia`参数，需要记得替换成`--gpus`。

安装好之后可以使用命令 `docker run --gpus all,capabilities=utility nvidia/cuda:9.0-base nvidia-smi` 测试下。

## Docker 的使用

docker的使用真的是参考 [docker-docs](https://link.zhihu.com/?target=https%3A//docs.docker.com/) 就好了，这里只说几个和后文有关及常用的。

首先是一些查看镜像（image）和容器（container）的命令：

- 列出本机的所有 image 文件：`docker image ls`
- 删除 image 文件：`docker image rm [imageName]`
- 列出本机正在运行的容器：`docker container ls -l`
- 列出本机所有容器，包括终止运行的容器：`docker container ls -l --all`
- 删除容器文件：`docker container rm [containerID]`
- 导出容器：`docker export [OPTIONS] CONTAINER`
- 导入容器：`docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]`

这里镜像可以简单粗暴的理解为系统安装包，而容器可以理解为虚拟系统。

容器是可以导入和导出的，也就是说配置好了一个自己满意的容器后可以导出，之后每次需要干净的新系统环境之后都可以导入这个，就有一个符合自己需要的干净系统了。

docker 的启动使用 `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]` 命令，例如：

```
docker run -it -p 8022:22 --ipc host --name docker_example --gpus all -v ~/work:/work tensorflow/tensorflow:latest-gpu-py3-jupyter /bin/bash
```

- -i 表示 Allocate a pseudo-tty
- -t 表示 Keep STDIN open even if not attached
- -p 表示对端口号进行映射，即将 docker 容器的 22 号端口映射到宿主机的 8022 端口，这样设置的目的是方便后续使用 VSCode 连接容器，可以根据需要进行设置
- -ipc host 的目的是为了增加主机与容器共享内存用的，如果这个参数报错，还可以采用--shm-size参数[1]
- --name docker_example 是将容器命名为 docker_example，docker 有长id、短id、name三个标识，如果不指定名称则会随机名称
- --gpus all 是使用全部宿主机 GPU，这里的设置可以参考 nvidia-docker 的 Usage 具体设置使用哪个卡
- -v ~/work:/work 是将宿主机的 ~/work 目录映射到容器的 /work 目录，方便主机和宿主机间共享数据，这里的 -v 是 volumes 的意思，而这个共享是有两种情况的，具体信息可以参考《 [Use volumes](https://link.zhihu.com/?target=https%3A//docs.docker.com/storage/volumes/) 》
- tensorflow/tensorflow:latest-gpu-py3-jupyter 是指定使用的镜像版本，这里的版本可以在 [docker-hub](https://link.zhihu.com/?target=https%3A//hub.docker.com/) 查到

docker 的镜像为了减小体积会精简掉很多内容，比如刚刚这个机遇 Ubuntu 的 tensorflow/tensorflow 就没有 vim、locate、man手册等等，有需要的可以自行安装。

docker 容器启动后可以使用exit退出，而如果使用 -d 模式的话会将 docker 放在后台，此时可以通过 docker [attach](https://link.zhihu.com/?target=https%3A//docs.docker.com/engine/reference/commandline/attach/) command 进入容器。

docker 容器关闭后，可以使用 `Usage: docker start [OPTIONS] CONTAINER [CONTAINER...]` 命令启动容器，此时不能再用 docker run。

nvidia-docker 的安装使用说明中还出现了 -rm 参数，此参数默认则为 false，当使用此参数时会在退出容器时自动清理容器。

如果在 docker 的使用时遇到了用户问题，还可以考虑在 docker run 的时候增加 -u 参数设置登录用户。

如果想修改已有容器的部分参数（如 -v 的目录映射），可以参考 《[docker-修改容器的挂载目录三种方式](https://link.zhihu.com/?target=https%3A//blog.csdn.net/zedelei/article/details/90208183)》进行修改。

需要注意 ：当 attach 到 docker 服务后，看到的输出是同样的，如果需要同时开启多个窗口可以考虑：

- 按照下文方法配置并启动 ssh 服务，这样就通过 xshell 等方式远程登录
- 使用 screen 或者 tmux 等终端复用工具，这两个工具以类似 session-windows-pane 的方式复用窗口
- 同时还可以通过 screen 或者 tmux 起到程序后台运行的作用，参考：[Linux 技巧：让进程在后台运行更可靠的几种方法](https://link.zhihu.com/?target=https%3A//www.ibm.com/developerworks/cn/linux/l-cn-nohup/index.html)

## TensorFlow Docker

以 TensorFlow 为例，参考 [TensorFlow Docker](https://link.zhihu.com/?target=https%3A//www.tensorflow.org/install/docker%3Fhl%3Dzh-cn)

先抄下介绍：

> [Docker](https://link.zhihu.com/?target=https%3A//docs.docker.com/install/) 使用容器创建虚拟环境，以便将 TensorFlow 安装与系统的其余部分隔离开来。TensorFlow 程序在此虚拟环境中运行，该环境能够与其主机共享资源（访问目录、使用 GPU、连接到互联网等）。系统会针对每个版本测试 [TensorFlow Docker](https://link.zhihu.com/?target=https%3A//hub.docker.com/r/tensorflow/tensorflow/) 映像。
> Docker 是在 Linux 上启用 TensorFlow [GPU 支持](https://link.zhihu.com/?target=https%3A//www.tensorflow.org/install/gpu%3Fhl%3Dzh-cn)的最简单方法，因为只需在主机上安装 [NVIDIA® GPU 驱动程序](https://link.zhihu.com/?target=https%3A//github.com/NVIDIA/nvidia-docker/wiki/Frequently-Asked-Questions%23how-do-i-install-the-nvidia-driver)（无需安装 NVIDIA® CUDA® 工具包）

可以看出 TensorFlow 官方还是很支持使用 docker 方法的，官方也会发布每个版本经测试的 TensorFlow Docker 镜像，而且 docker 是 Linux 上启用 TensorFlow GPU 支持的最简单方法，直接使用官方打好的镜像，避免我们这些渣渣到处去提“tensorflow gpu 如何配置”这种低级问题。

TensorFlow Docker 要求在本地主机上安装 Docker，如果需要在 Linux 上启用 GPU 支持，请安装 nvidia-docker

注意：要在没有 sudo 的情况下运行 docker 命令，请创建 docker 组并添加用户。有关详情，请参阅[针对 Linux 的安装后步骤](https://link.zhihu.com/?target=https%3A//docs.docker.com/install/linux/linux-postinstall/)。

是使用 TensorFlow Docker 首先要拉取官方镜像，镜像位于 [tensorflow/tensorflow](https://link.zhihu.com/?target=https%3A//hub.docker.com/r/tensorflow/tensorflow/)，是基于 Ubuntu 的，映像版本按照以下格式进行标记：

- latest ：TensorFlow CPU 二进制映像的最新版本，默认项
- nightly ：TensorFlow 映像的每夜版 （不稳定）
- version ：指定 TensorFlow 二进制映像的版本，例如：1.13.1
- devel ：TensorFlow master 开发环境的每夜版。包含 TensorFlow 源代码

每个基本标记都有用于添加或更改功能的变体：

- tag-gpu：支持 GPU 的指定标记版本
- tag-py3：支持 Python 3 的指定标记版本
- tag-jupyter：带有 Jupyter 的指定标记版本

例如 `tensorflow/tensorflow:latest-gpu-py3-jupyter` 表示最新版(latest）、带GPU支持(-gpu)、基于Python3(-py3)、含 jupyter （-jupyter)

需要注意的是并非所有版本都包含这些，例如 tensorflow 1.12.0 并没有 jupyter 项，具体镜像可以通过 [tensorflow/tensorflow/tags](https://link.zhihu.com/?target=https%3A//hub.docker.com/r/tensorflow/tensorflow/tags) 搜索查询

选定好版本后可以通过`docker pull [OPTIONS] NAME[:TAG|@DIGEST]` 拉取镜像到本地

- 例如 `docker pull tensorflow/tensorflow:latest-gpu-jupyter` 即为拉取基于py2的最新版TensorFlow，带 gpu 和 jupyter
- 实际使用 `docker run` 命令时如果本地没有对应镜像则会自动拉取

拉取镜像（或者不拉取也行）后，可以通过上文介绍过的 `docker run`命令启动容器

- 例如 `TensorFlow：docker run -it tensorflow/tensorflow bash`，其余容器名称、目录映射、端口映射等设置项可以参考上文
- 如果需要启用 gpu 支持需要使用`--gpus all` 命令，其中 all 为使用所有 gpu，具体设置参考 [《Usage》](https://link.zhihu.com/?target=https%3A//github.com/NVIDIA/nvidia-docker%23usage)

习惯使用 jupyter notebook 的同学可以参考官方设置启用 jupyter notebook，例如

```
docker run -it -p 8888:8888 tensorflow/tensorflow:latest-gpu-jupyter
```

但我不习惯使用 jupyter notebook，所以没有认真测试过是否好用。

## 容器系统配置

## 配置 SSH服务

此部分参考 [1] 以及 [2]，之所以配置 ssh 是为了能够在启动容器后可通过 ssh 连接，也是为了后续能够通过 VSCode 进行远程连接。

然而似乎在 docker 中不应该启用 ssh 服务，并且 VSCode 似乎也是可以直接通过 docker 容器远程连接[3]，因此此处只是介绍方法，具体如何使用还需自行判断。

```bash
apt update
apt install -y openssh-server
mkdir /var/run/sshd
echo 'root:passwd' | mypasswd
sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
echo "export VISIBLE=now" >> /etc/profile
service ssh restart
```

此处第 4 行设置了容器的 root 用户登录密码为 mypasswd，第 5 行修改了 ssh 的设置为可以通过密码登录。

需要注意的是，根据 [2] 的回复中的记录看，似乎第 5 行可能有问题，导致密码依旧无法登录，如有问题可以自行查阅 ssh 配置文档进行修改。

当然此处更推荐的方式是参考 [4] 中的方式生成 ssh-key，并将 ssh-key 加入容器系统的 `/root/.ssh/authorized_keys`，这样可以实现免密登录，减少了每次登录输入密码的麻烦。

还需要注意的是每次停用 docker 的时候 ssh 服务会关闭，再次使用时候需要再次启用 ssh 服务，这样也间接避免了容器内 ssh 服务在不使用时开启的问题。

## 配置 oh-my-zsh

Oh-My-Zsh 是一款基于 zsh 的命令行工具，可以说是一种生活方式，而我大概更多是因为莫名的觉得 oh-my-zsh看起来有点像orz（笑）。

不过也有很多人不习惯使用 zsh，而是更喜欢系统的 bash，因此此处只是介绍下 oh-my-zsh 的配置，使用与否自行决定。

oh-my-zsh 的官方网站是[[ohmyzsh](https://link.zhihu.com/?target=https%3A//ohmyz.sh/)]，使用文档可以参考 [wiki](https://link.zhihu.com/?target=https%3A//github.com/ohmyzsh/ohmyzsh/wiki)。

首先需要 [安装zsh](https://link.zhihu.com/?target=https%3A//github.com/ohmyzsh/ohmyzsh/wiki/Installing-ZSH)。并将 zsh 设置为默认shell，此处可能会设置失败，参考：[zsh default without chsh](https://link.zhihu.com/?target=https%3A//www.google.com/search%3Fq%3Dzsh%2Bdefault%2Bwithout%2Bchsh)

之后需要安装 ohmyzsh，需要注意的是必须已安装 curl 或者 wget 以及 git，安装方式由以下三种：

- `via curl：sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
- `via wget：sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
- `curl -Lo install.sh https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh && sh install.sh`

安装好 ohmyzsh 之后可以设置下主题和插件

插件比较值得推荐的是 [zsh-syntax-highlighting](https://link.zhihu.com/?target=https%3A//github.com/zsh-users/zsh-syntax-highlighting)，安装方式参考 [How to install](https://link.zhihu.com/?target=https%3A//github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md)，在 ohmyzsh 中可以直接通过命令安装

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

安装插件之后运行 `plugins=( [plugins...] zsh-syntax-highlighting)` 启动插件。

ohmyzsh的插件和主题及按键映射等配置项都在 ~/.zshrc 中，其中

- `ZSH_THEME="ys"`：设置主题，主题可以参考 [Themes](https://link.zhihu.com/?target=https%3A//github.com/ohmyzsh/ohmyzsh/wiki/Themes)
- 可以像 [oh my zsh 哪些主题比较好看、有特点？](https://www.zhihu.com/question/33277508) 中的回一样答使用独(丧)树(心)一(病)帜(狂)的'random'—— 使用 random 主题的 ohmyzsh 就像一盒巧克力，你永远不知道下一词打开会给你带来什么样的主题
- **需要注意的是部分主题需要使用 [Powerline fonts](https://link.zhihu.com/?target=https%3A//github.com/powerline/fonts) ，需要自行安装字体才能正确显示各种图标**
- `plugins=(git z zsh-syntax-highlighting)`：用来设置启用的插件，插件可以参考 [Plugins](https://link.zhihu.com/?target=https%3A//github.com/ohmyzsh/ohmyzsh/wiki/Plugins)
- 部分插件如 zsh-syntax-highlighting 需要单独安装，详细信息参考具体插件
- 小键盘按键映射问题：如果遇到小键盘数字键无法使用的问题可以参考 [Num Pad can't input #4916](https://link.zhihu.com/?target=https%3A//github.com/ohmyzsh/ohmyzsh/issues/4916) 设置按键映射

## 配置 VSCode

上文已经介绍了如何使用 docker 创建一个属于自己的丹炉，那么下一步就是使用这个丹炉炼丹了。习惯使用 jupyter notebook 的同学可以参考官方设置启用 jupyter notebook，或者参考[5]，但我没有具体测试过。接下来要参考 [3] 及 [4] 中方法介绍 VSCode 远程炼丹方法。

PS：以下介绍实际上并不基于 docker，而是在 docker 容器内启用 ssh 服务后的 VSCode 的通用方法。也就是说对于非 docker 情况，只要能用 ssh 连接则是通用的。

PPS：貌似这种用 ssh 连接 docker 容器的方式比较不符合 docker 理念，而 VSCode Remote 提供了直连容器的方法 [[Developing inside a Container](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/docs/remote/containers)]，但我目前还没研究明白…有精通此道的同学请指点下我，或者等我自行领悟后也有可能更新下此文。

## 为何选择 VSCode

VSCode 作为宇宙第一IDE—— Visual Studio 的兄弟，目标自然是宇宙第一编辑器（Vim、Emacs：嗯？）。

从多年前刚刚发布时不支持插件的版本开始，VSCode 的市场份额在逐步扩张。

关于 VSCode 能够成功的原因可以参考下：[Visual Studio Code 可以翻盘成功主要是因为什么？](https://www.zhihu.com/question/363365943)我这里说明下我选择 VSCode 的原因吧。

如果有人问写 Python 最好的工具是什么，那我一定毫不犹豫的说 Pycharm；而 C++ IDE 则必须是前文提到的 VS。但是很多时候我要面对的不仅仅是 Python，作为一个打杂的，C++、Python、Shell 都是常用的，在一些情况下 Java、Perl 也是要会一点的，偶尔做个 Demo 的话 Html 也是需要了解一二的，在更偶尔的情况下世界上最好的语言 PHP 也要能看能改的，面对这种复杂环境，我觉得还是 VSCode 这种能试用不同场景的编辑器比较符合我的需求。

还有就是 VSCode 还能远程连接服务器，头文件依赖库都是直接基于服务器内环境，跳转、查阅都方便。当然 Pycharm 和 VS 也提供远程开发——但那不是收费的嘛。

不过 VSCode 也存在它的局限性，毕竟有些时候临时使用线上机器或者他人机器，不可能在每台机器上都配置VSCode，这时还是要靠 Vim 的。总体而言很多开发工作是可以转移到 VSCode 进行的。

## VSCode 安装及配置

没啥好说的，看看 [官方网站](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/) 。而 VSCode 的常用介绍及配置介绍可以参考下 [6] 和 [7] 及其他网上能搜到的相关文章。这里说几个我觉得需要注意的地方

- VSCode 更新速度还是很快的，而一些博客的方案介绍是基于比较早期的版本。比如一些方案介绍的是使用 sftp 同步代码，但是现在 VSCode 已经支持远程开发，sftp 的方式变得意义不大。因此最好还是多看看官方文档，自行判断需求。
- VSCode 的设置是以 Json 方式写到 setting.json 中的，而且区分本地和远程的，可以根据实际需要为不同项目编写不同的设置
- 很多公司为了安全会采用跳板机/堡垒机的方式连接服务器，在需要进行远程调整时的 VSCode 配置可以尝试参考 [8]（因为我暂时不需要，因此没有认真进行测试）

## VSCode 访问 Docker 容器

VSCode 的 Remote Development 配置可以参考官方文旦 [[Remote development over SSH](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/remote-tutorials/ssh/getting-started)]，也可以参考 [4]，其中比较需要注意的地方就是 ssh-key 的生成，当然这里 ssh-key 的应用场景不仅仅是 VSCode。

在按照上文提到的方式在 docker 容器中启用 ssh 服务后，自然也可以通过 VSCode 的 Remote Development 以 SSH 方式访问 docker 容器，需要注意的一点是此时 VSCode 的 ssh 配置时要增加 Port 参数，例如

```bash
Host TestServer
  HostName ${ip}
  User root
  Port 8022
```

其中 ${ip} 为远程服务器 ip 地址， Port 则为之前在使用 'docker run' 命令时使用'-p'参数设置的转发端口。也就是说使用 8022 端口连接，而这个连接会转发到容器的 22 号端口（默认ssh端口）。因此如果在 'docker run' 时没指定端口转发的话，就不能以此方式连接容器。

## VSCode 扩展推荐

VSCode 的一个强大之处就在于丰富的扩展——特别是一些官方支持的扩展。前文提到的 Remote Development 就是微软官方提供的远程开发扩展集合。

网络上现在也有很多 VSCode 扩展的推荐，但普遍是前端偏多，毕竟 VSCode 实属前端神器。而以下会介绍一些我日常使用的扩展，我的目标是把 VSCode 打造成打杂神器。

VSCode 的扩展是有本地安装和需要远程安装两种情况的，本地安装的扩展是指在使用 VSCode Remote Development 时不需要在服务器再安装一遍的，而远程安装的是指需要在服务器再次安装的，其中远程安装的扩展也可以使用本地的全局配置。

## 强烈推荐

- [Code Spell Checker](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dstreetsidesoftware.code-spell-checker)：据说大多数代码 bug 是因为拼写错误，装个拼写检查能够减少拼写错误…然后发现自己定义的各种缩写命名也都被识别成错误拼写

- [GitLens](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Deamodio.gitlens)：Git 高级扩展，文件历史、行历史等等功能，用 Git 的都应该装一个，然而对 git 版本有要求，具体多少我忘了，反正 git 版本太低的是遗憾了

- [koroFileHeader](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DOBKoro1.korofileheader) / [vscode-fileheader](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dmikey.vscode-fileheader)：自动增加代码文件头部注释，可以根据需要配置下要添加的内容，还能自动修改文件最新修改时间

- [Path Autocomplete](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dionutvmi.path-autocomplete)：在写代码时候自动目录提示，通常写代码时候都会有个 src 目录还有个 data 目录，有时候在src时候需要指向 data 下的文件，用这个扩展可以避免手动一点点敲相对路径

- [Remote Development](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dms-vscode-remote.vscode-remote-extensionpack)：包括 Remote - SSH、Remote-Containers、Remote-WSL 三个组件，堪称神器

- - [Remote-SSH](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dms-vscode-remote.remote-ssh)：通过 SSH 方式连接服务器或虚机，上文介绍的方法即依赖于这个扩展
  - [Remote-Containers](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dms-vscode-remote.remote-containers)：连接 docker 容器进行开发，然而还没搞清具体用法…
  - [Remote-WSL](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dms-vscode-remote.remote-wsl)：配合 [Windows Subsystem for Linux (WSL)](https://link.zhihu.com/?target=https%3A//docs.microsoft.com/en-us/windows/wsl/about) 使用，然而我没用过这个…

- [Resource Monitor](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dmutantdino.resourcemonitor)：资源使用情况显示，请对系统资源占用有点 x 数

![img](https://pic3.zhimg.com/80/v2-3912281e0337265dac69354435fde842_1440w.png)

- [Vim](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dvscodevim.vim)：Vim 扩展，适合习惯使用 Vim 的同学，然而我还是喜欢鼠标…
- [Visual Studio IntelliCode](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DVisualStudioExptTeam.vscodeintellicode)：微软官方 IntelliCode 扩展，支持 Python，需要在线下载对应 server，貌似现在在推广这种使用模型进行提示的策略

### 主题图标

- [Community Material Theme](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DEquinusocio.vsc-community-material-theme)：质感主题，VSCode 的主题似乎没有 Atom 的漂亮，但是也是有挺多的，我比较喜欢这个
- [Material Icon Theme](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DPKief.material-icon-theme)：质感图标，VSCode 也是支持各种图标的

### 代码高亮美化

- [autoconf](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dmaelvalais.autoconf)：提供 Autoconf M4 和 Automake files 的语法高亮，因为平常接触的 C++ 项目中就有使用 Autoconf 的，因此使用了这个扩展
- [Beautify](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DHookyQR.beautify)： javascript, JSON, CSS, Sass, 和 HTML 代码格式化，下载量很高
- [Better Comments](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Daaron-bond.better-comments)：为代码注释中的 TODOs 等信息提供代码高亮
- [Bracket Pair Colorizer](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DCoenraadS.bracket-pair-colorizer)：为代码中的括号对提供不同颜色的高亮，嗯，可能有些同学会觉得太浮夸了，但对我这种经常数括号的人还是挺方便的 —— 这个现在**不推荐**，在Remote打开大一点的源码文件可能会导致卡死！
- [Log File Highlighter](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Demilast.LogFileHighlighter)：日志文件高亮，主要是针对 INFO、WARN、ERROR 高亮，方便查看日志文件
- [MATLAB](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DGimly81.matlab)：Matlab 脚本的高亮、代码片段、代码检查功能
- [Output Colorizer](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DIBM.output-colorizer)：为 output/debug/extensions 面版以及 .log 文件提供高亮，刚刚发现这个似乎和 Log File Highlighter 有些重叠
- [prototxt](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Ddmitry-korobchenko.prototxt)：prototxt 语法高亮，使用 caffe 时候用的… 然而现在都 tf/pytorch 了，caffe 也该功成身退了
- [XML](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dredhat.vscode-xml)：XML 文件高亮…等等等功能
- [YAML](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dredhat.vscode-yaml)： YAML 文件高亮…等等等功能

### C++ 语言扩展

- [C/C++](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dms-vscode.cpptools)：微软官方 C/C++ 扩展，支持 IntelliSense 和 debugging

- - 在远程开发时有些公用头文件会默认安装在系统目录，使用这个扩展能够方便的进行跳转及查询，感觉非常方便

- [CMake Tools](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dms-vscode.cmake-tools) ：微软官方 CMake 工具扩展，用于项目配置，当然如果像我这样只想稍微修改下别人的 CMake 文件那可以试试有高亮的 [CMake](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dtwxs.cmake)

- [vscode-cudacpp](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dkriegalex.vscode-cudacpp)：CUDA C++ 语法扩展，提供 cuda c++ 语法高亮、代码片段功能，深度学习高级玩家应该会用到，我就是用高亮看看代码

### Docker 扩展

- [Docker](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dms-azuretools.vscode-docker)：微软官方 Docker 扩展，支持建立、管理docker容器，Dockerfile 编写等功能，然而上文介绍的方式是基于 ssh 的，因此并未使用此扩展

### Java 语言扩展

- [Java Extension Pack](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dvscjava.vscode-java-pack)：微软官方 Java 扩展 Pack，可以通过这个扩展一键开关相关 Java 扩展
- [Language Support for Java™ by Red Hat](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dredhat.java)：Code Navigation、Auto Completion、Refactoring、Code Snippets
- [Debugger for Java](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dvscjava.vscode-java-debug)：微软官方 Java debug 扩展
- [Java Test Runner](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dvscjava.vscode-java-test)：Run & Debug JUnit/TestNG Test Cases
- [Maven for Java](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dvscjava.vscode-maven)
- [Java Dependency Viewer](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dvscjava.vscode-java-dependency)：View Java projects, referenced libraries, resource files, packages, classes, and class members

### Python 语言扩展

- [Python](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dms-python.python)：微软官方 Python 扩展，支持 Python2、3

- - 支持 Python 解释器的选择，同样支持 Python 虚拟环境，也就是说可以直接选择 Python 环境作为解释器

![img](https://pic1.zhimg.com/80/v2-1ef6c3ccea3f8cc600bd1ba22d205828_1440w.jpg)

- - 支持 IntelliSense, linting, debugging, code navigation, code formatting, Jupyter notebook, refactoring, variable explorer, test explorer, snippets 等功能，Python 程序员必备
  - IntelliSense 还可以考虑使用 [Kite](https://link.zhihu.com/?target=https%3A//kite.com/) 或者 [TabNine](https://link.zhihu.com/?target=https%3A//tabnine.com/) 什么的
  - linting 可以通过 `"python.formatting.provider": "autopep8"`, 设置，还支持 yapf 和 black
  - Jupyter notebook 的支持为使用 docker 远程炼丹提供了方便，效果类似于下图，具体参考[[Working with Jupyter Notebooks in Visual Studio Code](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/docs/python/jupyter-support)]

![img](https://pic2.zhimg.com/80/v2-dc23531a55b3710f0392aa0402005219_1440w.jpg)

- - debugging 可以直接调用远程 Python 环境进行 debug，图下图所示，最左边红箭头指的地方是 debug 面板，上方红色箭头指的地方是局部变量，右方红色箭头指的是设置断点的地方，具体参考 [[Python debug configurations in Visual Studio Code](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/docs/python/debugging)]

![img](https://pic4.zhimg.com/80/v2-84ab02fc15dbffa72e35c02e5d50599b_1440w.jpg)

- [python snippets](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dfrhtylcn.pythonsnippets)：一些 python 的代码片段
- [kite](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dkiteco.kite)：Kite 的 VSCode 扩展，AI自动补全工具，不过我还是用的 Visual Studio IntelliCode，类似的还有 TabNine

### Shell 脚本扩展

- [Bash IDE](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dmads-hartmann.bash-ide-vscode)：Shell 脚本跳转、补全等等，基于 [bash language server](https://link.zhihu.com/?target=https%3A//github.com/mads-hartmann/bash-language-server)，需要 nodejs，所以属于看需求使用
- [shellman](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DRemisa.shellman)：Bash Scripting Snippet，适合我这种 Shell 二把刀，还有配套教程电子书 [shellman ebook](https://link.zhihu.com/?target=https%3A//github.com/yousefvand/shellman-ebook)

### 版本管理扩展

- [SVN](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Djohnstoncode.svn-scm)：VSCode 的 SVN 扩展，可以查看下 svn 提交历史，在使用时可以直接查看已修改代码与原始代码区别，推荐用 svn 的同学装一个

### 其他扩展

- [Bookmarks](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dalefragnani.Bookmarks)：为 VSCode 提供书签功能，有书签面板，不过书签的位置和断点是重叠的，不是点击即添加还算有点遗憾
- [Chinese (Simplified)](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DMS-CEINTL.vscode-language-pack-zh-hans) ：简体中文官翻包，然而我没安装，因为受不了菜单中中英混杂的情况（该死的强迫症）
- [Markdown All in One](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dyzhang.markdown-all-in-one)：Markdown 扩展，基本上 Markdown 需要的功能都支持了，写个文档挺方便的
- [Partial Diff](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dryu1kn.partial-diff)：任意两段文本之间的 diff，在某些时候挺方便的其实
- [Project Manager](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dalefragnani.project-manager)：VSCode 项目管理，可以方便在项目中进行切换，然而目前还不支持远程项目…
- [Settings Sync](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DShan.code-settings-sync)：VSCode 尚未支持配置同步是个遗憾，这个扩展通过 GitHub Gist 对配置进行同步，算是曲线救国吧
- [Todo Tree](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DGruntfuggly.todo-tree)：能够吧代码中的 Todo、Fixme 之类的在边栏显示，可以参考[9]，值得推荐

![img](https://pic3.zhimg.com/80/v2-4c4e8733941af75f66e1b7e581be1596_1440w.jpg)

- - 这个扩展使用 [ripgrep](https://link.zhihu.com/?target=https%3A//github.com/BurntSushi/ripgrep) 进行关键字搜索，在遇到目录中有大文件时会占用大量资源，可以通过 `todo-tree.filtering.excludeGlobs`把数据文件排除

### 偶尔用一用的扩展

- [Ascii Tree Generator](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Daprilandjan.ascii-tree-generator)：生成类似下图的 Ascii Tree，我觉得会有用到的时候，然而并没有用过…

![img](https://pic2.zhimg.com/80/v2-8e3888209311a3e5e662026f58232d79_1440w.jpg)

- [Code Runner](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dformulahendry.code-runner)：国人写的扩展，参考 [10]，下载量和评价都很高，支持多种语言的运行配置，不过我没怎么用过…
- [Color Picker for VS Code](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Danseki.vscode-color)：方便在 VSCode 中取色调色，下载量不低，但是评分不高，我也不常用
- [Excel Viewer](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3DGrapeCity.gc-excelviewer)：CSV 文件的扩展，提供更好的查看模式
- [Rainbow CSV](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dmechatroner.rainbow-csv)：CSV 文件的扩展，分列高亮，偶尔需要查看 CSV 文件时候可以用一用
- [Readme Pattern](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dthomascsd.vscode-readme-pattern)：README.md 模板，有 Bot, Hackathon, Minimal, Standard 四种，写项目时候留个 README.md 是好习惯，然而不随代码更新的话就很坑了
- [SFTP](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Dliximomo.sftp)：VSCode 的 SFTP 的扩展，然而我更习惯 [WinSCP](https://link.zhihu.com/?target=https%3A//winscp.net/eng/docs/lang%3Achs)
- [Sort JSON objects](https://link.zhihu.com/?target=https%3A//marketplace.visualstudio.com/items%3FitemName%3Drichie5um2.vscode-sort-json)：把 Json 文件排序，我装这个扩展的目的就是把我的 VSCode 配置文件 sort 了一下（我这强迫症啊）

## 总结

以上用了 8000 来字介绍了一些 docker + vscode 的配置，然而大家可以我介绍的详细程度看出来：没错，我就是来吹 VSCode 的（笑）。

言归正传，其实除了我这里提到的 Docker 的方法外，其实 Python 的虚拟环境也是个很好的隔离选择，可以用来配置不同版本的 python 解释器。

最后祝大家都能配置好自己炼丹炉，不受别人的炉子干扰，也不干扰别人的炉子…

## 参考资料

1. [VSCode+Docker: 打造最舒适的深度学习环境](https://zhuanlan.zhihu.com/p/80099904)
2. [PyCharm+Docker：打造最舒适的深度学习炼丹炉](https://zhuanlan.zhihu.com/p/52827335)
3. [在 VS Code 中使用容器开发](https://zhuanlan.zhihu.com/p/88565144)
4. [使用vscode进行远程炼丹](https://zhuanlan.zhihu.com/p/89662757)
5. [如何构建包含TensorFlow/Python3/Jupyter的Docker？](https://zhuanlan.zhihu.com/p/65886733)
6. [万字长文把 VSCode 打造成 C++ 开发利器](https://zhuanlan.zhihu.com/p/96819625)
7. [如何让 VS Code 更好用10倍？这里有一份VS Code 新手指南](https://zhuanlan.zhihu.com/p/99462672)
8. [[VS Code\]如何通过跳板机连接服务器进行远程开发：Remote-SSH 篇](https://link.zhihu.com/?target=https%3A//blog.csdn.net/maokelong95/article/details/91801944)
9. [vscode 插件推荐 todo-tree](https://zhuanlan.zhihu.com/p/63303926)
10. [[VSCode插件推荐\] Code Runner: 代码一键运行，支持超过40种语言](https://zhuanlan.zhihu.com/p/54861567)