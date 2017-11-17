# [原文链接](https://www.elastic.co/blog/hot-warm-architecture-in-elasticsearch-5-x)

# 译文
当elasticsearch用于大量实时数据分析的场景时，我们推荐使用基于时间的索引然后使用三种不同类型的节点（Master, Hot-Node 和 Warm-Node）进行结构分层，这就是所谓的"Hot-Warm"架构。每种节点有自己的任务，下面会进行介绍。
## Master节点
我们推荐每个集群运行三个专用的master节点来提供最好的弹性。使用时，你还得把discovery.zen.minimum_master_nodes setting设置为2，以免出现脑裂的情况。使用三个专用的master节点，专门负责处理集群的管理以及加强状态的整体稳定性。因为这三个master节点不包含数据也不会实际参与搜索以及索引操作，在JVM上它们不用做相同的事，例如处于繁重的索引或者耗时，资源耗费很大的搜索中。因此不太可能会因为垃圾回收而导致停顿。因此，我们可以配置比data节点少很多的CPU，内存以及磁盘。
