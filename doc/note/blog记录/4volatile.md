##什么原因造成了线程安全问题
       首先需要把问题约束在同一个JVM进程中。因为现在包含了很多共享内存映射，分布式缓存，缓存服务器等相关的技术，会给跨进程的线程安全或者是说数据同步带来问题。
       按照Java的内存模型，线程将会私有PC计数器，虚拟机栈。 公用JVM堆内存。
       1. 当有线程并发的时候，因为多个线程可能操作同一块堆内存地址这样就会造成资源竞争引发线程安全问题。
       2. JVM server模式下对程序进行的优化：主要是重排序和执行时为了性能会把主存中的数据拷贝一份到线程自己的寄存器中，操作该对象并回写主内存。 这两种方式都会导致当前线程操作的数据对于其他线程来讲可见性方面出现问题，因此我统一称为可见性问题。
       3. JVM可能将long，double等64位数的操作转化为两个32位的操作。

###可见性问题
1. 拷贝：
上代码：
```
public class NoVisibility {  
    private boolean ready;  
      
    public void change(){  
        ready = true;  
    }  
      
    public boolean isReady() {  
        return ready;  
    }  
    public static void main(String[] args) throws InterruptedException {  
        NoVisibility nv = new NoVisibility();  
        ReadThread read = new ReadThread(nv);  
        read.start();  
        nv.change();  
    }  
}  
```
```
class ReadThread extends Thread {  
    private NoVisibility nv;  
    public ReadThread(NoVisibility nv) {  
        this.nv = nv;  
    }  
    @Override  
    public void run() {  
        while(!nv.isReady()){  
            Thread.yield();  
        }  
        System.out.println("");  
    }  
}  
```

这段代码在-server模式下，在某些硬件配置下的某些JVM实现下有可能永远不会退出。  也就是ReadThread读不到NoVisibily线程写入的ready值。也就是NoVisibily对ready的修改，对于ReadThread来讲不可见。
解决办法是声明ready为volatile的。如下：

	private boolean volatile ready; 
这里利用了volatile 的一个语义，是关闭寄存器拷贝优化，每一次都直接读写主内存。
另外一个办法使使用synchronize关键字修饰change方法：
	public synchronized void change() 

这两种办法稍有不同， volatile是直接读取主存，而synchronized是在解锁的时候保证寄存器回写主存，还是会利用到寄存器的

2. 重排序
这个比较好的例子是double check的单例模式：

```
public class DoubleCheckSingleton {  
    private static DoubleCheckSingleton instance;  
      
    public static DoubleCheckSingleton getInstance(){  
        if(instance == null){  
            synchronized (DoubleCheckSingleton.class) {  
                instance = new DoubleCheckSingleton();  
            }  
        }  
        return instance;  
    }  
}  
```

这个本来是一个线程安全的单例模式， 为了性能上的优化，又加了一个instance==null的判断。本来是极好的，但是因为有重排序的存在，就会存在问题。
重排序：
JVM在编译的时候会保证单线程模式下的结果是正确的，但是其中代码的顺序可能会进行重排序，或者乱序，主要是为了更好的利用多cpu资源(乱序)， 以及更好的利用寄存器， 比如1 a = 1; b = 2; a=3;三个语句，如果b执行的时候可能会占用a的寄存器位置，JVM可能会把a=3语句提到b=2前面，减少寄存器置换次数。
比如上面的 instance = new DoubleCheckSingleton()这部分代码的伪字节码为：

1. memory = allocate()    // 分配内存
2. init(memory)           // 初始化对象
3. instance = memory      // 实例指向刚才初始化的内存地址。
4. 第一次访问instance
在JVM的时候有可能2.3的位置进行了重新排序，因为JVM只保证构造器执行完之后的结果是正确的，但是执行顺序可能会有变化。  这个时候并发调用getInstance的时候就有可能出现如下的情况：

| 时间        | 线程A           | 线程B  |
| ------------- |:-------------:| -----:|
| t1     | A1：分配对象的内存空间 |  |
| t2      | A3：设置instance指向内存空间      |   |
| t3 |      |    B1：判断instance是否为空 |
| t4 |      |    B2：由于instance不为null，线程B将访问instance引用的对象 |
| t5 |  A2：初始化对象    |     |
| t6 |   A4：访问instance引用的对象   |     |
这样就返回了一个未初始化的对象，这样就出现了问题。解决的办法可以这样：

```
public class DoubleCheckSingleton {  
    private static DoubleCheckSingleton instance = new DoubleCheckSingleton();  
    public static DoubleCheckSingleton getInstance(){  
        return instance;  
    }  
}  
```
恶汉模式，  坏处就是在不需要的时候也会创建实例
```
public class DoubleCheckSingleton {  
    private static DoubleCheckSingleton instance;  
      
    public static DoubleCheckSingleton getInstance(){  
        if(instance == null){  
            synchronized (DoubleCheckSingleton.class) {  
                DoubleCheckSingleton temp = new DoubleCheckSingleton();  
                instance = temp;  
            }  
        }  
        return instance;  
    }  
}  
```
这种办法允许重排序但是重排序对B线程不可见
还可以这样：
```
public class DoubleCheckSingleton {  
    private static class InstanceHolder{  
        public static DoubleCheckSingleton instance = new DoubleCheckSingleton();  
    }  
      
    public static DoubleCheckSingleton getInstance(){  
        return InstanceHolder.instance;  
    }  
```

这种办法没有加锁，但是利用了JVM静态方法的特性，保证其是线程安全的，也是允许重排序但是重排序对B线程不可见。
下面还有一种不允许重排序，就是利用volatile关键字：

```
public class DoubleCheckSingleton {  
    private static volatile DoubleCheckSingleton instance;  
      
    public static DoubleCheckSingleton getInstance(){  
        if(instance == null){  
            synchronized (DoubleCheckSingleton.class) {  
                instance = new DoubleCheckSingleton();  
            }  
        }  
        return instance;  
    }  
}  
```
volatile关键字可以让JVM不进行重排序。就不会出现上诉的问题了。

总结下volatile关键字的用法：
其对于线程安全方面的控制很少，一般仅仅用来保证一个对象状态变化的可见性， 比如使用一个属性进行关键判断的时候，这个属性就应该使用volatile进行修饰，保证其他线程对该属性的修改能够及时可见。

复合操作和原子操作
原子操作表示对于两个操作A,B，对于A来讲B要么都执行了，要么都没执行。那么B对于A来讲就是原子操作。
原子操作是线程安全的，排除使用synchronized来使得一个操作是原子操作外，Java本身很多语句可以认为是原子的，比如volatile int i=1.
复合操作由多个操作构成，因为多个操作执行的时候会有时间差，这个时候就会产生线程安全的问题。 除了很明显的多个语句外，Java很多单条语句可能会变为多个操作，就是上面说的这几种情况了：
1. i=1的时候可以分解为 从主存read i 到寄存器，修改i， 回写主存这三步。
2. 重排序
3. long等64的运算，可能分解为多个运算。
对于这三个操作，使用volatile就可以是他们变成原子操作。

但是对于多语句的情况就只能使用锁的机制来进行同步了。  这个后面再说。

来源： <http://blog.csdn.net/three_man/article/details/42777187>