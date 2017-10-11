高可用： 宕机，磁盘坏，系统升级， 停机维护，集群扩容等都能访问。
##架构图
使用冗余的方式：服务器热备，数据多分存储，灵活失效转移。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/struts/s-34.png)

 - 应用程序服务器： 是访问端对系统发起数据操作请求。
 - 数据存储服务器： 他们是存储系统的核心，响应数据操作
 - 管理中心服务器：主-主热备的对数据服务进行心跳检测，扩容，故障恢复等。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/struts/s-35.png)

##故障处理
###正常模式
应用写 路由计算两台写， 读，随机选择一台读。
###瞬时故障
一般是因为网络问题，严重性较低。  通过重试可以解决
如果重试不行，则重新申请故障仲裁，重新判断故障种类

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/struts/s-36.png)

###临时故障
需要人为操作的故障

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/struts/s-37.png)
就是先存临时存储，然后恢复后再转存
###永久故障高可用解决方案
是服务器数据完全丢失。 这种没办法只能从一个好的服务器中重新获取得到新的数据。
