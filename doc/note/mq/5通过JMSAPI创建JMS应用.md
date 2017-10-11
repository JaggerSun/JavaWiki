简单的JMS应用。
###简单的应用
  步骤：

 - 获取connection factory
 - 用factory创建一个conneciton
 - 用connection创建一个session
 - 创建一个JMS destination
 - 创建一个producer（可以是同步或者异步的listener）
 - 创建一个Consumer(同步或者异步的listener)
 - 发送和接收一个消息
 - 关闭所有的资源
关于JMS的并发。 协议规定。 Factory,Connection,destination支持并发访问。 而session,produer,consumer都不支持并发访问。