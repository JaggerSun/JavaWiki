## 简介
模仿了bigTable。

分布式多版本、列存储。使用HDFS作为文件系统。

向下提供存储，向上提供运算。  用mapReduce来并行处理数据。

## 基本操作
[http://hbase.apache.org/book.html#quickstart](http://hbase.apache.org/book.html#quickstart "官方说明")

### 安装
[版本支持情况](http://blog.csdn.net/javastart/article/details/51329665 "版本支持情况")
单机啥都不用， 分布式需要hadoop

需要拷贝hadoop的jar包到hbase下面

    <configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://K-Master:9000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
	</configuration>

	vim conf/hbase-env.sh
	export JAVA_HOME=/usr/java/jdk1.7.0_80
	export HBASE_MANAGES_ZK=true

	./start-hbase.sh 

启动成功之后的测试界面：
[http://114.55.139.161:16010/master-status](http://114.55.139.161:16010/master-status)


###hbase shell
   
	./hbase shell
	help
	create 'test', 'cf'  // 第一个参数是表名，后面是列族名称
	list	// 查看所有表
	// 插入数据，在row1行， cf:a列族， 值为value1.
	put 'test', 'row1', 'cf:a', 'value1'
	scan 'test'		// 查看表内的所有数据
	scan 'test',{COLUMNS=>'cf', LIMIT=>1} // 只查一行
	get 'test', 'row2'	// 查看指定行

## 体系结构
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/hadoop/hadoop-7.png)

主从结构，  zk用来协调。

 1. HRegion 逻辑存储结构，过大的被切割成多个。每一个存储一张表的若干行。 主键起始id.
 2. HMaster
HRegionServer和Master通信， 同步每天Server维护哪些HRegion。
主要负责：增删改查等操作。 重新分配region等。

 2. HRegion Server

一个Region只会被一个Server维护，数据保存到hdfs，Server负责跟hdfs通信。
更新写入数据的时候会分配到一个Server上面先写cache和Hlog. 然后flush
flush之后会生成HSoreFile.Btree类型的。因此很快。
如果经常刷新会导致文件过多。所以最好是定期合并一下，用compact方法。

 3. ROOT和META.

记录HRegion的元数据叫meta, 表名+开始主键+唯一ID。 可以分成很多的Region.

ROOT是用来记录上面的这个Region结构的，就是对应表去那个Region找元数据。



## 数据模型
### 概念视图
行，列族(family：quality),时间戳，版本来唯一确定一条数据。
关于版本
[http://www.php3.cn/a/130.html](http://www.php3.cn/a/130.html)

每次put都会有新的版本，默认只显示最新版本。
每行可以有多个列族的组合。列族可以为空。
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/hadoop/hadoop-8.png)

### 物理视图
物理上是按照列来存储的。 而不是行。
可以认为一列是一个存储结构。存了很多行的相同列的值。


## Hbase与RDBMS
 - 简单字符串类型
 - 很简单的操作，没有连接等
 - 基于列式存储
 - 可伸缩性，很容易增加硬件数量进行扩容。

## Hbase与HDFS
基于HDFS

## Java API
[http://blog.csdn.net/qq_31570685/article/details/51757604](http://blog.csdn.net/qq_31570685/article/details/51757604)

妈蛋的，又有问题，不知道为什么执行要卡死。

应该还是环境搞得有问题。

## 与MapReduce整合
没啥，就是不读hdfs了，改成读hbase了。

## 模式设计

这不边有两个例子，先略过了。

