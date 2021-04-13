### 查看用户信息

| 序号 | 命令       | 作用                       |
| ---- | ---------- | -------------------------- |
| 01   | id[用户名] | 查看用户UID和GID信息       |
| 02   | who        | 查看当前所有登录的用户列表 |
| 03   | Whom       | 查看当前登录用户的账户名   |

#### passwd 文件

`/et/passwd` 文件存放的是用户的信息，由6个分号组成的7个信息，分别是：

1. 用户名
2. 密码(x 表示加密密码)
3. UID(用户组标识)
4. GID(组标识)
5. 用户全名或本地账号
6. 家目录
7. 登录使用的Shell.就是登录之后，使用的终端命令，`ubuntu`默认是`dash`







## systemd资料

系统启动和服务器守护进程管理器，负责在系统启动或运行时，激活系统资源，服务器进程和其它进程。

#### 核心概念 `Unit`

`Unit`表示不同类型的Systemd对象，通过配置文件进行标识和配置；文件中主要包含了系统服务，监听socket,保存的系统快照以及其它与init相关的信息

#### 配置文件：

- `/usr/lib/systemd/system` 每个服务最主要的启动脚本设置，类似于之前的/etc/init.d/
- /run/systemd/system:系统执行过程中所产生的服务脚本，比上面目录优先运行
- /etc/systemd/system: 管理员建立的执行脚本，类似于/etc/rc.d/rcN.d/Sxx 类似的功能，比上面目录优先运行

```bash
systemctl -t help
```

#### Unit 类型， #systemctl -t help

- Service unit 文件扩展名为service,用于定义系统服务
- Target unit:文件扩展名为 target,用于模拟实现运行级别
- Device unit: device 用于定义内核识别的设备
- Mount unit: mount 定义文件系统挂载点
- Socket unit: socket 用于标识进程间通信的socket文件，也可在系统启动时，延迟启动服务，实现按需启动
- Snapshot unit snapshot 管理系统快照
- Swap unit: swap 用于标识swap 设备
- Automount unit: automount, 文件系统的自动挂载点
- Path unit:

#### Service unit 文件格式

在unit文件中，以“#”开头的行后面的内容别认为是注释，相关布尔值，yes,on, true都是开启， 0 no off false 都是关闭，时间单位默认是秒，所以要用毫秒(ms)和 分钟(m)等显示说明

#### Service unit file 

- [Unit]:定义与 Unit 类型无关的通用选项，用于提供unit 的描述信息、unit行为及依赖关系等
- \[Service] 与特定类型相关的专用选项；此处为Service类型
- \[install] 定义由 "systemctl enable" 以及 "systemctl disable"命令在实现服务启用或禁用时用到的一些选项

#### Service unit file 文件通常由三部分组成

- [Unit] ： 定义与Unit类型无关的通用选项；用于提供unit的描述信息，unit行为及依赖关系等。
- [Service]：与特定类型相关的专用选项，此处为Service类型
- [Install]：定义由"systemctl enable"以及"systemctl disable"命令实现服务启用或禁用时用到的一些选项

#### unit段的常用选项

- Description 描述信息
- After: 定义unit的启动次序，表示当前unit应该晚于哪些unit启动，其功能与Before相反
- Requires:依赖到的其他units,强依赖，被依赖的units 无法激活时，当前unit也无法激活
- Wants:依赖到的其它units,弱依赖
- Conflicts:定义units 间的冲突关系

#### Service段的常用选项

- EnvironmentFile: 环境配置文件
- ExecStart: 指明启动unit要运行命令或脚本的绝对路径
- ExecStartPre:ExecStart前运行
- ExecStartPost:ExecStart后运行
- ExecStop:指明停止unit要运行的命令或脚本
- Restart:当设定Restart=1时，则当次daemon服务意外终止后，会再次自动启动此服务
- Type:定义影响ExecStart以相关参数的功能的unit进行启动类型。

![image-20210321211335428](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210321211335428.png)

**注意**

对于新创建的unit文件，或者修改了的unit文件，要通知systemd 重载此配置文件，而后可以选择重启

```bash
systemctl daemon-reload
```

![image-20210321212228994](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210321212228994.png)

#### 查看当前系统所有服务

```bash
systemctl list-units --type service --all
```

![image-20210321212846182](/Users/dengwenqi/Library/Application Support/typora-user-images/image-20210321212846182.png)





### rc-local.service 设置开机自启动

Ubuntu 18.04 默认进程启动管理已经切换至systemd,不再使用sysV。如果想像之前一样使用 /etc/rc.local设置开机自启动，请如下设置：

#### 修改rc-local.service

/lib/systemd/system/rc-local.service新增：

```bash
[install]
WantedBy = multi-user.target
Alias=rc-local.service
```

#### 设置开机自启动rc-local

```bash
systemctl enable rc-local
```

#### 创建/etc/rc.local

```bash
cat > /etc/rc.local << EOF
#!/bin/bash
echo "test rc.local" > /tmp/rctest.log
EOF
```

Ubuntu 18 好像已经不支持rc.local 这个开机自动启动的脚本了，所以为了能继续用这个脚本，我们需要去编写一个ubuntu 18 下的启动脚本，通过这个脚本来启动我们的rc.local 脚本。

#### 创建一个rc-local.service

```bash
sudo vi /ect/systemd/system/rc-local.service
```

将如下内容写到rc-local.service中

```bash
[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local
 
[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99
 
[Install]
WantedBy=multi-user.target
```

#### 接下来再创建一个rc.local

```bash
sudo vi /ect/rc.local
```

内容如下：

```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
nohup /home/root/test.sh & # 启动脚本
exit 0
```

#### 给rc.local 执行权限

```bash
sudo chmod +x /ect/rc.local
```

#### 启动服务

```bash
sudo systemctl start rc-local.service
```

#### 测试服务的状态

```bash
sudo systemctl status rc-local.service
```



### Reference

[ubuntu 18实现本地的 rc.local 功能，开启自动启动](ubuntu18实现本地的 rc.local 功能，开机自动启动frpc)

http://www.jinbuguo.com/systemd/systemd.service.html























