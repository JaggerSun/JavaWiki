这个东西好啊。
fluent风格的接口。
它实现了锁， barriers, 缓存。
还提供了命名空间，自动重连，和其他一些功能。

###创建链接：
CuratorFramework zkc =
CuratorFrameworkFactory.newClient(connectString, retryPolicy);
connectString是一个连接的列表
retryPolicy标示重试策略。

### Fluent API
zk.create("/mypath",
new byte[0],
ZooDefs.Ids.OPEN_ACL_UNSAFE,
CreateMode.PERSISTENT);
zkc.create().withMode(CreateMode.PERSISTENT).forPath("/mypath", new byte[0]);
显然地下的这种看这爽多了。方法返回一个builder.
异步的话就需要这样：
zkc.create().inBackground().withMode(CreateMode.PERSISTENT).forPath("/mypath",
new byte[0]);

### Listener
CuratorListener masterListener = new CuratorListener() {
public void eventReceived(CuratorFramework client, CuratorEvent event) {
try {
switch (event.getType()) {
case CHILDREN:
下一步注册监听器
client = CuratorFrameworkFactory.newClient(hostPort, retryPolicy);
client.getCuratorListenable().addListener(masterListener);

此外还有一种后台处理的时候出现异常的时候使用的监听器：
UnhandledErrorListener errorsListener = new UnhandledErrorListener() {
client.getUnhandledErrorListenable().addListener(errorsListener);

他解决了边界情况，比如create的时候断开连接了，但是不知道是否断开了，他自己回去校验是否创建了
同样的删除操作也是这样。

## 食谱
已经实现了三张非常常用的场景： leader锁， leader选举， 子路径缓存。
###Leader latch
leaderLatch = new LeaderLatch(client, "/master", myId);
myId是当前这台master的id. 
 为了执行这台master获得leader或者是去leader之后的处理，需要实现一个listener



