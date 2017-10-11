## 作业的执行流程

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/hadoop/hadoop-3.png)

客户端：编写MapReduce代码，配置作业，提交作业
JobTracker: 初始化作业，分配作业，与TaskT通讯，协调整个作业
TaskT,   跟JobT通信，在分配的数据段上执行Map捉着reduce任务。
HDFS,保存作业数据、配置信息、作业结果等等。

### 提交作业
    提交前的配置：
    代码
Map和Reduce接口的配置， Map的输出与Reduce的输入必须一致。 四个参数： key， value, output(Map<key, List<value>>), report
FileInput和FileOutput来配置输入输出路径
### 初始化
    得到3写入的job.split输入数据的划分信息
根据数据划分的个数来觉得map的个数，然后给他建立好TaskInprogress来处理入参，并放到noRunningMapCache中，同样根据jobConf配置的mapred.reduce.tasks来决定reduce的数量，放在noRuningReduceCache中。  留着供给jobT分配任务的时候使用
创建两个TaskInprogress来一个创建map一个创建reduce.
### 分配任务
    JobT通过心跳知道有哪些TastT. 汇报工作状态和分配任务都是通过心跳完成的。 有固定的槽，会本地化，会尽量把数据和实际的处理放在同一台电脑上。
### 执行任务
    任务本地化，把运行所需要的数据、配置、代码从HDFS下载到版本低。然后通过launchTask启动任务。
执行的流程：
配置参数
在临时文件中添加MAP任务信息
配置log文件夹，配置map任务的通信和输出参数。
读取input,生成RecourdReader,然后生成MapRunnable，从RecourdReader中接收数据，并调用map方法
最后将输出调用collect收集到MapOutputBufffer中。
### 更新任务执行进度和状态。
    百分比是按照执行完的任务个数算的5s更新一次。 JobT不停的接收信息，直到完成。
## 错误处理机制
### 硬件故障
    单点就挂了。
    多点的时候如果多久没有响应，jobT任务taskT挂了。会让其他的taskT重新计算挂了节点算的map. 如果是reduce. 则会让其他节点继续执行未完成的reduce，这是因为reduce的结果会入hdfs.
### 任务异常失败
   抛出异常，jvm退出，taskT监听写日志，把任务标记为失败。或者执行太久也会强制退出。
    然后taskT会向jobT去申请新的任务，同时告诉jobT,一个失败了。jobT会重置失败任务的状态来达到重启任务的目的，重新分给别的人执行。如果4次都失败就不会重试了。
## 作业调度机制
    先入先出
    公平调度器， 让用户公平的共享集群，比如只有一个任务，也能够使用整个集群。
## shuffle和排序
分成了map端和reduce端
map端 对结果进行partition, 排序和分割。 然后把同一个partition的输出合并在一起，发给对应的reduce.
reduce.端会对不同map送来的同一个partition的进行合并，排序。  
优化： 减少io,设置缓冲区大小ip.sort.*. 来防止溢出写操作。在reduce端不行磁盘，直接用内存的等等。
## 任务执行
 推测式的，统计平均进度，比如有的执行时间比平均高，那么减少给他的任务，这种可能会因为是代码本身的问题
任务JVM重用， 每个任务都建一个新的JVM，有点浪费，可以配置为可以重用的：mapred.job.reuse.jvm.num.tasks.
跳过坏记录，重试4次，则跳过。 skipping模式。默认关闭，可以使用SkipBadRedord单独启动
每个任务都会有临时目录，要保证不同的任务不会同时写一个。