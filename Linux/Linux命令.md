

## 线上查询及帮助命令

### man

man 命令名

查看命令帮助，命令的词典，更复杂的还有info，但不常用

### help

help 命令名

命令名 --help

查看Linux内置命令的帮助，比如cd命令 

## 文件和目录操作命令

### ls

> `ls(list)` 功能是列出目录的内容及其内容属性信息

`ls -l` 显示文件和目录的详细资料

`ls -a` 列出全部文件，包含隐藏文件

`ls -R` 连同子目录的内容一起列出（递归列出），等于该目录下的所有文件都会显示出来

### cd

> `cd(change directory)` 功能是从当前工作目录切换到指定的工作目录

`cd` 进入个人的主目录

`cd -` 返回上次所在的目录

### cp

`cp(copy)` 其功能为复制文件或目录

### find

> `find` 查找的意思，用于查找目录及目录下的文件

`-cmin n` 在过去 n 分钟内被修改过

`-cnewer file` 比文件 file 还要新的文件

`-mtime n` 在过去 n 天内被创建或内容被修改过的文件

> `atime(access time)` 文件被读取或者执行的时间
>
> `mtime(modify time)` 更改文件内容
>
> `ctime(change time)` 更改文件任何属性

`-empty` 空的文件

`-name name` 文件名称符合 name 的文件，iname 会忽略大小写

`-size [+,-]n[b,k,M,G]` 文件大小是大于/小于 n 单位

`-type c` 文件类型是 c 的文件

> b	区块装置文件
>
> c	字型装置文件
>
> d	目录
>
> f	一般文件
>
> l	符号连结
>
> p	具名贮列
>
> s	socket

```markdown
# 将目前目录及其子目录下所有延伸档名是 xml 的文件列出来
	find . -name *.xml
		find / xxx 表示从根目录查找
		find . xxx 表示从当前目录查找

# 将目前目录其其下子目录中所有目录列出
	find . -type d

# 将目前目录及其子目录下所有最近 1 天内更新过的文件列出
	find . -ctime -1
```

### mkdir

mkdir：全拼make directories，其功能是创建目录。

### mv

`mv(move)` 其功能是移动或重命名文件

### pwd

`pwd(print working directory)` 其功能是显示当前工作目录的绝对路径

### rename

rename：用于重命名文件。

### rm

> `rm(remove)` 其功能是删除一个或多个文件或目录

`-i` 删除前逐一询问确认

`-f` 即使原档案属性设为唯读，亦直接删除，无需逐一确认

`-r` 将目录及以下之档案亦逐一删除

```markdown
# 删除spring目录下的所有文件及目录
	rm  -rf  spring
```

### rmdir

rmdir：全拼remove empty directories，功能是删除空目录。

### file

file：显示文件的类型。 

## 查看文件及内容处理命令

`cat` 从第一个字节开始正向查看文件的内容，`-n` 可标示文件行数

`more` 分页显示文件内容

`less` 分页显示文件内容，more命令的相反用法

`head` 显示文件内容的头部，`-n` 表示前 n 行

`tail` 显示文件内容的尾部，`-n` 表示最后 n 行

> `tail -n +1000 file1` 从1000行开始显示，显示1000行以后的

```shell
# 单个关键词高亮显示
tail -f xxx.log | perl -pe 's/(关键词)/\e[1;颜色$1\e[0m/g'
# 多个关键词高亮显示
tail -f xxx.log | perl -pe 's/(关键词1)|(关键词2)/\e[1;颜色1$1\e[0m\e[1;颜色2$2\e[0m/g' 
30m：黑
31m：红
32m：绿
33m：黄
34m：蓝
35m：紫
36m：青
37m：白
# 显示1000行到3000行
cat filename | head -n 3000 | tail -n +1000
```

`grep` 分析一行的信息，若当中有我们所需要的信息，就将该行显示出来，该命令通常与管道命令一起使用，用于对一些命令的输出进行筛选加工等等

```shell
# 在文件 '/var/log/messages'中查找关键词"Aug"
grep Aug /var/log/messages
# 在文件 '/var/log/messages'中查找以"Aug"开始的词汇
grep ^Aug /var/log/messages
# 选择 '/var/log/messages' 文件中所有包含数字的行
grep [0-9] /var/log/messages
# 在目录 '/var/log' 及随后的目录中搜索字符串"Aug"
grep Aug -R /var/log/*
```

wc：统计文件的行数、单词数或字节数。

iconv：转换文件的编码格式。

vi/vim：命令行文本编辑器。

## 文件压缩及解压缩命令

### tar

``tar`` 命令：用来压缩和解压文件。tar本身不具有压缩功能。他是调用压缩功能实现的。 

参数： 

-c ：创建新的文档 

-x ：解压文件 

-t ：查看文件 特别注意，在参数的下达中， c/x/t 仅能存在一个，不可同时存在，因为不可能同时压缩与解压缩。

-z 通过 gzip 来进行归档压缩 

-j 通过 bzip2 来进行归档压缩 

-v 显示详细的tar处理的文件信息 

-r 增加文件，把要增加的文件追加在压缩文件的末尾 

-u 仅将较新的文件附加到存档中 

-d 比较存档与当前文件的不同之处 

-f ：要操作的文件名，在 f 之后要立即接文件名，例如使用『 tar -zcvfP tfile sfile』就是错误的写法，要写成『 tar -zcvPf tfile sfile』 

-p ：使用原文件的原来属性（属性不会依据使用者而变） 

-P ：可以使用绝对路径来压缩 

-N ：比后面接的日期(yyyy/mm/dd)还要新的才会被打包进新建的文件中 

-C --directory=DIR 解压文件至指定的目录 

–remove-files 压缩后删除原文件 

–exclude FILE：在压缩的过程中，不要将 FILE 打包

## 信息显示命令

uname：显示操作系统相关信息的命令。

hostname：显示或者设置当前系统的主机名。

dmesg：显示开机信息，用于诊断系统故障。

uptime：显示系统运行时间及负载。

stat：显示文件或文件系统的状态。

du：计算磁盘空间使用情况。

df：报告文件系统磁盘空间的使用情况。

top：实时显示系统资源使用情况。

free：查看系统内存。

- total 总内存
- used 已用内存
- free 空闲内存
- shared 被共享使用的物理内存
- buff/cache 已使用的缓存
- avaiable 可用内存
- available = free + buffer + cache(注：只是大概的计算方法)
- total = used + free + buff/cache

date：显示与设置系统时间。

cal：查看日历等时间信息。

## 用户管理命令

useradd：添加用户。

usermod：修改系统已经存在的用户属性。

userdel：删除用户。

groupadd：添加用户组。

passwd：修改用户密码。

id：查看用户的uid,gid及归属的用户组。

su：切换用户身份。

sudo：以另外一个用户身份(默认root用户)执行事先在sudoers文件允许的命令。

## 基础网络操作命令

ping：测试主机之间网络的连通性。

ifconfig：查看、配置、启用或禁用网络接口的命令。

netstat：查看网络状态。

## 系统权限及用户授权相关命令

chmod：改变文件或目录权限。

## 内置命令及其它

rpm：管理rpm包的命令。

yum：自动化简单化地管理rpm包的命令。

clear：清除屏幕，简称清屏。

chkconfig：管理Linux系统开机启动项。 

vmstat：虚拟内存统计。

mpstat：显示各个可用CPU的状态统计。

iostat：统计系统IO。

## 进程管理相关命令

kill：终止进程。

### ps

``ps`` 是LINUX下最常用的也是非常强大的进程查看命令

``ps [选项]``，下面对命令选项进行说明：

-e 显示所有进程

-f 全格式 

-h 不显示标题 

-l 长格式 

-w 宽输出 

-a 显示终端上的所有进程，包括其他用户的进程 

-r 只显示正在运行的进程 

-x 显示没有控制终端的进程

``中间的“ | ”``是管道命令 是指ps命令与grep同时执行

``grep(Global Regular Expression Print``，表示全局正则表达式版本)命令是查找，是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。它的使用权限是所有用户。

```markdown
# 查找名字为xxx的线程信息
	ps -ef | grep xxx
```

### service

service：启动、停止、重新启动和关闭系统服务，还可以显示所有系统服务的当前状态。 

## 防火墙

firewall-cmd --list-all 参看防火墙 

sudo firewall-cmd --add-port=80/tcp --permanent 添加开放端口 

firewall-cmd --reload 重启 

systemctl stop firewalld 

systemctl start firewalld