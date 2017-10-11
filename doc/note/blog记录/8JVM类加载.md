类加载过程
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/blog/blog-3.png)

主要分为了 加载->链接(验证->准备->解析)->初始化->使用->卸载这几个阶段。
加载
三件事
1. 通过类的权限定名称来获取定义此类的二进制字节流(可以是文件，网络，数据库，动态等等等等)
2. 把类的结构放在方法区中
3. 创建Class对象作为访问入口
验证
主要包括了字节码验证，元数据验证(这部分在编译期间基本上避免了)，类文件格式验证。
准备
包括了内存分配和类变量(static)初始值的设定，以及常量池的写入。
解析
主要是解析符号引用和直接引用，符号引用就是个字面量，直接引用是真正的内存地址。
初始化
就是执行<clinit>()方法的过程
它是 由类变量赋值和static代码块合并成的
会先执行父类的clinit方法。
JVM会保证clinit方法的线程安全，比如写一个static的长执行方法，两个线程调用的时候会看到等待过程
初始化触发的时间点  new , 引用类的final字段，但是staticfinal的除外，  调用类的静态方法。  用reflect包反射调用，执行时main方法宿主类， 子类初始化先初始化父类

类加载器
不同的类加载器使用instanceof的时候会判断两个相同的Class生成的obj是返回false的。
双亲委派模型
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/blog/blog-4.png)
每个类加载器的功能图上已经说得很清楚了。
说是双亲委派模型主要是说得1.2之后的所有的类加载器的实现的模式都是
实现的时候就是我们自己去实现findClass（）方法，loadClass()方法里的逻辑是先调用了父类类加载器，当父类无法加载的时候调用findClass()方法。

破坏双亲委派模型
双亲的模型会有一个问题，默认了用户类会调用基础类，但是有些情况不是这样的，比如JNDI，JNDI是由启动加载器加载的，但是其确需要引用一些第三方的类，为了解决类似的情况，引入了ThreadContextClassLoader. JNDI, JDBC等都使用了这种情况

此外，为了热部署的OSGI技术也是一个没有使用这个结构的代表。


Tomcat类加载器架构
一个Web容器设计类加载的时候可能会遇到的问题：
1. 每个应用应该相互隔离， 因为可能用到了同一个类的不同版本
2. 各应用所使用的类库又应该可以共享，这样就不用每一个都拷贝一份servelt.jar jsp.jar等等
3. 服务器本身的类库应该与应用的类库相互隔离
4. 应该支持JSP生成类的热替换
所以tomcat的类加载器设计成这个样子的：
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/blog/blog-5.png)
可以看出，出了JDK自己的加载器外，还有tomcat设计的类加载的目录
/common  可以被Tomcat和所有的Web应用共同使用的
/server   可以被Tomcat使用，不能被Web应用使用
/shared 被所有Web程序共同使用
/WebApp/WEB-INF/ 仅此Web应用可见
这样的类设计可以很好的实现上述的功能

OSGI灵活的类加载器
模块化技术的事实标准。
OSGI给每一个bundle使用了一个类加载器，当需要更换bundle的时候连同类加载器一起换掉。
比如BundleA对应了ClassLoaderA, 其通过Export到处了一个包，  这个包被BundleB所引用。
bundle自己的类加载器负责加载自己的类，以及调用启动类加载器加载rt.jar等公用内容。
而当bundleB需要加载Import A的包的时候就会委派给ClassLoaderA进行加载，因为不在同一个ClassLoader中同名类是可以共存的， 这样就保证了可以同时存在不同版本的jar包在开发环境中。
其类加载器的结构

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/blog/blog-6.png)
对于OSGI还有很多需要学习的地方

来源： <http://blog.csdn.net/three_man/article/details/39964329>