```
man info --help                   //to show how to use currently command
```

ll  = ls -l
ls -al list contains hidden file
ls -lt  正序
ls -ltr 逆序
-rw-r--r-- 1 root root 94409 Jan  2 20:09 1
drwxr-xr-x 3 root root  4096 Jan  2 11:43 InstallShield
[1][2        ][3][4   ][5     ][6     ][7                 ][8               ]
[1] type. d:directory. -:file, l:link file,b:store device,c:串口设备如鼠标等
[2]permission.
        [421][421][421]对应了owner, group, other的权限。
        [rwx]读写，执行
        1. 用数组的办法
            chmod 777 *
        2.用字符
             chmod u=rwx,go=rx test.log   user, group,other. a all,   =+-
             chmod a+r test.log
[3]表示有多少个文档连接到这个文件
[4]owner
    chown [username]:[usergroup] [filename]
[5]group
     chgrp [group name] [file name]
[6]Size
[7]date last modify time
[8]filename
        Ext2/3 255 长度，  全路径名为4096， 名字不能包含特殊字符。

FHS Filesystem Hierarchy Standard
/bin   系统执行文件目录，如cat, chmod,mkdir等
/boot   开机所用的文件
/dev    接口设备
/etc     配置信息  /etc/host,/etc/init.d等
/home 新增一般账户的目录，~当前用户的目录 ~user user用户的目录
/lib  开机相关的函数库，/lib/modules 驱动程序
/media  DVD等
/mnt  暂时挂载额外的目录
/opt   第三方软件安装目录Optional application software packages以前还习惯于放在/usr/local目录中
/root  root用户的家目录，为的是能跟根目录放在一个分槽中
/sbin  一些系统配置命令，服务器软件一般放在/usr/sbin中，自己安装的可执行文件放在/usr/local/sbin中
/srv  网络服务所需要取用的数据/srv/www
/tmp  临时数据
其他一些重要的不在FHS中的目录：
/proc 虚拟文件放置的数据时在内存中的/proc/cpuinfo, /proc/dma等
/sys  跟/proc非常相似
/usr  Unix software resource  类似于C:/Program files/ 软件开发者应该将他们的数据合理的分配在这个目录
/usr/X11R6 X window
/usr/bin  用户可用指令
/usr/include C++等程序用的头
/usr/lib   应用软件函数库
/usr/local   自行安装软件的目录
/usr/sbin 非系统运行必须的系统指令
/usr/share 放共享文件
/usr/src 源代码

/var /usr安装就会占用,/var是运行时占用，比如一些数据文件，缓存等
/var/cache
/var/lib  程序本身运行所需要的一些数据
/var/lock  
/var/log 记录登陆者信息
/var/mail
/var/run  某些程序启动后放置PID的目录
/var/spool 队列数据，比如新邮件看了之后就会删除

目录操作指令
. .. - ~ ~account
cd pwd mkdir rmdir
pwd -P 不以链接挡显示而是显示实际目录
mkdir -p 递归创建 -m 774 指定创建目录的权限
rmdir -p 删除所有子目录

PATH
echo $PATH
PATH="$PATH":/root
ls [-a] [-d只显示目录][l]
cp [-p带属性复制][-i覆盖提示][-r递归赋值][-u update新的才复制]
rm [-f不提示][-i互动模式][-r递归]

basename 最后的档名
dirname 上一级目录名

查看文件内容：
cat [-n行号]
more   空格一页，Enter一行，/搜索，q退出
head -n filename 头几行
tail [-n][-f]
touch 修改文件的时间，没有则创建文件
file filename  文件内部数据的格式，比如字符集

umask -S 查看新建文件的默认权限。  其他一些特殊权限SUID什么的ignore

which ls 查看ls命令的位置，是在PATH中进行查找的
whereis 从一个系统的数据库中进行查找，因此已删除的也可能找到，新建的可能找不到
locate 模糊查询,从数据库中，updatedb更新这个数据库，默认是每天一次的
find 
时间相关 find / -mtime +4 or find / -newer file
用户相关 find / -user username
文件名相关find / -name *name* 

df -h 以G等宜读单位表示磁盘信息。   FileSystem通常是分区， mount on挂载点到这个分区的文件夹入口，相当于Windows的盘符。

inode, 除了文件本身外保存了文件的属性信息和指向具体文件。
hardlink 多个文件对应同一个inode. 删除这个硬链接或者是源文件都不影响源数据，修改修改的是源文件。所有hardlink都 被删除才会删除元数据  ln
Symbolic link，快捷方式，是一个单独的文件。如果源数据删除查看这个则会出错。
ln -s 

Zip
gzip filename gzip -d unzip.  会把源文件删掉
tar -zcvf test.tar.gz cont* about.html 多个文件，可以用通配符，后面的文件或者文件夹用空额分开
tar -ztvf test.tar.gz  查看
tar -zxvf test.tar.gz
 tar -zxvf test.tar.gz about.html 解压某个文件
tar -zxvf test.tar.gz -C ./test2 解压到指定目录
tar -zcvf test.tar.gz --exclude=about* cont* about.html压缩但是除了

备份还原dump restore 