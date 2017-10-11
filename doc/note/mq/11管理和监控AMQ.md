##  The JMX API and ActiveMQ
几乎所有的有关于Java应用的都是基于JMX的， AMQ也不例外。
 broker统计信息   比如有多少个消费者
增加新的connect 或者是移除已经存在的
改变broker的配置   

### Local vs. remote JMX access
启动的时候是本地的JMX可以使用JConsole找到activemq查看。在MBean标签页查看。
但是是不能远程访问的。
可以这样子：打开远程访问
<broker xmlns="http://activemq.org/config/1.0" useJmx="true"
brokerName="localhost"
dataDirectory="${activemq.base}/data">
...
</broker>
### Exposing the JMX MBeans for ActiveMQ
<managementContext>
<managementContext connectorPort="2011" jmxDomainName="my-broker" />
</managementContext>
不设置这个默认的是：1099
### Exploring broker properties using the JMX API
直接使用jconsole  然后新建链接输入：service:jmx:rmi:///jndi/rmi://localhost:2011/jmxrmi
就搞定了。

下面用代码搞定my-broker:brokerName=localhost,type=Broker
见Stats.java

## Advanced JMX configuration
主要是配置访问权限的。

## AMQ管理工具
### Command-line tools
命令行除了启动AMQ，还有别的功能：
启动和停止borker
列出可用的broker
查询broker的状态
查看broker的destination
./bin/activemq-admin start
尝试通过JMX API来停止 ：
./bin/activemq-admin stop \
--jmxurl service:jmx:rmi:///jndi/rmi://localhost:2011/jmxrmi \
--jmxdomain my-broker

## Command agent
XMPP
##Broker logging
data/activemq.log 日志中进行配置的