http://my.oschina.net/feichexia/blog/196575
http://blog.csdn.net/lifuxiangcaohui/article/details/37992725
http://pengjiaheng.iteye.com/blog/552456
http://blog.csdn.net/fenglibing/article/details/6411932
http://blog.csdn.net/fenglibing/article/details/17323515
http://blog.csdn.net/zgmzyr/article/details/8445014
http://blog.csdn.net/lijiecong/article/details/6905011

jmap -dump:file=heap.bin 3172

问题起源(来自UMP的监控报警)

Vop-api部署情况
4台物理机
12个实例（每台3实例）
Jvm配置：8实例： -Xms1024m -Xmx1024m -XX:MaxPermSize=256m， 4实例： -Xms2048m –Xmx2048m -XX:MaxPermSize=256m
 
问题分析
线程分析
    工具：UMP、自动部署(jstack)
    通过jstack查看线程情况，http-1601-xx的线程数大概为160，tomcat配置（maxThreads="1000"minSpareTHreads="50"acceptCount="1000"），可以看出最大并发量并不高。
 
Jvm快照分析
工具：自动部署（jmap），MemoryAnalyzer
通过自动部署导出jvm信息，使用MemoryAnalyzer进行分析
注意：开启MemoryAnalyzer的Keep unreachable objects(因为此时并没有内存溢出，很多对象等待着被GC)
 
查看Top Consumers
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jvm-7.png)
通过OQL查看StringBuilder对象
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jvm-8.png)
发现有大量StringBuilder对象，这些对象的特点是类似“相似”，进一步分析这些数据的内容（这些对象处于unreachable状态，已经没有调用桟关系）——发现是商品池数据。
这些对象极有可能会被移动到老年代（实际上没必要）——通过观察YGC频率
 
同样的方式分析Arrays$ArrayList
……
问题处理
找到对应的代码：
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jvm-9.png)
说明：这里的List为List<String>,长度：1 ~ 10000，可以先考虑下有什么问题~~
 
修改后的代码：
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jvm-10.png)
修改的很简单~~
 
上线、发布（时间：08/25 18:00 ~18:20）
 
UMP观察
上线发布时间为：08/25 18:00 ~18:20
以下信息来自UMP监控，主要对比了修改前后的YGC，堆内存和CPU使用率
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jvm-11.png)
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jvm-12.png)
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/jvm-13.png)
看起来挺好：
YGC：从峰值：30至40下降到1，极少情况下出现2次
堆内存：从之前的400M至600M左右下降到200M至400M
CPU使用：从平均值1.113%下降至0.646%
以上来自UMP统计结果！
 
总结
对于jvm报警（或者叫预警），说明系统可能存在一些问题，这些问题不一定会立即影响到你的系统，当数据，并发等一些因素到一定程度后，才会影响到系统；
       Jvm报警阀值不宜设置过大（至少要有一个实例用于预警吧），否则很多情况下都不会有报警。
对于报警的系统，适当的把jvm信息导出分析下原因，是否满足预期——最近我们通过jvm信息排查到了几个问题。
 
Tk
