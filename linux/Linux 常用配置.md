

# Linux 常用配置

#### 用户管理

```bash
#创建普通用户
useradd hadoop
#给用户设置密码
passwd hadoop
#创建组
groupadd monster
# 创建一个用户 fox, 并放入到 monster组中
useradd -g monster fox
# 修改文件所在的组
chgrp 组名 文件名
```

#### 给普通用户hadoop增加sudo 权限

```bash
vi /etc/sudoers

--------------下面为sudoers文件部门内容-----------
root	ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL
#只增加sudo 需要输入密码
hadoop  ALL=(ALL) ALL
#增加sudo 无密码
jenkins ALL=(ALL) NOPASSWD:ALL
```

#### 普通用户给文件或目录增加 权限

```bash
#修改权限 -chmod 通过chmod 指令，可以修改文件或目录的权限。
# 第一种方式 + 、-、 = 变更权限
# u: 所有者 、 g:所有组 o:其他人 a:所有人(u、g、o的总和)
chmod o+w 文件/目录名
chmod a-x 文件/目录名
```



#### 给普通用户(hadoop)文件 增加所有者权限

```bash
# 修改文件所有者
sudo chown -R hadoop:hadoop /opt/modules
```

#### Ubuntu 多台机器之间免密登录设置

```bash
#首先在一台机器上执行如下命令(一路回车)
ssh-keygen
#进行.ssh 目录
cd ~/.ssh
# 执行如下命令
ssh-copy-id jenkins@10.221.234.48
```

### Ubuntu 如何挂载磁盘

#### 查看系统硬盘分区情况

```bash
sudo fdisk -l
```

![这里写图片描述](https://img-blog.csdn.net/20180729205827673?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzaGl4aW5zaG91YWFh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可见设备 *sda2* 是博主要挂载的硬盘分区。

#### 查看硬盘的 UUID

```bash
sudo blkid
```

![这里写图片描述](https://img-blog.csdn.net/20180729210314734?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzaGl4aW5zaG91YWFh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 挂载配置

博主是将硬盘挂载到了 */home/haha/space* 下，新建文件夹：

```bash
sudo mkdir /home/haha/space
```

接下来编辑配置文件：

```bash
sudo vim /etc/fstab
jenkins@mi:~$ sudo cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/nvme0n1p2 during installation
UUID=b7e797b2-e846-42a4-a807-400b945b7254 /               ext4    errors=remount-ro 0       1
# /boot/efi was on /dev/nvme0n1p1 during installation
UUID=B054-9167  /boot/efi       vfat    umask=0077      0       1
#UUID=4824487a-c10d-4c5e-b4b6-7f71c33c6797 /home/data xfs defaults 0 1
UUID=4824487a-c10d-4c5e-b4b6-7f71c33c6797 /home/data ext4 defaults 0 1
/swapfile                                 none            swap    sw              0       0
```

![这里写图片描述](https://img-blog.csdn.net/20180729211004710?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dzaGl4aW5zaG91YWFh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

重启电脑即可完成挂载，查看一下。

[参考博客](https://blog.csdn.net/wshixinshouaaa/article/details/81275608)

#### 镜像挂载

普通镜像文件(.wic/vkx)文件，可以通过 mount 进行镜像中

```bash
export OFFSET=58720256
mkdir -p mnt
sudo mount -o loop,offset=$OFFSET $IMAGE ./mnt
sudo cp -r ./rootXx/usr/local ./mnt/usr/local
sudo umount ./mnt
```

### ssh 登录出现Are you sure you want to continue connecting (yes/no)?解决方法

1,可以使用ssh -o 的参数进行设置
例如: ssh -o StrictHostKeyChecking=no root@192.168.111.22