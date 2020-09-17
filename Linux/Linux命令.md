# ps -ef | grep xxx

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

## 防火墙

firewall-cmd --list-all 参看防火墙 

sudo firewall-cmd --add-port=80/tcp --permanent 添加开放端口 

firewall-cmd --reload 重启 

systemctl stop firewalld 

systemctl start firewalld

## tar -zxvf

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

