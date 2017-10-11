## HDFS文件结构
%HADOOP_HOME%\data\dfs\namenode\current\
        VERSION      
        edits
        fsimage
        fstime
### VERSION
    clusterId namenode和datanode一致
    namespaceID 格式化时候产生，唯一标志
    layoutVersion 每次结构发生变化-1

### 编辑日志及文件系统镜像
    写操作在编辑日志中记录，并且在内存中保存元数据，然后刷新，没写完之前不会刷新。
    fsimage不是随时写的， 重新启动恢复的时候是加载这个文件和执行编辑文件。
    因为编辑文件会越来越大，恢复的时间会变长，为了解决这个问题，使用secondary,他就是个帮助namenode处理编辑日志和文件镜像的。  定期去合并一下变成一个新的fsimage.

### DataNode目录
    在很深的目录下面。有current.都是blk和blk.meta
    VERSION文件中clusterID是一致的
    blk和meta的一个是实际的数据文件 ，一个是元数据文件。

## hadoop 的状态监视和管理工具
### 审计日志
vi etc\hadoop\log4j.properties中
更多配置项以后说。

### 监控日志
http://localhost:8088/logLevel
这个地址能够在线设置，取得是vi etc\hadoop\log4j.properties 重的值，
在log4j.appender.X这样的后面名字就是key，然后可以设置和修改值。
http://localhost:8088/stacks
这个地址能够看很多堆栈和线程信息

### Metrics
HDFS和mr的守护进行都会收集一些度量信息，比如dataNode要收集：写入的字节数、被复制的文件数和请求等。
vi etc\hadoop\hadoop-metrics.properties
默认都是：org.apache.hadoop.metrics.spi.NullContext这个标示啥都不收集。
看那个文件，能看到已经有了很多被别的：
#jvm.class=org.apache.hadoop.metrics.file.FileContext                    写到一个文件
# jvm.class=org.apache.hadoop.metrics.ganglia.GangliaContext     写一个ganglia的分布式集群监控
还有一个关键的：NullContextWithUpdateThread  

### JMX
这个也是用到了再说吧。

### Ganglia
一个分布式管理的开源软件。
能查看cpu等等各种信息。挺不错的。

### hadoop管理命令
hdfs dfsadmin -report  查看各种dfs的信息
hdfs fsck / 可以验证块是否丢失

## hadoop集群的维护
### 安全模式
    启动之后是加载fsimage到内容，然后用edits文件重构数据，生成新的fsimage. 这个过程处于安全模式，只能只读。
    安全模式的时候namenode不会处理复制块的请求，因为这个时候处理可能会造成冗余，有的还没准备好但是实际上已经有块副本了，它不知道，。
    强制默认30分钟。 小集群可以设置为0：dfs.safemodel.extension
    完成了最小副本集就退出安全模式， 指99.9%的文件快达到了dfs.replication.min所设置的副本数量。
    hdfs dfsadmin -safemode get   判断safemode的状态，还能够手工进入和退出安全模式

### hadoop备份
    1. 元数据的备份
        namenode中的数据，很重要。最直接的办法使写脚本备份previous.checkpoint子目录(fs.checkpoint.dir属性确定)。
    2. 数据的备份
        不能完全依赖于其副本机制。 一般是通过对关键数据进行额外的备份，拷贝到其他的hdfs集群中去。一般是个定时任务。比如distcp工具。

### hadoop的节点管理
    1. 添加新的节点。
    dfs.hosts在namenode的本地，包含每个datanode的地址。
    mapred.hosts 记录了taskt的列表。
    向include文件中添加新的节点的网路地址。
    执行hdfs dfsadmin -refreshNodes
    
    2.  撤出节点
 
集群升级。这些都先不看了，以后漫漫看。



