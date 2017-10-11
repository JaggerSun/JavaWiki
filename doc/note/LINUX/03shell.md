vi
三个模式：一般，编辑i等和命令:/?。
按键说明：
==一般模式下
n方向  移动多少个字符
ctrl+f下一页
ctrl+b上一页
n空格 向后移动n个字符能换行
G文档末尾
gg开头
nG文档的第n行
nEnter向下移动n行
搜索
/word向下找
?word向上找
n重复前一个搜索动作
N相反的搜索动作
:wq保存退出
:q!强制退出
:100,200s/vbird/VBIRD/g 搜索100行和200行之间用VBIRD取代vbird
删除复制
x,X向后向前删除一个
nx连续删除
dd删除一行
ndd删除n行
d1G删除光标所在到第一行的所有数据
dG删除到最后一行的所有数据
d0,$删除本行光标到行头和结尾的数据
yy复制所在一行
nyy复制乡下n列
p在下一行插入复制内容
u后退动作
ctrl+r重做上一次动作
======切换到编辑模式
i
o在新的一行插入
=======切换到指令列模式
:wq 保存退出
:q!强制退出
:w filename另存为
:set nu显示行号
=====编码转换
iconv -f 原本编码 -t 新编码 filename [-o newfile]

Shell 命令行人机界面，为了调用应用程序接口。与Kernel交互
/etc/shells 看支持哪些shell
bash 一种shell linux默认的
一些功能：
        [tab]补全
        别名 alias lm='ls -al'
        后台运行
        scripts
        通配符
type ls  查看是否是内建命令
\Enter 在输入命令的时候换行

环境变量
linux是多用户的，因此每个用户登录都会分配一个bash，会有一套对应的环境变量PATH,MAIL等
赋值的一些规则
        =两边不能有空格
        =右边如果有空格可以用"或‘括起来，区别是“的特殊字符$等会保持原有特性
        \可以将特殊字符变成一般字符
        ``或者是$(命令)可以赋值为命令的执行结果
        可用$PATH:的形式扩展变量内容
        export使普通变量变为环境变量，这样就能够在其他子程序中运行
        unset取消赋值
uname -r核心版本
env 查看环境变量 
set 查看所有变量，包含了环境变量
一些特殊的变量：
        PS1:等待输入命令状态左边的内容。可以设定为比较有用的展示方式PS1=''
        $ 当前进程号echo $$可以打印
        ? 上个命令的回传值，正确执行是0否则应该有错误信息
export
locale 配置的编码情况，-a支持的编码，一般去改LANG这个变量就可以了。还可以修改cat /etc/sysconfig/i18n
这个文件，永久生效的    
read -p "please keyin your name:" -t 30 yourname  -t等待多少秒， -p提示语言，输入会存在变量yourname中
declare 
    -a 数组
    -i 整数， 默认是字符串，现在可以用300+100这样的表达式赋值了
    -r ,表示是个常量不能改
数组： var[1]=”“  var[2]=""  echo "$(var[1])"

ulimit 限制用户可用资源
echo ${path#/*:} path是变量，#是最短，##是最长，/*:是匹配部分，删除
% 跟#差不多，不过是从后面开始删除，#是从前面开始
echo ${path/sbin/SBIN}sbin是被替换的，SBIN是替换的，path后面的/表示替换第一个，//表示提花你所有
username=${username-root}  username没有配置则配置为root
username=${username:-root} 及时配置为空串也会配置为root

!n 第多少条命令
!!上一条命令
!command最近的包含了这个命令的命令
进站欢迎信息cat /etc/issue

bash的环境配置文件
login shell会加载配置/etc/profile, 系统配置
~/.bash_profile 或 ~/.bash_login 或 ~/.profile  个人配置
/etc/profile还会呼叫很多其他的sh,比如用来配置字符集的/etc/profile.d/lang.sh
source 文件  会重新读一遍配置而不用重新登录

~/.bash_history 命令历史记录
~/.bash_logout  退出时要做的操作
通配符 跟正则表达式通用的
* ? [] [a-b] [^]
特殊符号 #,\,|,~,$,!,/,<,>'',"",``,(),{},&将动作变成后台运行

重定向管道
./test.sh > list_right 2> list_error  2>是把错误信息重定向到文件。默认是标准输入输出，也就是屏幕 >> 是增加，>是覆盖
find /home -name .bashrc 2> /dev/null 将错误信息丢进垃圾桶
./test.sh $> list_all正确和错误信息都写入
cat > catfile 用键盘新建文件用ctrl+d结束
cat > catfile < ~/.bashrc  把后面的文件的内容来代替键盘输入，跟cp命令一样
cat > catfile << EOF 键盘输入创建文件遇到EOF停止不用ctrl+d了

cmd;cmd  ;符号是顺序执行命令，可以一次执行多个命令了
cmd1 && cmd2 cmd1运行正确才开始运行cmd2  || 是运行错误才运行cmd2
ls /tmp/vbirding && echo "exist" || echo "not exist"  类似于if else

管线命令
 command1 | command2 管线命令，能够把前一个命令的输出变成下一个命令的输入
ll | less  把列表分页显示
撷取命令 ：cut, grep 把一段数据分析后取出想要的，通常是一行一行分析的
cut 
echo $PATH | cut -d ':' -f 5  以:为分隔符取第五个
export | cut -c 12-  第12个字符到结尾
grep 打印满足条件的行
    -c 次数
    -i 忽略大小写
    -n行号
    -v反向选择
    --color=auto 高亮
grep -cinv '字符串' filename
cat /etc/passwd | sort -t ':' -k 3  -t分隔符， -k那个区间用来排序
-f 忽略大小写，-b忽略最前面的空格-n纯数字-r反向-u去重复
uniq -i忽略大小写 -c计算数目。  去重复列
wc -l 多少行 
tee 即保存进文件，又输出到屏幕或者是给其他命令使用
ls | tee last.list | grep 
字符串转换
 DOS 断行字符与 Unix 断行字符转换：dos2unix  unix2dos
tr -d str 删除str,    -s取代重复的字符
       last | tr '[a-z]' '[A-Z]'   大写变小写
col -x 把[tab]改成空格
cat /etc/man.config | col -x | cat -A | more    -A查看特殊字符。
join -t ':' -1 4 /etc/passwd -2 3 /etc/group
    有点像数据库操作。-t以什么分割 -1是第一个文件中用来分析的字段   -2是第二个，意思是上面文件中的一行中，第四个和第三个一样就合成一行
paste -d 分隔符 把两行何在一起
expand -t file把tab改空格-t跟空格个数
split 把大文件分割成小的-b 按大小分可以带k,b,m这样的单位 -l按行数分
 xargs 为某个命令产生参数
            -n 次数， -p询问 -e遇到字符终止   -0还原特殊字符
 tar -cvf - /home | tar -xvf -  -号表示前一个命令的stdout
start vnc
vncserver :1
 
设置环境变量
export JAVA_HOME=/usr/java/jdk1.6.0_22
export PATH=$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
 
su 更换用户
 
rpm -qa | grep Nokia-REPORTER-cntgen
-i install
 
解压rpm包 rpm2cpio oracle-instantclient11.2-basic-11.2.0.2.0.i386.rpm | cpio -div
 
你可以到/opt/WebSphere/AppServer/profiles/clab684node04/installedApps/SOL  查看jar包是否更新
 
rpm -qa|grep mysql
rpm -e
 
 =============route
route -n
0.0.0.0         10.8.83.1       0.0.0.0         UG    0      0        0 bond0.83
0.0.0.0         10.8.117.193    0.0.0.0         UG    0      0        0 bond0.74

route add default gw 10.8.83.1
route del default gw 10.8.117.193

:.,$d全删除在vi模式下

`sed '/dubbo.application.name/!d;s/.*=//' conf/dubbo.properties | tr -d '\r'`
``是执行的意思。 sed是处理文件每一行的。
sed command file 的结构
command中用;分割
/dubbo.application.name/!d; 表示含有dubbo.application.name的不删除
s/.*=// s表示替换， s/正则/替换的内容   表示把=号前的所有内容替换为空

找安装路路径  ls -lrt /usr/bin/java
