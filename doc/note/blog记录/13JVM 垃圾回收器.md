1:新生代串行收集器：(默认收集器)
     算法：复制算法
     -XX:+UseSerialGC 指定使用新生代串行收集器和老年代串行收集器
     优点：效率高，久经考验
     缺点：串行，如果回收对象过多，或者堆过大，停顿时间会过长。

2：老年代串行收集器（cms收集器的备选）
     算法：标记-压缩算法
     -XX:+UseSerialGC:指定新生代串行收集器和老年代串行收集器
     -XX:+UseParNewGc:新生代使用并行收集器和老年代使用串行收集器
     -XX:+UseParallelGc:新生代使用并行回收收集器和老年代使用串行收集器


3：并行收集器
     算法：复制算法
     工作在新生代的垃圾收集器，简单的将串行回收器多线程化，策略和串行一样。也是独占式的。在多cpu环境下会比串行收集器好，停顿时间短。
     -XX:+UseParNewGC:新生代使用并行收集器，老年代使用串行回收器
     -XX:+UseConcMarkSweepGC：新生代使用并行收集器，老年代使用cms

     线程数量：
     -XX:ParallelGCThreads指定，一般最好与cpu数量相当。cpu小于8个时和cpu数量一样，cpu大于8个时 3+[(5*cpu/8)]

4：新生代并行回收收集器（Parallel Scavenge）
     算法：复制算法
     和并行收集器一样，区别在于这个收集器关注系统吞吐量
     -XX:+UseParallelGC:新生代使用并行回收收集器，老年代使用串行收集器
     -XX:+UseParallelOldGC:新生代和老年代都使用并行回收收集器

     吞吐量设置：
     -XX:MaxGCPauseMillis：设置停顿时间不超过多少。大于0的整数，收集器会调整对的大小或者其他一些参数，使得垃圾收集停顿时间控制在设置的时间

     -XX:GCTimeRatio:设置吞吐量大小，0~100之间的整数 公式：1/(1+n),系统将花费不超过这个时间来用于垃圾收集。

     -XX：useAdaptiveSizePolicy:打开自适应gc策略，新生代大小，eden和servivor的比例，晋升老年代的对象年龄都会自动调整以达到对大小，吞吐量和停顿时间之间的平衡。

5：老年代并行回收收集器
     算法：标记压缩算法
     和新生代并行回收器一样，它也是一种关注吞吐量的收集器

     -XX:+UseParallelOldGC：新生代和老年代都使用并行回收收集器


6：cms收集器
     算法：标记-清除算法
     和并行回收收集器的区别是：注重系统停顿时间
     
     工作步骤：
     初始标记：独占资源
     并发标记：非独占资源
     重新标记：独占资源
     并发清除：非独占资源
     并发重置：非独占资源
     -XX:ParallelGCThreads:设置cms的线程数量，默认启动线程数是：(ParallelGCThreads+3/4)
     -XX:CMSInitiatingOccupancyFraction:设置当老年代空间实用率达到百分比值时进行一次cms回收

     因为使用标记清除算法，所以长时间后会有碎片产生
     -XX:+UseCMSCompactAtFullCollection:设置cms在垃圾收集完成后进行一次内存碎片整理
     -XX:CMSFullGCsBeforeCompaction:设定进行多少次cms回收后，进行一次内存压缩。

     应为cms不停止应用程序，所以在cms回收过程中，可能因为内存不足而导致回收失败，失败的话会启动老年代串行收集器进行垃圾回收，这样应用程序将完全中断，这时停顿时间可能会很长。可以通过设置-XX:CMSInitiatingOccupancyFraction来解决


7：G1收集器（garbage first） jdk1.6update14才提供预览版，jdk1.7才发布






总结：
     在众多的垃圾回收器中，没有最好的，只有最适合应用的回收器，根据应用软件的特性以及硬件平台的特点，选择不同的垃圾回收器，才能有效的提高系统性能。


jvm调优思路与方法：

1：将新对象预留在新生代
     一般来说，当survivor区空间不够，或者占用量达到50%时，就会将对象进入老年代（不管对象年龄有多大）

2：大对象进入老年代
     开发中要避免短命的大对象，目前没有特别好的方法回收短命大对象，大对象最好直接进入老年区，因为大对象在新生区，占用空间大，会由于空间不足而导致很多小对象进入到老年区。
 -XX：PretenureSizeThreshold：设置大对象直接进入老年代的阈值，当对象的大小超过这个值将直接分配在老年代

3：设置对象进入老年代 的年龄：
  -XX:MaxTenuringThreshold:设置对象进入老年代的年龄，默认值时15，但是如果空间不够，还是会将对象移到老年代。

4：稳定与震荡的堆大小

     稳定的堆大小能减少gc次数，但是每次gc时间增加
     震荡的堆大小能增加gc次数，但是每次gc时间减少

     -XX:MinHeapFreeRatio:最小空闲比例，当堆空间空闲内存小于这个比例，则扩展
     -XX:ManHeapFreeRatio：最大空闲比例，当堆空间空闲内存大于这个比例，则压缩

     -Xms和-Xmx相等时，上面的参数失效


5:吞吐量方案：
     4G内存32核吞吐量优先方案：尽可能减少系统的执行垃圾回收的总时间，考虑使用关注吞吐量的并行回收收集器。
     -Xms:3800 
     -Xmx:3800
     -Xss:128k //减少线程栈大小，使剩余系统内存支持更多线程
     -Xmn:2g //设置新生代大小
     -XX:UseParallelGC:新生代并行回收收集器
     -XX:ParallelGCTHreads:设置线程数
     -XX:+UseParallelOldGC：老年代也使用并行回收收集器
     

6:使用大页案例
      -Xmx:2506 
     -Xms:2506
     -Xss:128k //减少线程栈大小，使剩余系统内存支持更多线程
     -XX:UseParallelGC:新生代并行回收收集器
     -XX:ParallelGCTHreads:20
     -XX:+UseParallelOldGC：老年代也使用并行回收收集器
     -XX:LargePageSizeInBytes=256m


7：降低停顿案例:
   降低停顿首先考虑的是使用关注系统停顿的cms回收器，其次为了减少fullgc次数，应尽可能将对象预留在新生代，因为新生代minorgc的成本远小于老年代的fullgc

     -Xms:3550
     -Xmx:3550
     -Xss:128k //减少线程栈大小，使剩余系统内存支持更多线程
     -Xmn:2g //设置新生代大小
     -XX:ParallelGCThreads:20
     -XX:+UseConcMarkSweepGC //老年代使用cms回收器
     -XX:+UseParNewGC //新生代使用并行回收器
     -XX:SurvivorRatio=8 //设置eden和survivor比例为8:1
     -XX:TargetSurvivorRatio=90 //设置survivor使用率
     -XX:MaxTenuringThreshold=31 //年轻对象进入老年代的年龄，默认是15，这里是31

来源： <http://blog.csdn.net/jiushuai/article/details/9762997>
 