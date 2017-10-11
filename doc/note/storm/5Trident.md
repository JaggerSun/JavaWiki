是Storm的一种高度抽象的实时计算模型，  有联结、聚合、分组、函数以及过滤器的功能。
为数据库或者其他持久化存储上层的状态化、增量式处理提供了基础原语。
有一致性的、执行一次且仅仅执行一次的语义。

## Trident API:
topology.newStream("spout", spout)
  .each(new Fields("actor", "text"), new PerActorTweetsFilter("dave"))
  .each(new Fields("text", "actor"), new UppercaseFunction(), new Fields("uppercased_text"))
.each 对batch中的每一个tuple进行操作。
Filter的情况，主要是判断哪些能skip， 第一个参数为需要的字段，可以过滤掉不需要的字段
Function的情况， 第三个参数为追加的字段。
过滤字段并不会真的去掉，如果真的舍弃掉，可以调用.project(new Fields("b", "d"))
parallelismHint().  并行度， 有多少个线程同时执行。 可以在Filter中的prepare方法中通过context.getPartitionIndex();来得到线程的id
topology.newStream("spout", spout)
  .parallelismHint(2)
  .shuffle()
  .each(new Fields("actor", "text"), new PerActorTweetsFilter("dave"))
  .parallelismHint(5)
  .each(new Fields("actor", "text"), new Utils.PrintFilter());
上面用了两个Spout和5个Filter.
上面的shuffle()是重定向， 是tuple如何route到下一处理层，不同的处理层有不同的并行度。shuffle()是随机的。
partitionBy() 会按照一致性hash算法route, 指定字段名，字段值相同的tuple会被route到同一个线程中。
有很多种：
shuffle：通过随机分配算法来均衡tuple到各个分区
broadcast：每个tuple都被广播到所有的分区，这种方式在drcp时非常有用，比如在每个分区上做stateQuery
partitionBy：根据指定的字段列表进行划分，具体做法是用指定字段列表的hash值对分区个数做取模运算，确保相同字段列表的数据被划分到同一个分区
global：所有的tuple都被发送到一个分区，这个分区用来处理整个Stream
batchGlobal：一个Batch中的所有tuple都被发送到同一个分区，不同的Batch会去往不同的分区
Partition：通过一个自定义的分区函数来进行分区，这个自定义函数实现了 backtype.storm.grouping.CustomStreamGrouping

聚合：
.aggregate(new Fields("word"), new SumAggregator(), new Fields("count_map"))  
主要注意： init 接收到batch时执行
    aggregate 收到batch中的每一个tuple是执行
    complete 每个partition结束时执行
partitionAggregate(new Fields("word"), new SumAggregator(), new Fields("count_map"))    
            // 不是重定向操作了，partition比batch小的话，会有多个batch都在这个partition里面执行，因此现象就是同一个单词只会在一个SumAggregator中，并且会有多个单词的计数。
            // 截止到这里我们有两个batch，3个partition, init执行了6次，每个batch每个partition一次。统计执行了3次。每个partition一次。  
.groupBy(new Fields("word"))  
，它会根据指定的字段创建一个GroupedStream，相同字段的tuple都会被重定向到一起，汇聚成一个group。groupBy()之后是aggregate，与之前的聚合整个batch不同，此时的aggregate会单独聚合每个group。我们也可以这么认为，groupBy会把Stream按照指定字段分成一个个stream group，每个group就像一个batch一样被处理。
groupBy()本身并不是一个重定向操作，但如果它后面跟的是aggregator的话就是，跟的是partitionAggregate的话就不是。

Storm是一个实时流计算框架，Trident是对storm的一个更高层次的抽象，Trident最大的特点以batch的形式处理stream。
     一些最基本的操作函数有Filter、Function，Filter可以过滤掉tuple，Function可以修改tuple内容，输出0或多个tuple，并能把新增的字段追加到tuple后面。
     聚合有partitionAggregate和Aggregator接口。partitionAggregate对当前partition中的tuple进行聚合，它不是重定向操作。Aggregator有三个接口：CombinerAggregator, ReducerAggregator，Aggregator，它们属于重定向操作，它们会把stream重定向到一个partition中进行聚合操作。
     重定向操作会改变数据流向，但不会改变数据内容，重定向操会产生网络传输，可能影响一部分效率。而Filter、Function、partitionAggregate则属于本地操作，不会产生网络传输。
     聚合器：有三种：CombinerAggregator、ReducerAggregator 和 Aggregator
     CombinerAggregator， 每一个tuple都会执行一次init()， 然后会按照combine中的规则进行处理，处理完partition中的所有
   ReducerAggregator 
    
    Trident中有aggregate()和persistentAggregate()方法对流进行聚合操作。aggregate()在每个batch上独立的执行，persistemAggregate() 对所有batch中的所有tuple进行聚合，并将结果存入state源中。
aggregate()对流做全局聚合，当使用ReduceAggregator或者Aggregator聚合器时，流先被重新划分成一个大分区(仅有一个partition)，然后对这个partition做聚合操作；另外，当使用CombinerAggregator时，Trident首先对每个partition局部聚合，然后将所有这些partition重新划分到一个partition中，完成全局聚合。相比而言，CombinerAggregator更高效，推荐使用。


来源： http://www.flyne.org/article/216

     GroupBy会根据指定字段，把整个stream切分成一个个grouped stream，如果在grouped stream上做聚合操作，那么聚合就会发生在这些grouped stream上而不是整个batch。如果groupBy后面跟的是aggregator，则是聚合操作，如果跟的是partitionAggregate，则不是聚合操作。
    
http://www.bubuko.com/infodetail-467560.html

下面这个有点基础的看着就会感觉说的很好：    partition不再是线程而是分区。  汇总都是批次分区汇总，而不是单纯的批次汇总。
http://blog.csdn.net/opensure/article/details/45847545
