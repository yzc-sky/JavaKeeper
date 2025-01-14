# 一、HBase 简介

### 1.1 HBase 定义

HBase 是一种分布式、可扩展、支持海量数据存储的 NoSQL 数据库。

 

### 1.2 HBase 的起源

HBase 是一个基于 HDFS 的分布式、面向列的开源数据库，是一个结构化数据的分布式存储系统，利用 HBase 技术可在廉价 PC Server上搭建起大规模结构化存储集群。

HBase 的原型是 Google 的 BigTable 论文，受到了该论文思想的启发，目前作为 Hadoop 的子项目来开发维护，用于支持结构化的数据存储。

[Apache](http://www.apache.org/) HBase™是 [Hadoop](http://hadoop.apache.org/) 数据库，这是一个分布式，可扩展的大数据存储。

当您需要随机，实时读取/写入您的大数据时使用Apache HBase™。该项目的目标是托管非常大的表 - 数十亿行×数百万列 - 在商品硬件集群上。Apache HBase是一个开源的，分布式的，版本化的非关系数据库，其模型是由 Chang 等人在 Google 的 [Bigtable：一种用于结构化数据](http://research.google.com/archive/bigtable.html)的[分布式存储系统](http://research.google.com/archive/bigtable.html)之后建模的。就像Bigtable利用Google文件系统提供的分布式数据存储一样，Apache HBase 在 Hadoop 和HDFS 之上提供了类似 Bigtable 的功能。



### 1.3 **HBase**在商业项目中的能力

每天：

1. 消息量：发送和接收的消息数超过60亿
2. 将近1000亿条数据的读写
3. 高峰期每秒150万左右操作
4. 整体读取数据占有约55%，写入占有45%
5. 超过2PB的数据，涉及冗余共6PB数据
6. 数据每月大概增长300千兆字节。



### 1.4 特性

Hbase是一种NoSQL数据库，这意味着它不像传统的RDBMS数据库那样支持SQL作为查询语言。Hbase是一种分布式存储的数据库，技术上来讲，它更像是分布式存储而不是分布式数据库，它缺少很多RDBMS系统的特性，比如列类型，辅助索引，触发器，和高级查询语言等待。那Hbase有什么特性呢？如下：

- 强读写一致，但是不是“最终一致性”的数据存储，这使得它非常适合高速的计算聚合
- 自动分片，通过Region分散在集群中，当行数增长的时候，Region也会自动的切分和再分配
- 自动的故障转移
- Hadoop/HDFS集成，和HDFS开箱即用，不用太麻烦的衔接
- 丰富的“简洁，高效”API，Thrift/REST API，Java API
- 块缓存，布隆过滤器，可以高效的列查询优化
- 操作管理，Hbase提供了内置的web界面来操作，还可以监控JMX指标



### 1.5 什么时候用Hbase？

Hbase不适合解决所有的问题：

- 首先数据库量要足够多，如果有十亿及百亿行数据，那么Hbase是一个很好的选项，如果只有几百万行甚至不到的数据量，RDBMS是一个很好的选择。因为数据量小的话，真正能工作的机器量少，剩余的机器都处于空闲的状态
- 其次，如果你不需要辅助索引，静态类型的列，事务等特性，一个已经用RDBMS的系统想要切换到Hbase，则需要重新设计系统。
- 最后，保证硬件资源足够，每个HDFS集群在少于5个节点的时候，都不能表现的很好。因为HDFS默认的复制数量是3，再加上一个NameNode。

Hbase在单机环境也能运行，但是请在开发环境的时候使用





### 1.6 HBase 数据模型

逻辑上，HBase 的数据模型同关系型数据库很类似，数据存储在一张表中，有行有列。但从 HBase 的底层物理存储结构（K-V）来看，HBase 更像是一个 **multi-dimensional map**（多维度map）。

#### 1.6.1 HBase逻辑结构

![](https://img-blog.csdnimg.cn/20210118182455612.png)

> 通过横向切分 Region 和纵向切分 列族 来存储大数据

#### 1.6.2 HBase 物理存储结构

![](https://img-blog.csdnimg.cn/20210118182525234.png)



### 1.7 数据模型(相关术语)

#### Name Space

命名空间，类似于关系型数据库的 database 概念，每个命名空间下有多个表。HBase 两个自带的命名空间，分别是 hbase 和 default，hbase 中存放的是 HBase 内置的表，default 表是用户默认使用的命名空间。
一个表可以自由选择是否有命名空间，如果创建表的时候加上了命名空间后，这个表名字以 `<Namespace>:<Table>`作为区分！

#### Table

类似于关系型数据库的表概念。不同的是，HBase 定义表时只需要声明列族即可，数据属性，比如超时时间（TTL），压缩算法（COMPRESSION）等，都在列族的定义中定义，不需要声明具体的列。
这意味着，往 HBase 写入数据时，字段可以动态、按需指定。因此，和关系型数据库相比，HBase 能够轻松应对字段变更的场景。

#### Row

HBase 表中的每行数据都由一个 RowKey 和多个 Column（列）组成。一个行包含了多个列，这些列通过列族来分类，行中的数据所属列族只能从该表所定义的列族中选取，不能定义这个表中不存在的列族，否则报错 `NoSuchColumnFamilyException`。

#### RowKey

Rowkey 由用户指定的一串不重复的字符串定义，是一行的唯一标识！数据是按照 RowKey 的字典顺序存储的，并且查询数据时只能根据 RowKey 进行检索，所以 RowKey 的设计十分重要。
如果使用了之前已经定义的 RowKey，那么会将之前的数据更新掉！

#### Column Family

列族是多个列的集合。一个列族可以动态地灵活定义多个列。表的相关属性大部分都定义在列族上，同一个表里的不同列族可以有完全不同的属性配置，但是同一个列族内的所有列都会有相同的属性。
列族存在的意义是 HBase 会把相同列族的列尽量放在同一台机器上，所以说，如果想让某几个列被放到一起，你就给他们定义相同的列族。
官方建议一张表的列族定义的越少越好，列族太多会极大程度地降低数据库性能，且目前版本 Hbase 的架构，容易出 BUG。

#### Column Qualifier

Hbase 中的列是可以随意定义的，一个行中的列不限名字、不限数量，只限定列族。因此列必须依赖于列族存在！列的名称前必须带着其所属的列族！例如 info：name，info：age。
因为 HBase 中的列全部都是灵活的，可以随便定义的，因此创建表的时候并不需要指定列！列只有在你插入第一条数据的时候才会生成。其他行有没有当前行相同的列是不确定，只有在扫描数据的时候才能得知！

#### TimeStamp

用于标识数据的不同版本（version）。时间戳默认由系统指定，也可以由用户显式指定。
在读取单元格的数据时，版本号可以省略，如果不指定，Hbase默认会获取最后一个版本的数据返回！

#### Cell

由 `{rowkey, column Family：column Qualifier, time Stamp}` 唯一确定的单元。
Cell 中的数据是没有类型的，全部是字节码形式存储。

#### Region

Region 由一个表的若干行组成！在 Region 中行的排序按照行键（rowkey）字典排序。
Region 不能跨 RegionSever，且当数据量大的时候，HBase 会拆分 Region。
Region 由 RegionServer 进程管理。HBase 在进行负载均衡的时候，一个 Region 有可能会从当前 RegionServer移动到其他 RegionServer 上。
Region 是基于 HDFS 的，它的所有数据存取操作都是调用了 HDFS 的客户端接口来实现的。



### 1.8 HBase基本架构

![img](https:////upload-images.jianshu.io/upload_images/426671-b201f552a0cc7e1b.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1194/format/webp)

- Zookeeper，作为分布式的协调。RegionServer也会把自己的信息写到ZooKeeper中。
- HDFS是Hbase运行的底层文件系统
- RegionServer，理解为数据节点，存储数据的。
- Master RegionServer要实时的向Master报告信息。Master知道全局的RegionServer运行情况，可以控制RegionServer的故障转移和Region的切分。



详细点的：

![img](https://img-blog.csdnimg.cn/20210118183353261.png)

架构角色：

#### Region Server

RegionServer是一个服务，负责多个Region的管理。其实现类为HRegionServer，主要作用如下:

- 对于数据的操作：get, put, delete；
- 对于Region的操作：splitRegion、compactRegion。
- 客户端从ZooKeeper获取RegionServer的地址，从而调用相应的服务，获取数据。

#### Master

Master是所有Region Server的管理者，其实现类为HMaster，主要作用如下：

- 对于表的操作：create, delete, alter，这些操作可能需要跨多个ReginServer，因此需要Master来进行协调！
- 对于RegionServer的操作：分配regions到每个RegionServer，监控每个RegionServer的状态，负载均衡和故障转移。
- 即使Master进程宕机，集群依然可以执行数据的读写，只是不能进行表的创建和修改等操作！当然Master也不能宕机太久，有很多必要的操作，比如创建表、修改列族配置，以及更重要的分割和合并都需要它的操作。

#### Zookeeper

RegionServer非常依赖ZooKeeper服务，ZooKeeper管理了HBase所有RegionServer的信息，包括具体的数据段存放在哪个RegionServer上。
客户端每次与HBase连接，其实都是先与ZooKeeper通信，查询出哪个RegionServer需要连接，然后再连接RegionServer。
Zookeeper中记录了读取数据所需要的元数据表hbase:meata,因此关闭Zookeeper后，客户端是无法实现读操作的！
HBase 通过 Zookeeper 来做 Master 的高可用、RegionServer 的监控、元数据的入口以及集群配置的维护等工作

#### HDFS

HDFS为Hbase提供最终的底层数据存储服务，同时为HBase提供高可用的支持。


#### 架构细化

![img](https:////upload-images.jianshu.io/upload_images/426671-794d662b7ebba360.png?imageMogr2/auto-orient/strip|imageView2/2/w/632/format/webp)

- HMaster是Master Server的实现，负责监控集群中的RegionServer实例，同时是所有metadata改变的接口，在集群中，通常运行在NameNode上面，[这里有一篇更细的HMaster介绍](https://links.jianshu.com/go?to=http%3A%2F%2Fblog.zahoor.in%2F2012%2F08%2Fhbase-hmaster-architecture%2F)
  - HMasterInterface暴露的接口，Table(createTable, modifyTable, removeTable, enable, disable),ColumnFamily (addColumn, modifyColumn, removeColumn),Region (move, assign, unassign)
  - Master运行的后台线程：LoadBalancer线程，控制region来平衡集群的负载。CatalogJanitor线程，周期性的检查hbase:meta表。
- HRegionServer是RegionServer的实现，服务和管理Regions，集群中RegionServer运行在DataNode
  - HRegionRegionInterface暴露接口：Data (get, put, delete, next, etc.)，Region (splitRegion, compactRegion, etc.)
  - RegionServer后台线程：CompactSplitThread，MajorCompactionChecker，MemStoreFlusher，LogRoller
- Regions，代表table，Region有多个Store(列簇)，Store有一个Memstore和多个StoreFiles(HFiles)，StoreFiles的底层是Block。





## 二、Hello HBase

https://hbase.apache.org/book.html#quickstart



## 三、HBase 进阶

#### 3.1 Hbase中RegionServer架构

![](https://img-blog.csdnimg.cn/2021011920584950.png)

##### 1）StoreFile

保存实际数据的物理文件，StoreFile 以 Hfile的形式存储在HDFS上。每个Store会有一个或多个StoreFile（HFile），数据在每个StoreFile中都是有序的。

##### 2）MemStore

写缓存，由于HFile中的数据要求是有序的，所以数据是先存储在MemStore中，排好序后，等到达刷写时机才会刷写到HFile，每次刷写都会形成一个新的HFile。

##### 3）WAL

由于数据要经MemStore排序后才能刷写到HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做Write-Ahead logfile的文件中，然后再写入MemStore中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。(l类似于NameNode中fsimage和edits_log的作用)
每间隔hbase.regionserver.optionallogflushinterval(默认1s)， HBase会把操作从内存写入WAL。
一个RegionServer上的所有Region共享一个WAL实例。
WAL的检查间隔由hbase.regionserver.logroll.period定义，默认值为1小时。检查的内容是把当前WAL中的操作跟实际持久化到HDFS上的操作比较，看哪些操作已经被持久化了，被持久化的操作就会被移动到.oldlogs文件夹内（这个文件夹也是在HDFS上的）。一个WAL实例包含有多个WAL文件。WAL文件的最大数量通过hbase.regionserver.maxlogs（默认是32）参数来定义。

##### 4）BlockCache

读缓存，每次查询出的数据会缓存在BlockCache中，方便下次查询。



### 3.2 Hbase读写流程

#### 一、写数据流程

![](https://img-blog.csdnimg.cn/2021011921165373.png)

写流程：

1. Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。
2. 访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。
3. 与目标Region Server进行通讯；
4. 将数据顺序写入（追加）到WAL；
5. 将数据写入对应的MemStore，数据会在MemStore进行排序；
6. 向客户端发送ack；
7. 等达到MemStore的刷写时机后，将数据刷写到HFile。

读写都是交给regionserver负责处理！



写数据分两步：

- step1: 找所写数据region所在的regionserver
- step2： regionserver写数据



① 如何知道应该将请求发送给哪个regionserver?

> 如何知道region和regionserver的对应关系？ hbase:meta
> 如何知道 hbase:meta 所在的regionserver? 查询zk的/hbase/meta-region-server获取

当获取meta表后，如何得知当前的数据应该放入哪个region? 继而得知如何放入哪个regionserver?

> 根据当前数据插入的表获取所有的region,再根据插入数据的rowkey和region的startkey和endkey进行比较，判断当前数据应该放入哪个region!
>  再读取region所在行的info:server列，获取对应的regionserver即可！

②向regionserver发送写请求

a) 由regionserver找到对应region的WAL对象，进行预写日志记录
b) 数据根据列族，写入到对应的store对象的memstore中
c) 通知客户端写成功

③后续

memstore满的时候，或触发了其他的flush条件，memstore中的数据会被刷写到storefile中！
 书写后，过期的WAL文件会移动到oldWALS目录中。





#### 二、读取数据流程

![](https://img-blog.csdnimg.cn/2021011921180170.png)

读流程

1. Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。
2. 访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。
3. 与目标Region Server进行通讯；
4. 分别在Block Cache（读缓存），MemStore和Store File（HFile）中查询目标数据，并将查到的所有数据进行合并。此处所有数据是指同一条数据的不同版本（time stamp）或者不同的类型（Put/Delete）。
5. 将查询到的数据块（Block，HFile数据存储单元，默认大小为64KB）缓存到Block Cache。
6. 将合并后的最终结果返回给客户端。



总体分两步：

①找查询的region对应的regionserver
②regionserver处理读请求，返回数据

step1:

a) 查询zk的/hbase/meta-region-server获取 hbase:meta表的regionserver
b) 向 hbase:meta表的regionserver 发请求，下载meta表，缓存到客户端本地，方便以后使用
c) 从hbase:meta表中，根据查询的rowkey和表名，确定要查询的region，继而确定region所在的regionserver

step2:

d) 向这些region的regionserver发请求，查询指定列族的数据
e) 先查memstore，再查blockcache， 再查storefile
f) 如果扫到了storefile，那么数据所在的block(16k)会被缓存如blockcache



#### 三、存储设计

在Hbase中，表被分割成多个更小的块然后分散的存储在不同的服务器上，这些小块叫做Regions，存放Regions的地方叫做RegionServer。Master进程负责处理不同的RegionServer之间的Region的分发。在Hbase实现中HRegionServer和HRegion类代表RegionServer和Region。HRegionServer除了包含一些HRegions之外，还处理两种类型的文件用于数据存储

- HLog， 预写日志文件，也叫做WAL(write-ahead log)
- HFile 真实的数据存储文件

##### HLog

- MasterProcWAL：HMaster记录管理操作，比如解决冲突的服务器，表创建和其它DDLs等操作到它的WAL文件中，这个WALs存储在MasterProcWALs目录下，它不像RegionServer的WALs，HMaster的WAL也支持弹性操作，就是如果Master服务器挂了，其它的Master接管的时候继续操作这个文件。

- WAL记录所有的Hbase数据改变，如果一个RegionServer在MemStore进行FLush的时候挂掉了，WAL可以保证数据的改变被应用到。如果写WAL失败了，那么修改数据的完整操作就是失败的。

  - 通常情况，每个RegionServer只有一个WAL实例。在2.0之前，WAL的实现叫做HLog
  - WAL位于*/hbase/WALs/*目录下
  - MultiWAL: 如果每个RegionServer只有一个WAL，由于HDFS必须是连续的，导致必须写WAL连续的，然后出现性能问题。MultiWAL可以让RegionServer同时写多个WAL并行的，通过HDFS底层的多管道，最终提升总的吞吐量，但是不会提升单个Region的吞吐量。

- WAL的配置：

  ```jsx
  // 启用multiwal
  <property>
    <name>hbase.wal.provider</name>
    <value>multiwal</value>
  </property>
  ```

[Wiki百科关于WAL](https://links.jianshu.com/go?to=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FWrite-ahead_logging)

##### HFile

HFile是Hbase在HDFS中存储数据的格式，它包含多层的索引，这样在Hbase检索数据的时候就不用完全的加载整个文件。索引的大小(keys的大小，数据量的大小)影响block的大小，在大数据集的情况下，block的大小设置为每个RegionServer 1GB也是常见的。

> 探讨数据库的数据存储方式，其实就是探讨数据如何在磁盘上进行有效的组织。因为我们通常以如何高效读取和消费数据为目的，而不是数据存储本身。

###### Hfile生成方式

起初，HFile中并没有任何Block，数据还存在于MemStore中。

Flush发生时，创建HFile Writer，第一个空的Data Block出现，初始化后的Data Block中为Header部分预留了空间，Header部分用来存放一个Data Block的元数据信息。

而后，位于MemStore中的KeyValues被一个个append到位于内存中的第一个Data Block中：

**注**：如果配置了Data Block Encoding，则会在Append KeyValue的时候进行同步编码，编码后的数据不再是单纯的KeyValue模式。Data Block Encoding是HBase为了降低KeyValue结构性膨胀而提供的内部编码机制。

![img](https:////upload-images.jianshu.io/upload_images/426671-fc9efc43916684b1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1118/format/webp)



###### 读写简流程

![img](https:////upload-images.jianshu.io/upload_images/426671-726c0d6e0f57814a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)





## 四、HBase API

> 不过在公司使用的时候，一般不使用原生的Hbase API，使用原生的API会导致访问不可监控，影响系统稳定性，以致于版本升级的不可控。



## 五、与 Hive的集成

https://cwiki.apache.org/confluence/display/Hive/HBaseIntegration

### HBase 与 Hive 的对比

1．Hive

(1)  数据仓库

Hive 的本质其实就相当于将 HDFS 中已经存储的文件在 Mysql 中做了一个双射关系，以方便使用 HQL 去管理查询。

(2)  用于数据分析、清洗

Hive 适用于离线的数据分析和清洗，延迟较高。

(3)  基于 HDFS、MapReduce

Hive 存储的数据依旧在 DataNode 上，编写的 HQL 语句终将是转换为 MapReduce 代码执行。

2．HBase

(1)  数据库

是一种面向列族存储的非关系型数据库。

(2)  用于存储结构化和非结构化的数据

适用于单表非关系型数据的存储，不适合做关联查询，类似 JOIN 等操作。

(3)  基于 HDFS

数据持久化存储的体现形式是 HFile，存放于 DataNode 中，被 ResionServer 以 region 的形式进行管理。

(4)  延迟较低，接入在线业务使用

面对大量的企业数据，HBase 可以直线单表大量数据的存储，同时提供了高效的数据访问速度。



## 来源与感谢：

- https://hbase.apache.org/

- https://hbase.apache.org/book.html#_preface

- [《入门HBase，看这一篇就够了》](https://www.jianshu.com/p/b23800d9b227)

- [《hbase 系列文章》](https://blog.csdn.net/weixin_42796403/category_10748539.html)

