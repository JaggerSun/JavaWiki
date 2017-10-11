DiamondManager manager = new DefaultDiamondManager(group, dataId, new ManagerListener() {

public void receiveConfigInfo(String configInfo) {

// 客户端处理数据的逻辑

}

});


String configInfo = manager.getAvailableConfigInfomation(timeout);

全是http的接口。

看下接口， group和dataId唯一确定一个配置项。
Listener是回调函数。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/diamond-1.png)


服务器之间同步。
    写数据的时候先写数据库，然后写本地文件，然后通知所有server从数据库中拉取进行配置进行同步。
每个server都会每隔一个较长的时间从数据库中做全量的拉取
保证了最终一致性

Client获取Server重的配置信息
    每个服务器都会维护一个服务器列表，然后客户端拉取到之后访问随机一台
在本地维护了缓存，过期时间外的请求都返回缓存内的
读取数据时直接读取的是server端本地的值，减少对数据库的访问。
通过配置可以直接拿数据库中

容灾能力：
    
数据库主库不可用，可以切换到备库，Diamond继续提供服务
数据库主备库全部不可用，Diamond通过本地缓存可以继续提供读服务
数据库主备库全部不可用，Diamond服务端全部不可用，Diamond客户端使用缓存目录继续运行，支持离线启动
数据库主备库全部不可用，Diamond服务端全部不可用，Diamond客户端缓存数据被删，可以通过拷贝备份的缓存目录到容灾目录下继续使用

客户端采用推拉结合的策略在长连接和短连接之间取得一个平衡，让服务端不用太关注连接的管理，又可以获得长连接的及时性。
客户端发起一个对比请求到服务端，请求中包含客户端订阅的数据的指纹
服务端检查客户端的指纹是否与最新数据匹配
如果匹配，服务端持有连接
如果30秒内没有相关数据变化，服务端持有连接30秒后释放
如果30秒内有相关数据变化，服务端立即返回变化数据的ID
如果不匹配，立即返回变化数据的ID
客户端根据变化数据的ID去服务端获取最新的内容
来源： http://www.codeweblog.com/diamond-%E9%98%BF%E9%87%8C%E7%9A%84%E9%85%8D%E7%BD%AE%E6%9C%8D%E5%8A%A1/

我们自己的实现：
提供client和server端。
提供创建项目， 每个项目创建profile的和属性值的功能
提供多级缓存
提供回调
提供接口可以直接拿数据库中的值
