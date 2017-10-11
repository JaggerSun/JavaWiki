先把git readme给看了
分布式高可用的搜索引擎
所有索引都部署在可配置的分片上
每一个分片至少有一个或多个副本
读和查询操作可以在任何的分片上
多租户多类型
索引配置
支持多个索引
支持多种类型的索引
多样的API
restful
java
面向文档
不需要预先定义schema
可靠的，异步写入持久化存储
接近于实时查询
基于Lucene
一致性， 但文档的操作是原子的
# Geting Start
##安装
    就是第一张说的那个
##索引
    让我们来模拟twitter创建索引
    curl -XPUT 
  curl -XGET
  curl -XPUT 'http://localhost:9200/twitter/user/kimchy?pretty' -d '{ "name" : "Shay Banon" }'。
  curl -XPUT 'http://localhost:9200/twitter/tweet/1?pretty' -d '
{
    "user": "kimchy",
    "post_date": "2009-11-15T13:12:00",
    "message": "Trying out Elasticsearch, so far so good?"
}'
    加了一个用户和一篇博文/_index/_type/_id可以理解为对应/库/表/行
    curl -XGET 'http://localhost:9200/twitter/user/kimchy?pretty=true'得到id对应的值 
    curl -XGET 'http://localhost:9200/twitter/tweet/_search?q=user:kimchy&pretty=true'
    search tweet表中所有user='kimchy'的
        返回的_source就是内容
    curl -XGET 'http://localhost:9200/twitter/tweet/_search?pretty=true' -d '
{
    "query" : {
        "match" : { "user": "kimchy" }
    }
}'
       curl -XGET 'http://localhost:9200/twitter/_search?pretty=true' -d '
{
    "query" : {
        "range" : {
            "post_date" : { "from" : "2009-11-15T13:00:00", "to" : "2009-11-15T14:00:00" }
        }
    } 
}'

使用json的格式来查找。

##体现多租户多类型
    可能有的索引过大，我们可以改变另外的维度创建索引
    比如刚才的结构还可以写成以个人维度来定义索引。
    curl -XPUT 'http://localhost:9200/kimchy/info/1?pretty' -d '{ "name" : "Shay Banon" }'
curl -XPUT 'http://localhost:9200/kimchy/tweet/1?pretty' -d '
{
    "user": "kimchy",
    "post_date": "2009-11-15T13:12:00",
    "message": "Trying out Elasticsearch, so far so good?"
}'
### 完全的控制，每个索引有几个分片几个镜像可以配置。
    设置一下：
curl -XPUT http://localhost:9200/another_user?pretty -d '
{
    "index" : {
        "number_of_shards" : 1,
        "number_of_replicas" : 1
    }
}'
    查询所有的index:
curl -XGET 'http://localhost:9200/_search?pretty=true' -d '
{
    "query" : {
        "match_all" : {}
    }
}'
## 分布式高可用
分片多副本
