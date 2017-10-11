http://www.flyne.org/article/190
（一） — Transactional topology
一个0.9版本中被弃用的原语，取而代之的是trident框架

为了防止重复技术
Storm通过保证每个tuple至少被处理一次来提供可靠的数据处理。关于这一点最常被问到的问题就是“既然tuple可能会被再次发送(replay), 那么我们怎么在storm上面做统计个数之类的事情呢？storm有可能会重复计数(overcount)吧？”

Storm 0.7.0引入了Transactional Topology, 它可以保证每个tuple”被且仅被处理一次”(exactly once), 这样你就可以实现一种非常准确、易于扩展、并且高度容错方式来实现计数类应用。

跟Distributed RPC类似，transactional topology其实不能算是storm的一个特性，它其实是用storm的底层原语spout, bolt, topology, stream等等抽象出来的一个特性。


（二） — DRPC
主要是利用storm的实时计算能力来并行化CPU密集型（CPU intensive）的计算任务。DRPC的storm topology以函数的参数流作为输入，而把这些函数调用的返回值作为topology的输出流。

从指定的数据源中查， 产生数据源的方式是 persistentAggregate或者自己创建一些数据源。
然后把查询结果聚合起来。暂时先看这么多。

http://www.flyne.org/article/199

（五） — State in Trident
你已经看到实现有且只有一次被执行的语义时的复杂性。Trident这样做的好处把所有容错想过的逻辑都放在了State里面 — 作为一个用户，你并不需要自己去处理复杂的txid，存储多余的信息到数据库中，或者是任何其他类似的事情。你只需要写如下这样简单的code：

http://www.flyne.org/article/222




其他的事务性的API暂时不看了：
http://ifeve.com/getting-started-of-storm8/
http://ifeve.com/storm-trident-tutorial/
http://ifeve.com/storm-trident-spouts/
http://ifeve.com/storm-trident-state/
http://www.tuicool.com/articles/Vnii6f



