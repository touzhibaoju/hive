- [快速了解hive](https://mp.weixin.qq.com/s?__biz=MzI1NjM1ODEyMg==&mid=2247484038&idx=1&sn=5b26a4540dcce9e3d6ed358abf45308e&chksm=ea26a103dd5128158efda10d0cf61fb12c40059e2ad09409af827fc3eb772d2491edb3f0d1b2&mpshare=1&scene=23&srcid=0426tljMfZpOrYXqxj7a22T6#rd)
- [Hive窗口函数进阶指南](https://mp.weixin.qq.com/s?__biz=MzI1NjM1ODEyMg==&mid=2247484046&idx=1&sn=8b9bf86f742652ee6595de388af80424&chksm=ea26a10bdd51281d7fb5f9d5e8648eea7ba68d09e9bf271e9b9338bf36f5a36b27beca2dc77c&mpshare=1&scene=23&srcid=0429v25xh5i4SjpbqOb7lY7e#rd)
- [数据分析师之快速掌握SQL基础](<https://mp.weixin.qq.com/s?__biz=MzI1NjM1ODEyMg==&mid=2247483959&idx=1&sn=23f9ee016dca61c2e78a5b6e8f8b1eae&chksm=ea26a1b2dd5128a4bcd8a3d425876e8ec8ed2698899fb12c60a128dbad2d5eb7d68ab3998b76&scene=21#wechat_redirect>)
- [Hive on Spark新增的参数介绍](https://www.iteblog.com/archives/1541.html)

```
1.定义:
Facebook为了解决海量日志数据的分析而开发了hive，后来开源给了Apache基金会组织。hive是一种用SQL语句来协助读写、管理存储在HDFS上的大数据集的数据仓库软件。
2.特点:
▪ hive最大的特点是通过类SQL来分析大数据，而避免了写mapreduce Java程序来分析数据，这样使得分析数据更容易。 
▪ 数据是存储在HDFS上的，hive本身并不提供数据的存储功能 
▪ hive是将数据映射成数据库和一张张的表，库和表的元数据信息一般存在关系型数据库上（比如MySQL）。 
▪ 数据存储方面：他能够存储很大的数据集，并且对数据完整性、格式要求并不严格。 
▪ 数据处理方面：不适用于实时计算和响应，使用于离线分析。
```
![hive架构图](94C484D676B340CBAFB17EA60F1777E5)
内核
```
hive 的内核是驱动引擎，驱动引擎由四部分组成，这四部分分别是：
▪ 解释器：解释器的作用是将hiveSQL语句转换为语法树（AST）。 
▪ 编译器：编译器是将语法树编译为逻辑执行计划。 
▪ 优化器：优化器是对逻辑执行计划进行优化。 
▪ 执行器：执行器是调用底层的运行框架执行逻辑执行计划。
```
底层存储
```
hive的数据是存储在HDFS上，hive中的库和表可以看做是对HDFS上数据做的一个映射，所以hive必须运行在一个hadoop集群上。
```
程序执行过程
```
hive执行器是将最终要执行的mapreduce程序放到YARN上以一系列job的方式去执行。
```
元数据存储
```
hive的元数据是一般是存储在MySQL这种关系型数据库上的，hive与MySQL之间通过 MetaStore服务交互。

```
| Owner          | 库、表的所有者         | CreateTime  | 创建时间     |
| -------------- | ---------------------- | ----------- | ------------ |
| LastAccessTime | 最近修改时间           | Location    | 存储位置     |
| Table Type     | 表类型(内部表、外部表) | Table Field | 表的字段信息 |

客户端
```
hive有很多种客户端，下边简单列举了几个：
▪ cli命令行客户端：采用交互窗口，用hive命令行和hive进行通信。 
▪ hiveServer2客户端：用Thrift协议进行通信，Thrift是不同语言之间的转换器，是连接不同语言程序间的协议，通过JDBC或者ODBC去访问hive(这个是目前hive官网推荐使用的连接方式)。 
▪ HWI客户端：hive自带的客户端，但是比较粗糙，一般不用。 
▪ HUE客户端：通过Web页面来和hive进行交互，使用比较，一般在CDH中进行集成。
```

优缺点
```
优点：
1）操作接口采用类 SQL 语法，提供快速开发的能力（简单、容易上手）
2）避免了去写 MapReduce，减少开发人员的学习成本。
3）Hive 的执行延迟比较高，因此 Hive 常用于数据分析，对实时性要求不高的场合；
4）Hive 优势在于处理大数据，对于处理小数据没有优势，因为 Hive 的执行延迟比较高。
5）Hive 支持用户自定义函数，用户可以根据自己的需求来实现自己的函数。
缺点：
1）Hive 的 HQL 表达能力有限
（1）迭代式算法无法表达
（2）数据挖掘方面不擅长
2）Hive 的效率比较低
（1）Hive 自动生成的 MapReduce 作业，通常情况下不够智能化
（2）Hive 调优比较困难，粒度较粗
```
Hive调优
```
hadoop就像吞吐量巨大的轮船，启动开销大，如果每次只做小数量的输入输出，利用率将会很低。所以用好hadoop的首要任务是增大每次任务所搭载的数据量。hive优化时，把hive Sql当做mapreduce 程序来读，而不是当做SQL来读。hive优化这要从三个层面进行，分别是基于mapreduce优化、hive架构层优化和hiveQL层优化。
```

1. 基于mapreduce优化
1.1 合理设置map数
```
上面的mapreduce执行过程部分介绍了，在执行map函数之前会先将HDFS上文件进行分片，得到的分片做为map函数的输入，所以map数量取决于map的输入分片(inputsplit)，一个输入分片对应于一个map task，输入分片由三个参数决定，如表3：
```
| 参数名                   | 默认值 | 备注               |
| ------------------------ | ------ | ------------------ |
| dfs.block.size           | 128M   | HDFS上数据块的大小 |
| mapreduce.min.split.size | 0      | 最小分片数         |
| mapreduce.max.split.size | 256M   | 最大分片数         |
```
公式：分片大小=max(mapreduce.min.split.size,min(dfs.block.size, mapreduce.max.split.size)),默认情况下分片大小和dfs.block.size是一致的，即一个HDFS数据块对应一个输入分片，对应一个map task。这时候一个map task中只处理一台机器上的一个数据块，不需要将数据跨网络传输，提高了数据处理速度。
```
1.2 合理设置reduce数
决定 reduce 数量的相关参数如表4：
| 参数名                               | 默认值 | 备注                                                         |
| ------------------------------------ | ------ | ------------------------------------------------------------ |
| hive.exec.reducers.bytes.per.reducer | 1G     | 一个reduce数据量的大小                                       |
| hive.exec.reducers.max               | 999    | hive 最大的个数                                              |
| mapred.reduce.tasks                  | -1     | reduce task 的个数,-1 是根据hive.exec.reducers.bytes.per.reducer 自动调整 |
```
所以可以用set mapred.reduce.tasks手动调整reduce task个数。
```
2. hive架构层优化
2.1 不执行mapreduce

hive从HDFS读取数据，有两种方式：启用mapreduce读取、直接抓取。
```
set hive.fetch.task.conversion=more

hive.fetch.task.conversion参数设置成more，可以在 select、where 、limit 时启用直接抓取方式，能明显提升查询速度。
```
2.2 本地执行mapreduce
```
hive在集群上查询时，默认是在集群上N台机器上运行，需要多个机器进行协调运行，这个方式很好地解决了大数据量的查询问题。但是当hive查询处理的数据量比较小时，其实没有必要启动分布式模式去执行，因为以分布式方式执行就涉及到跨网络传输、多节点协调等，并且消耗资源。这个时间可以只使用本地模式来执行mapreduce  job，只在一台机器上执行，速度会很快。
```
2.3 JVM重用
```
因为hive语句最终要转换为一系列的mapreduce job的，而每一个mapreduce job是由一系列的map task和Reduce task组成的，默认情况下，mapreduce中一个map task或者一个Reduce task就会启动一个JVM进程，一个task执行完毕后，JVM进程就退出。这样如果任务花费时间很短，又要多次启动JVM的情况下，JVM的启动时间会变成一个比较大的消耗，这个时候，就可以通过重用JVM来解决。

set mapred.job.reuse.jvm.num.tasks=5

这个设置就是制定一个jvm进程在运行多次任务之后再退出，这样一来，节约了很多的 JVM的启动时间。
```

2.4 并行化
```
一个hive sql语句可能会转为多个mapreduce job，每一个job就是一个stage，这些 job 顺序执行，这个在hue的运行日志中也可以看到。但是有时候这些任务之间并不是是相互依赖的，如果集群资源允许的话，可以让多个并不相互依赖stage并发执行，这样就节约了时间，提高了执行速度，但是如果集群资源匮乏时，启用并行化反倒是会导致各个job相互抢占资源而导致整体执行性能的下降。 
启用并行化：
set hive.exec.parallel=true
```

3. hiveQL层优化
3.1 利用分区表优化
```
分区表是在某一个或者某几个维度上对数据进行分类存储，一个分区对应于一个目录。在这中的存储方式，当查询时，如果筛选条件里有分区字段，那么hive只需要遍历对应分区目录下的文件即可，不用全局遍历数据，使得处理的数据量大大减少，提高查询效率。 
当一个hive表的查询大多数情况下，会根据某一个字段进行筛选时，那么非常适合创建为分区表。

```
3.2 利用桶表优化
```
桶表的概念在本教程第一步有详细介绍，就是指定桶的个数后，存储数据时，根据某一个字段进行哈希后，确定存储再哪个桶里，这样做的目的和分区表类似，也是使得筛选时不用全局遍历所有的数据，只需要遍历所在桶就可以了。
hive.optimize.bucketmapJOIN=true;
hive.input.format=org.apache.hadoop.hive.ql.io.bucketizedhiveInputFormat; 
hive.optimize.bucketmapjoin=true; 
hive.optimize.bucketmapjoin.sortedmerge=true; 
```
3.3 join优化
```
▪ 优先过滤后再join，最大限度地减少参与join的数据量。
▪ 小表join大表原则 
应该遵守小表join大表原则，原因是join操作的reduce阶段，位于join左边的表内容会被加载进内存，将条目少的表放在左边，可以有效减少发生内存溢出的几率。join中执行顺序是从左到右生成job，应该保证连续查询中的表的大小从左到右是依次增加的。
▪ join on条件相同的放入一个job 
hive中，当多个表进行join时，如果join on的条件相同，那么他们会合并为一个
mapreduce job，所以利用这个特性，可以将相同的join on的放入一个job来节省执行时间。

select pt.page_id,count(t.url) PV 
from rpt_page_type pt join (  select url_page_id,url from trackinfo where ds='2016-10-11' ) t on pt.page_id=t.url_page_id join (  select page_id from rpt_page_kpi_new where ds='2016-10-11' ) r on t.url_page_id=r.page_id group by pt.page_id;
```
3.4 其他优化
```
▪ 启用mapjoin 
mapjoin是将join双方比较小的表直接分发到各个map进程的内存中，在map进程中进行join操作，这样就省掉了reduce步骤，提高了速度。 
▪ 桶表mapjoin 
当两个分桶表join时，如果join on的是分桶字段，小表的分桶数时大表的倍数时，可以启用map join来提高效率。启用桶表mapjoin要启用hive.optimize.bucketmapjoin参数。
▪ Group By数据倾斜优化 
Group By很容易导致数据倾斜问题，因为实际业务中，通常是数据集中在某些点上，这也符合常见的2/8原则，这样会造成对数据分组后，某一些分组上数据量非常大，而其他的分组上数据量很小，而在mapreduce程序中，同一个分组的数据会分配到同一个reduce操作上去，导致某一些reduce压力很大，其他的reduce压力很小，这就是数据倾斜，整个job 执行时间取决于那个执行最慢的那个reduce。
解决这个问题的方法是配置一个参数：set hive.groupby.skewindata=true。 
当选项设定为true，生成的查询计划会有两个MR job。第一个MR job 中，map的输出结果会随机分布到Reduce中，每个Reduce做部分聚合操作，并输出结果，这样处理的结果是相同的Group By Key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；第二个MR job再根据预处理的数据结果按照Group By Key分布到Reduce中（这个过程可以保证相同的GroupBy Key被分布到同一个Reduce中），最后完成最终的聚合操作。
▪ Order By 优化 
因为order by只能是在一个reduce进程中进行的，所以如果对一个大数据集进行order by，会导致一个reduce进程中处理的数据相当大，造成查询执行超级缓慢。
▪ 一次读取多次插入 
▪ Join字段显示类型转换 
▪ 使用orc、parquet等列式存储格式 ，针对数据仓库不同层级使用不同的储存格式。
```
Map Reduce数量相关
```
1. 数据分片大小 (分片的数量决定map的数量) 
计算公式: splitSize = Math.max(minSize, Math.min(maxSize, blockSize))

    set mapreduce.input.fileinputformat.split.maxsize=750000000;

1. 单个reduce处理的数据量 (影响reduce的数量) 
计算公式: Max(1, Min(hive.exec.reducers.max [1099], ReducerStage estimate/hive.exec.reducers.bytes.per.reducer)) x hive.tez.max.partition.factor

    set hive.exec.reducers.bytes.per.reducer=629145600;

1. tez将会根据vertice的输出大小动态预估调整reduce的个数

    set hive.tez.auto.reducer.parallelism = true; 

- 执行计划相关

1. hive执行引擎 mr/tez/spark

    set hive.execution.engine=mr;

1. 调整Join顺序，让多次Join产生的中间数据尽可能小，选择不同的Join策略

    set hive.cbo.enable=true;

1. 如果数据已经根据相同的key做好聚合，那么去除掉多余的map/reduce作业

    set hive.optimize.reducededuplication=true;

1. 如果一个简单查询只包括一个group by和order by，此处可以设置为1或2

    set hive.optimize.reducededuplication.min.reducer=4;

1. Map Join优化, 不太大的表直接通过map过程做join

    set hive.auto.convert.join=true; 
    set hive.auto.convert.join.noconditionaltask=true;

1. Map Join任务HashMap中key对应value数量

    set hive.smbjoin.cache.rows=10000;

1. 可以被转化为HashMap放入内存的表的大小(官方推荐853M)

    set hive.auto.convert.join.noconditionaltask.size=894435328;

1. map端聚合(跟group by有关), 如果开启, Hive将会在map端做第一级的聚合, 会用更多的内存 ,开启这个参数 sum(1)会有类型转换问题

    set hive.map.aggr=false;

1. 所有map任务可以用作Hashtable的内存百分比, 如果OOM, 调小这个参数(官方默认0.5)

    set hive.map.aggr.hash.percentmemory=0.5;

1. 将只有SELECT, FILTER, LIMIT转化为FETCH, 减少等待时间

    set hive.fetch.task.conversion=more; 
    set hive.fetch.task.conversion.threshold=1073741824;

1. 聚合查询是否转化为FETCH

    set hive.fetch.task.aggr=false;

1. 如果数据按照join的key分桶，hive将简单优化inner join(官方推荐关闭)

    set hive.optimize.bucketmapjoin= false; 
    set hive.optimize.bucketmapjoin.sortedmerge=false;

1. 向量化计算,如果开启, sum(if(a=1,1,0))这样的语句跑不过

    set hive.vectorized.execution.enabled=false; 
    set hive.vectorized.execution.reduce.enabled=false; 
    set hive.vectorized.groupby.checkinterval=4096; 
    set hive.vectorized.groupby.flush.percent=0.1; 

- 动态分区相关

1. hive0.13有个bug, 开启这个配置会对所有字段排序

    set hive.optimize.sort.dynamic.partition=false;

1. 以下两个参数用于开启动态分区

    set hive.exec.dynamic.partition=true; 
    set hive.exec.dynamic.partition.mode=nonstrict;

- 小文件相关

1. 合并小文件

    set hive.merge.mapfiles=true; ##在 map only 的任务结束时合并小文件
    set hive.merge.mapredfiles=true;  ## true 时在 MapReduce 的任务结束时合并小文件
    set hive.merge.tezfiles=true; 
    set hive.merge.sparkfiles=true; ## true 时在 Spark 的任务结束时合并小文件
    set hive.merge.size.per.task=536870912; ##合并文件的大小
    set hive.merge.smallfiles.avgsize=536870912; 
    set hive.merge.orcfile.stripe.level=true; 

- ORC相关

1. 如果开启将会在ORC文件中记录metadata

    set hive.orc.splits.include.file.footer=false;

1. ORC写缓冲大小

    set hive.exec.orc.default.stripe.size=67108864; 

- 统计相关

1. 新创建的表/分区是否自动计算统计数据

    set hive.stats.autogather=true; 
    set hive.compute.query.using.stats=true; 
    set hive.stats.fetch.column.stats=true; 
    set hive.stats.fetch.partition.stats=true;

1. 手动统计已经存在的表

    ANALYZE TABLE COMPUTE STATISTICS; 
    ANALYZE TABLE COMPUTE STATISTICS for COLUMNS; 
    ANALYZE TABLE partition (coll=”x”) COMPUTE STATISTICS for COLUMNS; 

- 其他

1. 在order by limit查询中分配给存储Top K的内存为10%

    set hive.limit.pushdown.memory.usage=0.1;

1. 是否开启自动使用索引

    set hive.optimize.index.filter=true;

1. 获取文件块路径的工作线程数

    set mapreduce.input.fileinputformat.list-status.num-threads=5;
```

