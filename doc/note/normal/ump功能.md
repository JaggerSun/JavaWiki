端口存活监控         主要监控系统是否启动了，配置规则： 连续三次检测不存活告警。
url监控                  检查url联通性，这个也主要是检查存活的，不存活告警。
JVM监控                JVM内存使用情况图标，minor和full gc频率
方法监控                方法调用次数， tpXX指标，  告警连续多少次超过多少ms则告警， 可用率指标， 定期执行指标，单位时间内执行次数指标。
服务器监控            内存，磁盘，网卡，cpu使用率    这个已经可以使用zabbix实现了

http://www.aliyun.com/product/ecs/sla
http://blog.csdn.net/scutshuxue/article/details/8351810
