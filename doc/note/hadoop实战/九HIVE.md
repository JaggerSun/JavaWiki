把sql变成mapreduce任务，极大的推动了hadoop作为数据仓库的应用。
高延迟的大批量处理，很大的可扩展性。

## 数据存储
表、外部表、分区partition、桶bucket
每个表都是一个存储目录 /${hive.metastore.waehouse.dir}/htable.
分区是子目录每一个分区都对应一个目录  /../htabel/ds=20100301/city=beijing
桶 用hash值切分数据 /../htabel/ds=20100301/city=beijing/part-00010
外部表 有点像视图。
hive元数据不存储在hdfs中，可以选择存储在Mysql,Derby中。

## hive的基本操作
### 安装
下载： 
http://hive.apache.org/
配置环境变量Path和HIVE_HOME;
必须创建了HADOOP_HOME环境变量
 $ $HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
  $ $HADOOP_HOME/bin/hadoop fs -mkdir       /user/hive/warehouse
  $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
  $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse
default 变成hive-site.xml
${system:user.name} 换成hive
${system:java.io.tmpdir} 换成tmp
配置mysql
 <name>datanucleus.schema.autoCreateAll</name>
    <value>true</value>

./hive
Cannot find hadoop installation: $HADOOP_HOME or $HADOOP_PREFIX must be set or hadoop must be in the path

vi hive-config.sh
export HADOOP_HOME=/usr/local/prince/hadoop-2.7.2
export PATH=${HADOOP_HOME}/bin:$PATH
default 变成hive-site.xml
配置mysql,jar包拷贝过去
./schematool -dbType mysql -initSchema
${system:user.name} 换成hive
${system:java.io.tmpdir} 换成/usr/local/prince/hive/tmp
http://blog.csdn.net/freedomboy319/article/details/44828337
>create table h_user(id int ,name string)row format delimited fields terminated by ',';
>show tables
>describe h_user;

DDL操作，略过。用的时候再说。
load data local inpath '/usr/local/prince/apache-hive-2.1.1-bin/data/1.csv' overwrite into table h_user;

### Hive QL的使用
http://blog.csdn.net/wulantian/article/details/38271803

###hwi 需要编译出来一个war包，懒得搞了。

###jdbc访问。 
./hiveserver2
jdbc:hive2://localhost:10000/default

###然后是beeline
新型的hive CLI
本地连接：   
./beeline -u jdbc:hive2://
执行show tables;

远程连接就他大爷的不行了：
./beeline -u jdbc:hive2://127.0.0.1:10000
./hadoop namenode -format 这样了还是不行
加入了<property>
    <name>hadoop.proxyuser.root.hosts</name>
    <value>localhost</value>
</property>
<property>
    <name>hadoop.proxyuser.root.groups</name>
    <value>root</value>
</property>
还是不行。这样不行的话，那么远程执行java代码就肯定不行。

## 优化
1. 列裁剪
2. 分区裁剪   指定分区，减少查询范围
3. JOIN  驱动表尽量少
 JOIN 的key会被合并为一个mapreduce
4. Map JOIN, 有点像mysql的索引列。 不需要reduce全在map中就能完成
5. Group By
6. 合并小文件。





