一些基本
        \可以命令分行
        读取到Enter就尝试执行
        ./xxx.sh来执行
        还可以用sh xx.sh 或者bash等命令执行
        想加入path可以把这个命令拷贝到已经加入到PATH的目录
#!/bin/bash  宣告使用的sh解析器的名称
exit n 运行完script之后的$?为n，这样可以进行一些特殊的操作
var=$((运算内容))  进行加减乘除的运算或者用declare
运行方式
        sh xx.sh  其中操作的变量不会再bash环境中
        source xx.sh 在父bash环境中是存在的
test 
       -e  存在
        -f 存在且为文件
        -d 存在且为目录
[] 判断符号，[ “$name” == "VBird" ] 各处都有空格，本身能直接使用，但是通常跟if一起使用

脚本执行默认变量
        $0 脚本名
        $n 脚本后的第几个变量
        $# 参数个数
        $@ 所有参数用空格聘妻的字符串

if [ 条件判断式一 ] && [ 条件判断式一 ]; then
    当条件判断式一成立时，可以进行的命令工作内容；
elif [ 条件判断式二 ]; then
    当条件判断式二成立时，可以进行的命令工作内容；
else
    当条件判断式一与二均不成立时，可以进行的命令工作内容；
fi

case $1 in
  "hello")
    echo "Hello, how are you ?"
    ;;
  "")
    echo "You MUST input parameters, ex> {$0 someword}"
    ;;
  *)  
# 其实就相当於万用字节，0~无穷多个任意字节之意！
 
    echo "Usage $0 {hello}"
    ;;
esac
定义 function
function name(){
        内建变量与脚本相同
}


while [ "$yn" != "yes" -a "$yn" != "YES" ]
do
    read -p "Please input yes/YES to stop this program: " yn
done
echo "OK! you input the correct answer."
循环
for animal in dog cat elephant
do
    echo "There are ${animal}s.... "
done 
遍历
for username in $users 
for sitenu in $(seq 1 100)   1-100的数遍历
filelist=$(ls $dir)  命令执行结果的遍历
for (( i=1; i<=$nu; i=i+1 ))   普通的for循环
do
    s=$(($s+$i))
done    
 
sh -n 检查语法
sh -x 把运行过程都列出来

