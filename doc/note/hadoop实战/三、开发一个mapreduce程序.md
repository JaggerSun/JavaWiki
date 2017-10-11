尝试搞明白一个大数据的框架具体怎么搭建，没有找到特别直接的方式。

大概了解到是用spoot来做mysql到nosql数据转换，然后用hive sql来直接转为job使用hadoop来查询的模式。  不是很清晰，最后准备还是先把所有的技术看一遍再说。

Configuration conf = new Configuration();
conf.addResource("tm/core-site.xml");
System.out.println(conf.get("fs.defaultFS"));

一些配置，可以add多个资源文件，会被覆盖掉。 但是配置了final的不会被覆盖掉。

为了用web来看任务，
最开始是说默认配置的本地模式，所以加上yarn等的配置，但是还是不行。
hadoop dfsadmin -safemode leave
最后只能放弃
改成集群模式就不行了linux和windows都不行。目前没有找到解决办法。

先继续看文档吧。

突然之间苦尽甘来了。
日死他大爷的，居然是服务器名称不能有下划线。。。。
又发现了个问题，居然是jquerynotfound自己换了个js上去。 在hadoop-yarn-common-2.7.2.jar中


