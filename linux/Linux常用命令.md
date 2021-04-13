## Linux常用命令

#### 双中括号[[]]

1. [[是 bash 程序语言的关键字。并不是一个命令，[[ ]] 结构比[] 结构更加通用。在[[ 和 ]] 之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换。
2. 支持字符串的模式匹配，使用=~ 操作符时甚至支持shell的正则表达式。字符串比较时可以把右边的作为一个模式，而不仅仅是一个字符串，比如 [[ hello == hell? ]],结果为真。[[ ]]中匹配字符串或通配符，不需要引号。
3. 使用[[...]] 条件判断结构，而不是[....],能够防止脚本中的许多逻辑错误。比如 && || < 和 > 操作符能够正常存在于[[ ]]条件判断结构中。

```bash
if [[ $a != 1 && $a != 2 ]]
# 变量$file 中包含 FULL
if [[ $file =~ "FULL" ]]
```

#### openssh-server

功能：让远程主机可以通过网络访问sshd服务，开始一个安全shell

```bash
sudo apt install openssh-server
```

#### rsync

rsync (remote synchronize) 是一个远程数据同步工具，可以使用'rsync算法'同步本地和远程主机之间的文件。rsync的好处是只同步两个文件不同的部分，相同的部分不在传递。类似于增量备份，这使的在服务器传递备份文件或者同步文件，比起scp工具要省好多时间。

```bash
#1.首先开启 rsync
sudo vi /etc/default/rsync
RSYNC_ENABLE=true   #false改true
#2.修改配置文件
sudo cp /usr/share/doc/rsync/examples/rsyncd.conf /etc
rsync -avz testRsync mochangming@xxx:~/SSD/testRsync
```

#### ln 命令 功能是为文件在另外一个位置建立一个同步的链接

功能：为某一个文件或目录在另外一个位置建立一个同步的链接，类似Windows 下的超级链接。

链接分类： 软链接及硬链接

软链接：

1. 以路径的形式存在。类似于Windows 操作系统中的快捷方式
2. 软链接可以跨文件系统，硬链接不可以
3. 软链接可以对一个不存在的文件名进行链接
4. 软链接可以对目录进行链接

常用参数： -b 删除，覆盖以前建立的链接   -s 软链接(符号链接)  -v 显示详细处理过程

```bash
# sudo ln -s 源文件 目标文件
#举例：当前目录是/local，而我经常要访问/usr/local/linux/work那么我就可以使用在local下建立一个文件linkwork，然后sudo ln -s /usr/local/linux/work  /local/linkwork即建立两者之间的链接。
ln -sr hap/build jsenv/build
# -s 表示符号连接
# -r relateive 相对的
# 删除链接
rm -rf symbolic_name # 注意不是 rm -rf symbolic_name/
```



#### nc



#### grep

grep 强大的文本搜索命令，grep(Global Regular Exporession Print) 全局正则表达式搜索。grep在一个或多个文件中搜索字符串模版。

命令格式： grep [option] pattern file|dir

常用参数： -E 带参数  -i 忽略大小写

```bash
grep -r pattern files: 搜索子目录
grep -n pattern files: 显示匹配行及行号
# 查找当前目录及子目录下包含关键字 "NEON"的所有文件，并列出行号。	
grep -nr NEON ./

```

#### find 命令 用于在文件树中查找文件，并做出相应的处理。

命令格式： find pathname -options [-print -exec -ok ...]

命令参数：

-print: find命令将匹配的文件输出到标准输出。

-exec: find命令对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' { } \;，注意{ }和\；之间的空格。

-ok： 和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

- -name: 按照文件名找

- -user 按照文件属主来查找文件

- -mtime -n +n 按照文件的更改时间来查找文件， - n表示文件更改时间距现在n天以内，+ n表示文件更改时间距现在n天以前

- -type 查找某一类型的文件，

  ​	b - 块设备文件 d 目录  l 符号链接文件. f 普通文件

- -size n 查找文件长度为n块的文件

- -depth: 在查找文件时，查找深度

```bash
#在当前目录查询 文件中包含8080 的文件
find . -type f | xargs grep '8080'
# 在当前目录查找以.log 结尾的文件。 . 代表当前目录
find ./ -name '*.log'
#查找 /opt 目录下 权限为777的文件
find /opt -perm 777
#查找等于1000字符的文件
find -size +1000c
#在当前目录中查找更改时间在10日以前的文件并删除它们(无提醒）
find . -type f -mtime +10 -exec rm -rf {} \;
#查找当前目录下每个普通文件，然后使用 xargs 来判断文件类型
find . -type f -print | xargs file

```

#### xarg 命令的作用，是将标准输入转为命令行参数。

```bash
echo 'hello world' | xargs echo
hello world
```

####  > /dev/null

重定向到无底洞，不要输出到屏幕

分解这个组合: ">/dev/null 2>&1"为五部分。

1. `>`代表重定向到哪里，例如： echo "123" > /home/123.txt
2. /dev/null 代表空设备文件
3.  2> 代表stderr 标准错误
4. & 表示 **等同于**的意思， 2>&1 表示2的输出重定向 等同于 1
5. 1 表示stdout 标准输出，系统默认值是1， 所以 ">/dev/null" 等同于 "1>/dev/null"

#### 设置命令别名

```bash
vim ~/.bashrc
#添加
alias desk='ssh root@127.0.0.1'
```

#### ls 列举当前目录文件信息

ls  [选项].. [文件]..

```bash
ls -a #列出目录所有文件，包含.开始的隐藏文件
ls -A #列出除.以及..的其他文件
ls -r #反序排列
ls -t #以文件修改时间排序
ls -s #显示文件大小
ls -S #以文件大小排序
ls -h #以易读大小显示
```

#### crontab 命令

crontab -e 编辑定时任务表

crontab -r 删除

crontab -l 列出

```bash
#在 凌晨00:01运行
1 0 * * * /home/linrui/XXXX.sh
#每个工作日23:59都进行备份作业。
59 11 * * 1,2,3,4,5 /home/linrui/XXXX.sh   
59 11 * * 1-5 /home/linrui/XXXX.sh
#每分钟运行一次命令
*/1 * * * * /home/linrui/XXXX.sh
#每个月的1号 14:10 运行
10 14 1 * * /home/linrui/XXXX.s
```

#### awk

文本分析工具，空格作为默认的分隔符，$0表示所有域，$NF表示最后一列，S(NF-1)代表倒数第二列，NR代表行数
格式：awk '{pattern + action}' filename

































