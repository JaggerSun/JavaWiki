##  AMQ 高可用
  一个场景是容灾。  通常需要部署多个broker,在一个down掉之后，其他的能够接管。
  这就是AMQ的主/从部署。
  一个broker作为master或者primary。其他的一个或者多个slave看这这个主，一旦失败则接管。
  java的客户端提供了自动的故障转移，能够重连到这个新的主机上。
  将会有两种部署方案：有共享和无共享存储的。

###无共享存储

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-9.png)

最简单的一种情况了， 每个borker有自己的存储，通过replicated机制作数据同步
先启动主，然后启动从，从链接到主，从不会对外有访问能力。 一旦主失败，则切换主从
生产者向master发送消息， master会等待slave同步完成之后再进行处理，自己入库等等。
当master挂了：
  自杀，是为了保证与主库的状态。 需要手动做主从切换，并且增加新从
  取而代之， 启动自己的网络传输部分。
第二种，那么客户端做如下配置就能够实现自动的故障转移了：
failover://(tcp://masterhost:61616,tcp://slavehost:61616)?randomize=false

局限性：
  只会replicate链接点之后的数据给从，可以设置waiteforslave属性来保证有从连接到的时候才接客。
一主只能一从。 从不能再有从
需要手工干预完整性
当一定的停机时间是可接收的，并且需要一定的人工参与来进行新的从机接入配置的时候。
    
  如何配置：

    <services>
	<masterConnector remoteURI="tcp://remotehost:62001"
	userName="Rob" password="Davies"/>
	</services>
     shutdownOnMasterFailure   从机配置，是否陪葬
     waitForSlave                                       是否等slave
     shutdownOnSlaveFailure                   是否殉情
### 共享存储master/slave
   共享存储，但是只有一机在同一时间接客。这种不需要手动维护完整性。不会限制slave的数量。 可以通过数据库或者文件系统，共享。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-10.png)

已经使用了关系型数据库了的就会特别的简单。
  只有取得了锁的，才会提供服务。 其他的被认为是slave.
  什么时候使用：如果已经用了关系型数据库，那么就是比较理想的配置了。
   
使用共享文件：
  推荐GFS
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-11.png)

##  怎么通过网络topo发送消息。其实是一种集群
### 存储和转发
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-12.png)
存储前被转发。   
默认只能是单向的。

在大规模部署的时候一般都是讲高可用和网络配置结合起来：
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-13.png)

###网络发现。
  有两种方式： 静态：使用brokerUrl, 以及动态，使用组播或者多播以及广播。
  一个多播发现的配置：


    <networkConnectors>
	<networkConnector uri="multicast://default"/>
	</networkConnectors>
	   静态的
	<networkConnectors>
	<networkConnector
	uri="static:(tcp://remote-master:61617,tcp://remote-slave:61617)"/>
	</networkConnectors>
    initialReconnectDelay 重连接时间

### 网络配置
  这一章将的是具体的配置参数，用到的再说。

## 为大型分布式系统部署AMQ
  扩展AMQ，并且进行垂直及水平缩放。
###垂直缩放
   增加单一AMQ能够处理的连接数的方式的。
   默认可以使用NIO来达到这个目的。
     
	<broker>
	<transportConnectors>
	<transportConnector name="nio" uri="nio://localhost:61616"/>
	</<transportConnectors>
	</broker>
         
    broker可以用一个线程来调度多个客户端连接的消息。  
        AMQ可以选择使用一个线程池，设置这个系统参数：org.apache.activemq.UseDedicatedTaskRunner
        确保内存：ACTIVEMQ_OPTS="-Xmx1024M \
-Dorg.apache.activemq.UseDedicatedTaskRunner=false"
         可以修改broker占用的内存。  
        可以I关闭Tight Encoding，减少每个请求的CPUString uri = "failover://(tcp://localhost:61616?"
+ wireFormat.tightEncodingEnabled=false)";
        
这样基本上几千个连接是没有啥问题的。
        主要说了这么几个配置项：
                增加并发连接，减少线程使用，增大内存，使用正确的存储。
###Horizontal scaling水平扩展
   可以配置client连接到一个集群(network 配置)，使用如下的uri:
        failover://(tcp://broker1:61616,tcp://broker2:61616)?randomize=true
   使用随机分配的方式。为了防止都随机到一个上面，可以设置danymicOnly属性和，把reprechSize预加载设置小。
   横向因为会有更多的信息中转，所以会有更多的消耗。  还有一种混合的方式适合解决更复杂的问题。
### 流量分配
        客户端决定去哪个。在客户端维护多个JMS的连接。
        


