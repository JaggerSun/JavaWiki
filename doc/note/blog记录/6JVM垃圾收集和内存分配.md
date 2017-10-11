1. 垃圾判断算法
引用计数法，    给对象添加引用计数器，每有一个引用则+1, 没有则-1,为0为已死。python就是使用这种算法。  但是不能解决循环引用的问题。
根搜索算法。     从根开始向下搜索，如果有对象到根不可达则为死对象。 HopSpot使用的是这种算法。  
这里的根可以是栈中的引用对象， 方法区常量的引用对象，方法区静态属性的引用对象，JNI的引用对象。
两次标记。   虚拟机在回收对象时要做两次标记才会真的回收。无根的时候是一次，执行了finallise方法的时候会再标记一次。标记两次后，再执行到finalise方法就会回收对象。
回收方法区。 会回收常量池和做类卸载，可以使用-verbose:class来查看类的装载及卸载情况。
2. 垃圾收集算法
标记清除算法。  标记出所有需要回收的对象，标记完成后清除所有标记了的对象。   问题在于:效率不高，空间上会产生大量的碎片
复制算法。  把内存分为两个等块， 每次使用一块，当这一块满了之后把存活的对象转移到另外一块上去，然后清空这一块。  这样有点空间浪费。
HopSpot 的新生代基于这个算法，把内存分为了一个较大的Eden空间和两个较小的Survior空间。每次使用Eden和一个Survior，GC的时候把他们两个中存活的对象拷贝到另一个S空间，清空Eden和S1, 如果S2不够大，则会把对象复制到老生代。
标记整理                  当系统运行比较久后，对象的存活率比较高，复制算法就会有很多的复制操作，效率将会变低。因此对于老生代，使用标记-真理算法。
把存活对象标记并向一个方向清理，完成后把边界外的统一清空。
基于复制算法的新生代，标记整理的老生代，也就是分代收集的算法了。新生代GC(Minor GC)， 老生代GC(Major GC)
3. 垃圾收集器
有很多种，主要是针对问题的不同，有单线程的，为了减少停顿时间的，还有使用不同算法实现的。 目前最新的叫做G1
4. 内存分配
优先在Eden上分配
```
-verbose:gc -Xms20m -Xmx20m -Xmn10m -XX:SurvivorRatio=8 -XX:+PrintGCDetails  
```
-verbose:gc 打印GC的信息
-Xmn:新生代大小
-XX:Sur... : Eden和S的比
-XX:+Print.. : 打印详细的GC信息

代码:
	public class MemoryAllocation {  
	    private static final int MB = 1024 * 1024;  
	      
	    public static void main(String[] args) {  
	        byte[] allocation1 = new byte[2 * MB];  
	        byte[] allocation2 = new byte[2 * MB];  
	        byte[] allocation3 = new byte[2 * MB];  
	        byte[] allocation4 = new byte[4 * MB];  
	    }  
	}  
输出：

	[GC [DefNew: 6487K->148K(9216K), 0.0037546 secs] 6487K->6292K(19456K), 0.0037798 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]   
	Heap  
	def new generation   total 9216K, used 4572K [0x35c10000, 0x36610000, 0x36610000)  
	eden space 8192K,  54% used [0x35c10000, 0x36061fa8, 0x36410000)  
	from space 1024K,  14% used [0x36510000, 0x365352b8, 0x36610000)  
	to   space 1024K,   0% used [0x36410000, 0x36410000, 0x36510000) 
	tenured generation   total 10240K, used 6144K [0x36610000, 0x37010000, 0x37010000)  
	the space 10240K,  60% used [0x36610000, 0x36c10030, 0x36c10200, 0x37010000)
	compacting perm gen  total 12288K, used 378K [0x37010000, 0x37c10000, 0x3b010000)  
	the space 12288K,   3% used [0x37010000, 0x3706e8a0, 0x3706ea00, 0x37c10000)  
    ro space 10240K,  54% used [0x3b010000, 0x3b58ee00, 0x3b58ee00, 0x3ba10000)  
    rw space 12288K,  55% used [0x3ba10000, 0x3c0b2800, 0x3c0b2800, 0x3c610000)  

大概的过程是，前三个对象分配在Eden上，当分配第四个的时候发现Eden不够大， 因此做GC，做GC的时候发现对象都比S大，因此分配到老生代去了。
最后将是Eden上对象4，  老生代上有1,2,3

大对象直接进入老生代
上面那个例子，加入参数-XX:PertenureSizeThreshold参数指定大于这个值的对象会直接进入老生代。

长期存活的对象将进入老生代
每熬过一次MinorGC年龄增加一岁，  -XX:MaxTenuringThreshold设置几岁的对象进入老生代。

大于平均值的进入老生代
当处于一个年龄的对象达到了数目的一半以上，那么比这个年龄大的将会进入到老生代。

来源： <http://blog.csdn.net/three_man/article/details/39897049>
