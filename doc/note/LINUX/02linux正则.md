grep 进阶   -A 找到的行后面多少行也打印出来
    -B 前面多少行
--color=auto
dmesg | grep -n -A3 -B2 --color=auto 'eth'
      grep支持正则
            行首^,行尾$, \转义
grep -v '^$' /etc/syslog.conf | grep -v '^#'  不要空白不要#
    .任意一个字符
    * 重复前面的字符0到多次
grep -n 'go\{2,\}g' regular_express.txt    必须用\}转义，表示2到多个
注意正则表示跟通配符不同。。。。。。。


sad 命令，能够增加删除，替换等，甚至可以直接对文件进行操作
awk
        last -n 5 | awk '{print $1 "\t" $3}'   常用的print命令$1表示第一列 "\t"表示tab,
         会自动把一行按照空格分隔然后复制到$1这样的变量中。
FS 分隔符 NR所处理的是第几行
awk 'BEGIN {FS=":"} $3 < 10 {print $1 "\t " $3}'     在开始的时候指定分隔符， 然后第三行小于10的
        经常与printf连用，还可以进行分支判断。
       cat pay.txt | awk '{if(NR==1) printf "%10s %10s %10s %10s %10s\n",$1,$2,$3,$4,"Total"} NR>=2{total = $2 + $3 + $4 printf "%10s %10d %10d %10d %10.2f\n", $1, $2, $3, $4, total}'

diff  比较稳健或者目录 -B忽略空白行 -i忽略大小写
patch 用diff做补丁，然后用patch来更新补丁



