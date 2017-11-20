# [原文链接](https://www.elastic.co/blog/hot-warm-architecture-in-elasticsearch-5-x)

# 译文
当elasticsearch用于大量实时数据分析的场景时，我们推荐使用基于时间的索引然后使用三种不同类型的节点（Master, Hot-Node 和 Warm-Node）进行结构分层，这就是所谓的"Hot-Warm"架构。每种节点有自己的任务，下面会进行介绍。

### Master 节点
我们推荐每个集群运行三个专用的master节点来提供最好的弹性。使用时，你还得把 discovery.zen.minimum_master_nodes setting 设置为2，以免出现脑裂的情况。使用三个专用的master节点，专门负责处理集群的管理以及加强状态的整体稳定性。因为这三个master节点不包含数据也不会实际参与搜索以及索引操作，在JVM上它们不用做相同的事，例如处于繁重的索引或者耗时，资源耗费很大的搜索中。因此不太可能会因为垃圾回收而导致停顿。因此，我们可以配置比data节点少很多的CPU，内存以及磁盘。

### Hot 节点
指定的data节点会完成集群内所有的索引工作。这些节点同时还会保存近期的一些频繁被查询的索引。由于进行索引非常耗费CPU和IO，因此这些服务器需要强大的SSD存储来支撑。我们推荐部署最小化的三个Hot节点来保证高可用性。根据近期需要收集以及查询的数据量，可以增加服务器数量来获得想要的性能。

### Warm 节点
这种类型的节点是为了处理大量的而且不经常访问的只读索引而设计的。由于这些索引是只读的，warm 节点倾向于挂载大量磁盘（普通磁盘）来替代SSD。跟hot节点一样，我们建议部署最小化的三个warn节点来保证高可用性。然后跟之前一样地，数据量大的话还是需要额外的节点来达到性能要求。而且还需注意的是CPU和内存配置跟hot节点保持一致。通过测试一些类似生产环境中耗费比较大的查询可以确认这些东西。

Elasticsearch集群需要知道哪些服务器有hot节点以及哪些服务器有warm节点。这个可以通过分配所需的[属性](https://www.elastic.co/guide/en/elasticsearch/reference/5.1/allocation-awareness.html#forced-awareness)给服务器来实现。

例如，你可以在 elasticsearch.yml 这个配置文件中通过 node.attr.box_type: hot 把节点设置为hot，或者你也可以在启动节点时使用 ./bin/elasticsearch -Enode.attr.box_type=hot 参数指定。

box_type 这个属性字段你完全可以自定义成你要的。这些自定义的值用于告知 Elasticsearch 从哪里分配索引。

通过以下配置创建索引，我们可以确保今天的索引落在使用SSD的ho节点上：<br>
```Java
PUT /logs_2016-12-26
{
  "settings": {
    "index.routing.allocation.require.box_type": "hot"
  }
}
```

过几天之后如果索引不再需要在性能好的硬件上时，我们可以将这些节点标记成warm属性，更新索引配置如下：<br>
```Java
PUT /logs_2016-12-26/_settings 
{ 
  "settings": { 
    "index.routing.allocation.require.box_type": "warm"
  } 
}

```

那么现在我们可以使用logstash或者beats来实现：<br>
如果索引模板在logstash或者beats中管理，那么索引模板需要做一些更新，包括分配过滤器。"index.routing.allocation.require.box_type" : "hot" 这个配置会使新的索引创建在hot节点上。<br>
例如：<br>
```Java
{
  "template" : "indexname-*",
  "version" : 50001,
  "settings" : {
             "index.routing.allocation.require.box_type": "hot"
 ...
```
另外一个策略是给集群中的所有索引添加一个普通模板，在hot节点上 "template": "*" 模板可以生成新的索引。<br>
例如：<br>
```Java
{
  "template" : "*",
  "version" : 50001,
  "settings" : {
           "index.routing.allocation.require.box_type": "hot"
 ...
```


