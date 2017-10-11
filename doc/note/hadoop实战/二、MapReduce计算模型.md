org.apache.hadoop.hdfs.server.namenode.SafeModeException: Cannot delete /user/root/output. Name node is in safe mode.
解决方法：
        执行 hadoop dfsadmin -safemode leave
map() 把输入的值转为 key，value
reduce() 接收的为key,list(value)进行一个统计。

FileInputFormat.addInputPath(job, new Path(args[0]));
FileOutputFormat.setOutputPath(job, new Path(args[1]));
### InputFormat
定义map的入参， InputSplit定义如何分片。 
预定义了很多种。
比较常用是FileInputFormat有很多子类 KeyValueTextInputFormat， TextInputForm(默认)。
TextInputFormat是每个文件的每一行都会对应一条记录。
key是偏移量，value是行内容。

### OutputFormat
默认是TextOutputFormat。作用是每条记录以一行的方式存入文件。

### MR 的控制流
比较简单：JobT 把任务分给TaskT, 然后TaskT执行任务把进度告诉给JobT,JobT看到TaskT失败，会把job提交给别人。

### MR数据流
文件夹->TextInputFormat按文件/行分成多个InputSplit输入给map()，
map处理，然后写入结果到本地磁盘。
reduce再读取本地磁盘，然后处理。


![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/hadoop/hadoop-2.png)

另外需要注意：
    一般不只一个reducetask，如果是多个reducetask每个负责一部分的key,会有多个输出文件
如果没有reduce任务，那么就会认为map任务就是reduce.
### Hadoop任务优化
    1. 任务调度    计算：分给空闲的机器。   I/O:将map任务分给InputSplit所在的机器，尽量本机，减少I/O
    2. 数据预处理和InputSplit的大小
因为更适合处理少量的大数据，而不是大量小数据。通常可以预处理一下。变成相对大一点的块。  通常可以这样看，一个map如果只运行几秒那么就要分配更多的数据。通常一个map的运行时间在一分钟左右比较合适。可以通过设置map的输入数据大小来调解map的运行时间。可以合理得而设置InputSplit block快的大小。也可以通过设置map任务的数量来调解。
    3. map和reduce任务的数量
map槽/reduce槽， 集群能同时运行的最大的数量。map的数量是按照运行时间来衡量。一般来说是0.95或者1.75任务槽
    4. combine函数
用于本地合并数据的函数。  比如wordcount本地的就会有很多重复的，相当于发给reduce的所有的count都是1，所以先本地合并一下比较好。减少I/O
    5. 压缩
在map的结果压缩可以减少网络传输， 对reduce压缩，可以减少些hdfs的时间，但是会对读取产生影响。
   

