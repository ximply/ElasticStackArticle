# [原文链接](https://www.elastic.co/blog/hot-warm-architecture-in-elasticsearch-5-x)

# 译文
当elasticsearch用于大量实时数据分析的场景时，我们推荐使用基于时间的索引然后使用三种不同类型的节点（Master, Hot-Node 和 Warm-Node）进行结构分层，这就是所谓的"Hot-Warm"架构。每种节点有自己的任务，下面会进行介绍。
## Master节点
We recommend running 3 dedicated master nodes per cluster to provide the greatest resilience. When employing these, you should also have the discovery.zen.minimum_master_nodes setting at 2, which prevents the possibility of a "split-brain" scenario. Utilizing dedicated master nodes, responsible only for handling cluster management and state enhances overall stability. Because they do not contain data nor participate in search and indexing operations they do not experience the same demands on the JVM that may occur during heavy indexing or long, expensive searches. And therefore are not as likely to be affected by long garbage collection pauses. For this reason they can be provisioned with CPU, RAM and Disk configurations much lower than those that would be required for a data node. 
