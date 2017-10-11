## ZK 基础
   因为原语的表达比较难，所以ZK使用了一个类似于文件的系统，让调用者调用来实现自己的分布式原语。
   ZK通过操作一个小的数据节点，叫做ZNode来实现自己的原语。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/zookeeper/zk-2.png)

如图， 数据的缺失，经常表达了一些特殊的意义。
/master node缺失标示没有master被选举出来。
/workers    node标示当前活动的节点，不活动的节点要被删除
/tasks node标示，所有等待执行的任务，增加新的节点来增加恩物，另外还可以通过等待执行的结果表示执行的状态。
/assign 标示，保存了所有的分配信息。


## API概述
ZNode可以包含或者不包含数据。  如果包含数据，则存储为字节数组的形式。如果需要转化成别的需要额外使用其他的序列化的包。
create /path data
delete /path
exists /path
setData /path data
getData /path
getChildren /path
不支持部分写或者读。
### Znode的不同模式
        创建的时候需要指定模式：persistent 或者是ephmerel的persistent_sequential, and ephemeral_sequential.
        persistent的需要显示的调用命令删除
        ephmerel的客户端断掉连接就会删除
        sequential会给节点自动加上一个自增的序列号。

### Watches and Notifications
     远程访问开销很大，为了感知zk Znode的变化，需要使用轮询的方式，性能较差。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/zookeeper/zk-3.png)

  这个模式可能会有一个问题，因为watch只有一次有效，因此有可能在设置第二个watch的之前，有了新的变化。这个时候感觉会漏掉一次变化。
    zk保证不会漏掉，但是更多的部分没有看懂，以后有例子的时候再看。  大概的意思是保证顺序。或者是记录一个正在读的状态。

### 版本
每一次数据变化，有一个递增的版本号。有的操作需要指定版本。 比如setData和delData的操作，有点类似于CAS 

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/zookeeper/zk-4.png)

## ZK的架构
ZK的服务器有两种运行的模式: 独立standalone和集群  

### ZooKeeper Quorums
ZK集群
这种模式下，会复制所有的数据到每一个节点。
有一个最小的数量是选举算法的最小的数量，是三台。

### Session
    回话断开则临时节点释放。
    集群模式中，单点故障了，会转移这个回话，是对客户端安全的。
    一个回话的命令执行是FIFO的

## 开始ZK之旅
    bin/zkServer.sh start
    ls /
      create /workers ""
    delete /workers
### 回话的生命周期
    
### 集群部署
    PID和配置文件。
    多台之间进行选举。 client随机链接到一台， proxy算法只是决定每一次操作都是一致的。
   

## ZK实现锁原语
    每一个想获得锁的进程 尝试去创建一个znode. /lock， 如果成功则获得锁，并进行业务操作，一个问题是p可能崩溃，这样就不会释放锁了。 这样的话我就需要创建一个瞬时的锁， 其他的pi无法获得锁的时候就watch一下，一旦锁释放了，就重新尝试获得锁。
    
## 实现一个Master和Worker的例子。
    三个角色：
    Master:  观察worker的状态，和分配任务
    Worker:    执行任务和关注新的任务
    Client:    创建任务和等待执行的结果
###TheMasterRole
    只有一个，我们打开一个client，用创建一个/master来标示。
    create -e /master "master1.example.com:2223"
    为了保证主不挂掉，我们找一个主的备份，    新启动一个client然后create -e /master "master2.example.com:2223"
    会提示已经有了，然后stat /master true来设置watch.
    然后关掉第一个client等一会儿发现收到了watch的通知，这个时候可以再次执行创建的命令了。
### Workers,Tasks, and Assignments.
    create /workers ""        标示所有的工人
    create /tasks ""            标示所有的任务
    create /assign ""    标示任务分配
作为master需要知道worker和task的变更好安排工作。
    ls /workers true   用来监视 /workers这个目录
    ls /tasks true
### Worker Role
    告诉master是一个可用的workder.   create -e /workers/worker1.example.com "worker1.example.com:2224"
    master得到一个提示，然后知道有worker了。
    然后worder 来创建一个节点来接收任务：create /assign/worker1.example.com ""
        并且来监视这个任务分配，看自己是否被分配了任务。 ls /assign/worker1.example.com true
        
### Client Role
    客户端来添加任务到系统中。这里我们假设请求运行CMD命令 create -s /tasks/task- "cmd"
        创建了一个序列的节点。
        客户端关心这个任务是否执行完了，所以客户端ls /tasks/task-0000000000 true
    上面创建节点的时候master只有有任务来了。  master来查看任务，查看工人
            ls /tasks
            ls /workers
            分配任务  create /assign/worker1.example.com/task-0000000000 ""
    Worder知道自己被分配任务了，           来查看自己的任务：ls /assign/worker1.example.com
          如果他执行完了，然后设置完成的属性：create /tasks/task-0000000000/status "done"
           然后客户端收到提醒，任务被执行完了。来查看结果get /tasks/task-0000000000/status   get /tasks/task-0000000000