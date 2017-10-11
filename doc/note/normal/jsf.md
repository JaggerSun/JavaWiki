JSF（中文名：杰夫）是Jingdong Service Framework （京东服务框架）的缩写，JSF是SAF的演进（研发版本号：SAF210）完全自主研发的高性能服务框架；它据有如下的特性：

1.高效RPC调用，20线程场景下调用效率比SAF高30%以上；

2.高可用的注册中心，完备的容灾特性；

3.服务端口同时支持TCP与HTTP协议调用，支持跨语言调用，构造一个HTTP POST请求即可对接口进行测试；

4.支持msgpack、json等多种序列化格式，支持数据压缩；

5.提供黑白名单、负载均衡、provider动态分组、动态切换调用分组等服务治理功能；

6.提供对接口－方法的调用次数、平均耗时等在线监控报表功能；

7.兼容SAF协议，可以调用SAF1.X接口；

8.全部模块均为自主研发，自主设计应用层JSF协议；各模块功能可控，可扩展性较好；

 

适用场景
适合于分布式架构下，服务与服务之间的同步调用，数据量小（单个请求小于20K）并发大的场景；



服务接口设计原则：
1.接口参数尽量用简单的类型，比如基础类型Integer、Boolean等，如使用自定义类型也请保证此类型的结构尽量简单、嵌套层数尽量少，这样会减少序列化时的消耗；

2.传输的数据尽量少，大数据会占用更多有限的带宽；

3.保证每次RPC调用的原子性，尽量减少在一个事务过程中发起的RPC调用，检查RPC调用的返回值或异常；

4.保证关键接口的幂等性；

5.制定服务等级协议（SLA），估算线上并发量，考虑每个请求的大小以及服务器带宽，压测单个服务在单位时间内的处理能力，计算需要部署的服务实例数；


来源： <http://jpcloud.jd.com/display/cloud/JSF>

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jd/jd-2.jpg)

Index Service: 索引服务，提供注册中心地址列表；

注册中心：提供服务注册、订阅，服务上下线；配置下发、状态读取（如client所拥有的地址列表），为了容灾多实例部署；

Monitor Service：收集所有客户端所上传的性能统计数据；

Web管理端：服务管理界面，可以在此配置权重、路由规则等；



influxdb  分布式时序数据库

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jd/jd-3.png)

名词解释：

cluster：
代表一个集群，集群中有多个节点，其中有一个为主节点，这个主节点是可以通过选举产生的，主从节点是对于集群内部来说的。es的一个概念就是去中心化，字面上理解就是无中心节点，这是对于集群外部来说的，因为从外部来看es集群，在逻辑上是个整体，你与任何一个节点的通信和与整个es集群通信是等价的。

shards：
代表索引分片，es可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上。构成分布式搜索。分片的数量只能在索引创建前指定，并且索引创建后不能更改。

replicas：
代表索引副本，es可以设置多个索引的副本，副本的作用一是提高系统的容错性，当个某个节点某个分片损坏或丢失时可以从副本中恢复。二是提高es的查询效率，es会自动对搜索请求进行负载均衡。

recovery：
代表数据恢复或叫数据重新分布，es在有节点加入或退出时会根据机器的负载对索引分片进行重新分配，挂掉的节点重新启动时也会进行数据恢复。

river：
代表es的一个数据源，也是其它存储方式（如：数据库）同步数据到es的一个方法。它是以插件方式存在的一个es服务，通过读取river中的数据并把它索引到es中，官方的river有couchDB的，RabbitMQ的，Twitter的，Wikipedia的，river这个功能将会在后面的文件中重点说到。

gateway：
代表es索引的持久化存储方式，es默认是先把索引存放到内存中，当内存满了时再持久化到硬盘。当这个es集群关闭再重新启动时就会从gateway中读取索引数据。es支持多种类型的gateway，有本地文件系统（默认），分布式文件系统，Hadoop的HDFS和amazon的s3云存储服务。

discovery.zen：
代表es的自动发现节点机制，es是一个基于p2p的系统，它先通过广播寻找存在的节点，再通过多播协议来进行节点之间的通信，同时也支持点对点的交互。

Transport：
代表es内部节点或集群与客户端的交互方式，默认内部是使用tcp协议进行交互，同时它支持http协议（json格式）、thrift、servlet、memcached、zeroMQ等的传输协议（通过插件方式集成）。

系统配置优化：

①在linux系统：
   /etc/security/limits.conf
   编辑该文件，后面加上：
   user soft nofile 65536
   user hard nofile 65536

  用vi 编辑bin/elasticsearch文件后面加入
  -Des.max-open-files=ture

②elasticsearch内存设置为：
   bootstrap.mlockall: true
   这样可以elasticsearch确保使用物理内存，不使用linux swap 。
   在
   /etc/security/limits.conf
   编辑该文件，后面加上：
   user -memlock unlimited

配置项说明：

配置文件的路径可以通过在外部的系统属性来进行设置:
elasticsearch -f -Des.config=/path/to/config/file
cluster.name: elasticsearch
配置es的集群名称，默认是elasticsearch，es会自动发现在同一网段下的es，如果在同一网段下有多个集群，就可以用这个属性来区分不同的集群。

node.name: "Franz Kafka"
节点名，默认随机指定一个name列表中名字，该列表在es的jar包中config文件夹里name.txt文件中，其中有很多作者添加的有趣名字。

node.master: true
指定该节点是否有资格被选举成为node，默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。

node.data: true
指定该节点是否存储索引数据，默认为true。

index.number_of_shards: 5
设置默认索引分片个数，默认为5片。

index.number_of_replicas: 1
设置默认索引副本个数，默认为1个副本。

path.conf: /path/to/conf
设置配置文件的存储路径，默认是es根目录下的config文件夹。

path.data: /path/to/data
设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开

path.data: /path/to/data1,/path/to/data2
path.work: /path/to/work
设置临时文件的存储路径，默认是es根目录下的work文件夹。

path.logs: /path/to/logs
设置日志文件的存储路径，默认是es根目录下的logs文件夹

path.plugins: /path/to/plugins
设置插件的存放路径，默认是es根目录下的plugins文件夹

bootstrap.mlockall: true
设置为true来锁住内存。因为当jvm开始swapping时es的效率会降低，所以要保证它不swap，可以把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es。同时也要允许elasticsearch的进程可以锁住内存，linux下可以通过`ulimit -l unlimited`命令。

network.bind_host: 192.168.0.1
设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0。

network.publish_host: 192.168.0.1
设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址。

network.host: 192.168.0.1
这个参数是用来同时设置bind_host和publish_host上面两个参数。

transport.tcp.port: 9300
设置节点间交互的tcp端口，默认是9300。

transport.tcp.compress: true
设置是否压缩tcp传输时的数据，默认为false，不压缩。

http.port: 9200
设置对外服务的http端口，默认为9200。

http.max_content_length: 100mb
设置内容的最大容量，默认100mb

http.enabled: false
是否使用http协议对外提供服务，默认为true，开启。

gateway.type: local
gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，hadoop的HDFS，和amazon的s3服务器，其它文件系统的设置方法下次再详细说。

gateway.recover_after_nodes: 1
设置集群中N个节点启动时进行数据恢复，默认为1。

gateway.recover_after_time: 5m
设置初始化数据恢复进程的超时时间，默认是5分钟。

gateway.expected_nodes: 2
设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。

cluster.routing.allocation.node_initial_primaries_recoveries: 4
初始化数据恢复时，并发恢复线程的个数，默认为4。

cluster.routing.allocation.node_concurrent_recoveries: 2
添加删除节点或负载均衡时并发恢复线程的个数，默认为4。

indices.recovery.max_size_per_sec: 0
设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。

indices.recovery.concurrent_streams: 5
设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。

discovery.zen.minimum_master_nodes: 1
设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）

discovery.zen.ping.timeout: 3s
设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。

discovery.zen.ping.multicast.enabled: false
设置是否打开多播发现节点，默认是true。

discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]"]
设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。
下面是一些查询时的慢日志参数设置
index.search.slowlog.level: TRACE
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.query.info: 5s
index.search.slowlog.threshold.query.debug: 2s
index.search.slowlog.threshold.query.trace: 500ms
index.search.slowlog.threshold.fetch.warn: 1s
index.search.slowlog.threshold.fetch.info: 800ms
index.search.slowlog.threshold.fetch.debug:500ms
index.search.slowlog.threshold.fetch.trace: 200ms
 

分片配置：
cluster.routing.allocation.allow_rebalance
设置根据集群中机器的状态来重新分配分片，可以设置为always, indices_primaries_active和indices_all_active，默认是设置成indices_all_active来减少集群初始启动时机器之间的交互。
cluster.routing.allocation.cluster_concurrent_rebalance
设置在集群中最大允许同时进行分片分布的个数，默认为2，也就是说整个集群最多有两个分片在
进行重新分布。
cluster.routing.allocation.node_initial_primaries_recoveries
设置指定初始每个节点。由于多数情况下是使用local的gateway，这应该会更快，
cluster.routing.allocation.node_concurrent_recoveries
设置在节点中最大允许同时进行分片分布的个数，默认为2
cluster.routing.allocation.disable_allocation
使主要分片或副本的分布失效。要知道，如果主分片不存在（那个节点挂了）那么其副本仍然会被提升为主分片，这个设置只有在动态地使用集群更新设置api调用时才生效。
cluster.routing.allocation.disable_replica_allocation
使副本分布失效。和上一个设置一样，只有动态地使用集群更新设置api调用时才生效。
indices.recovery.concurrent_streams
当从一个点（peer）恢复分片时当前节点最多允许的文件读取流的个数，默认为5

代码结构说明：


来源： <http://jpcloud.jd.com/pages/viewpage.action?pageId=14361918>
 



action：执行的操作；

bootstrap：启动服务

 bulk：批量操作

cache：缓存相关

client：客户端相关

cluster：集群相关

common：公共部分，包括Google的Guice

discovery：节点发现

env：环境配置相关

gateway：集群数据恢复相关。节点挂掉重启后，用于节点数据数据恢复

http：http访问处理方式

index：数据索引操作相关

indices：索引管理

monitor：监控相关

node：节点相关

percolator：过滤数据相关

plugins：插件相关

repositories：仓库？

rest：rest请求处理相关

river：其他数据源相关

script：脚本

search：检索

snashots：快照

threadpool：线程池：包括索引线程池、搜索线程池、批量操作和更新操作线程池

transport：启动交互

tribe：

watcher：观察



节点服务包含模块：

SettingsModule：设置参数模块
NodeModule：节点模块
NetworkModule：网络模块
NodeCacheModule：缓存模块
ScriptModule：脚本模块
JmxModule：jmx模块
EnvironmentModule：环境模块
NodeEnvironmentModule：节点环境模块
ClusterNameModule：集群名模块
ThreadPoolModule：线程池模块
DiscoveryModule：自动发现模块
ClusterModule：集群模块
RestModule：rest模块
TransportModule：tcp模块
HttpServerModule：http模块
RiversModule：river模块
IndicesModule：索引模块
SearchModule：搜索模块
ActionModule：行为模块
MonitorModule：监控模块
GatewayModule：持久化模块
NodeClientModule：客户端模块

 

索引属性：

创建索引时属性格式：

{
      "settings": {
             "index": {
                       "number_of_shards": 3,
                       "number_of_replicas": 2
                       }
              }
}

number_of_shards：索引占用的主分片数据。注意：一经确定，无法进行修改

number_of_replicas：分片副本数。每个主分片对应的副本数。可以动态修改

ttl：文档过期时间。默认不开启，需要主动开启，默认只存储，不索引、不分词。

             格式："_ttl" : { "enabled" : true, "default" : "1d" }。时间单位：w（星期）d（天），h（小时），m（分钟），s（秒），ms（毫秒）。default设置默认

timestamp:文档索引时间戳。默认不开启。开启后只存储、不索引、不分词。path：相当于字段名。format：时间戳格式。

            格式："_timestamp" : {"enabled" : true,"path" : "post_date","format" : "YYYY-MM-dd" }

size：文档大小。默认禁止，配置开启。为了使用，可以存储起来。

            格式："_size" : {"enabled" : true, "store" : "yes"}

index：能够存储在一个文件索引中。默认禁止，配置开启。"_index" : { "enabled" : true }

 

routing：配置路由字段，默认是id。当索引数据和明确路由控制是需要的，路由字段允许控制_routing方向。也就是_routing可以控制索引数据去哪里。

            格式："_routing" : {"required" : true,"path" : "blog.post_id"}。path表示按照字段名。

parent：父字段是定义在孩子上的映射，指向与子类型相关的父类型。例如，一种情况是blog类型和其子文档blog_tag类型，blog_tag映射应该如下：{"blog_tag" : {"_parent" : {"type" : "blog"}}

boost：处理过程中提高文档或字段的相关性。也可以通过设置boost，来提高文档得分。"_boost" : {"name" : "my_boost", "null_value" : 1.0}

 

analyzer：映射允许用一个文档的field属性作为分词(analyzer)的名字,它将被用来索引这个文档。当索引未明确指定analyzer 或者index_analyzer 时，analyzer 将被运用在任何field的属性中。

            格式："_analyzer" : {"path" : "my_field"}。所有字段未明确设定analyzer值，将会设置whitespace作为索引的analyzer。

all：在建立文档索引时，_all field表示包含一个或多个field。特别是对于搜索请求，我们要对一个文件的内容执行搜索查询，不知道哪些字段搜索，用_all field就会很方便。但是也会花销cpu使用和增加索引量。禁止使用_all                 field，它是一个很好的做法在设置index.query.default_field 为一个不同值。_all field有一个很好的特性是，它需要考虑到特定field 的boost，可以提高其boost的值。意味着标题的boost要高于content的boost，标题(部分)在          _all field中要高于内容(部分)在_all field。

格式：

	{
	    "person" : {
	        "_all" : {"enabled" : true},
	        "properties" : {
	            "name" : {
	                "type" : "object",
	                "dynamic" : false,
	                "properties" : {
	                    "first" : {"type" : "string", "store" : "yes", "include_in_all" : false},
	                    "last" : {"type" : "string", "index" : "not_analyzed"}
	                }
	            },
	            "address" : {
	                "type" : "object",
	                "include_in_all" : false,
	                "properties" : {
	                    "first" : {
	                        "properties" : {
	                            "location" : {"type" : "string", "store" : "yes", "index_name" : "firstLocation"}
	                        }
	                    },
	                    "last" : {
	                        "properties" : {
	                            "location" : {"type" : "string"}
	                        }
	                    }
	                }
	            },
	            "simple1" : {"type" : "long", "include_in_all" : true},
	            "simple2" : {"type" : "long", "include_in_all" : false}
	        }
	    }
	}

source：在用JSON建立文档索引时，_source field会自动生成存储。_source field没有被索引，所以搜索不到，它只是存储。虽然非常方便生成source field，但在索引时，source field字段会产生存储开销。它可以被禁止。

 "_source" : {"enabled" : false}。在存储source时，允许特定的路径可以被包含(存储)/排除(不存储)，支持*通配符注解。"_source" : { "includes" : ["path1.*", "path2.*"],"excludes" : ["pat3.*"]}
type：每个文档索引时有一个相关性的id和type。当索引时，type 会自动索引在_type field._type field是被索引的，但是没有被analyzed(分词),没有存储。这意味着_type field可以被查询。

     _type field也可以被存储。例如：{"tweet" : {"_type" : {"store" : "yes"}}}。_type field也可以不被索引：{"tweet" : {"_type" : {"index" : "no"}}}

id：每个文档索引都有一个id和type。_id field被运来索引id，可能还存储它。在默认情况下，它是不索引和不存储（好像最新版本是索引的，待验证。）。在没有被索引时，使用uid。

    _id field可以被索引和存储。{"tweet" : {"_id" : {"index": "not_analyzed", "store" : "yes"}}}。

    _id映射可以相关联一个路径。这里的路径是指的Field。配置为：{"tweet" : { "_id" : {"path" : "post_id"}}}，不过会稍微影响性能。

uid：_uid 是elasticsearch的保留字段。elasticsearch内部_uid是一个文档索引的惟一id。在索引建立过程中，基于type过滤时(_type没有被索引)，_uid 会自动被使用。也就是说 建立索引时_uid是必须的。基于type过滤时,_id不需要被索引。

multi_field：multi_field类型允许映射为几个有相同值的core_types。这样就很方便，例如，当想要映射一个字符串类型，它分析的时候既可以不分析字符串，又可以分析字符串。实现了一对多的关系。例如，字符串”abc def”,不分析的时候为”abc def”，分析的时候就为”abc”,”def”。例如：


	{
	    "tweet" : {
	        "properties" : {
	            "name" : {
	                "type" : "multi_field",
	                "fields" : {
	                    "name" : {"type" : "string", "index" : "analyzed"},
	                    "untouched" : {"type" : "string", "index" : "not_analyzed"}
	                }
	            }
	        }
	    }
	}

 multi_field的其他使用方式：accessing fields 访问字段，include in all，merging 待深入调研。

注意事项：

一、索引创建后，主分片是不能修改的，只能修改副本分片数。所以请正确评估索引数据大小后，分配合理的主分片数据。默认主分片为5，副本分片数为1。

二、优化GC方式，方式GC停顿造成discovery超时造成节点状态为失效。同时修改discovery重试次数稍微多一些。

三、防止OOM。ES对字段数据缓存是无限制的，而且数据操作都在内存当中操作，处理不好会出现OOM。

     ①缓存类型设置为软引用，利于GC回收

     ②设置缓存最大数据条数和有效时间

四、保持各个节点之间的jdk版本一致。

五、正确使用API，防止操作卡死。超过100条的数据处理请使用批量操作函数

六、

