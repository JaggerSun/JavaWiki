一个逻辑的分布式文件系统。  
可以认为是一linux文件系统，同样支持各种api, shell命令的， sdk包的。
##访问方式
shell command : hadoop fs -ls /  等同于：hdfs dfs -ls /
java sdk
其他语言通过Thrift
WebDev:http://localhost:50070/dfshealth.html#tab-overview

## 优势和问题
处理大文件
流式读取，适用于一次写入，然后每次读取很多文件
不适合低延迟的访问，这种可以使用其他弥补措施，比如hbase
不适合存储大量小文件
不支持多用户同时写入

## HDFS体系结构
### 概念
    块： 文件系统块。 固定大小默认64M，副本为3份，   查看块信息命令。 hdfs fsck / -files -blocks
    NameNode和DataNode分别承担master和worker的角色。
            name管理文件系统的命名空间， 维护整个文件系统目录树和索引目录。 从name中能够获得每个文件每个块所在的datanode. 每次重启动态重建。
### hdfs体系结构
    file:///D:/hadoop/hadoop-2.7.2/share/doc/hadoop/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/hadoop/hadoop-4.png)

系统宕机是常态
自动回复是重点
适用于批处理而不是用户交互
高吞吐不是低延迟
只能增加不能更新
迁移计算而不是迁移数据
一个节点一个datanode, 一个文件分成了多个块，这些快存在一组datanode上。
    1. 
副本策略是 一个在本地机架，另一个在同机架不同设备， 最后一个在不同机架上，这样就能更好的容错和保证性能。
    2. 
刚启动的会进入一个安全模式，不能进行副本复制。  当确认一定比例安全且 + 30s后则取消这个模式。
   3. 文件安全
如果namenode挂了就完了。两个办法：
   namenode持久化
   同步运行一个secondary NameNode. 同步编辑日志。但是有一定延后，必然丢失。
