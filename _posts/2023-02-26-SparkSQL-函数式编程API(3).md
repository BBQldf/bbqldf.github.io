---
layout:     post
title:     SparkSQL-函数式编程API(3)
subtitle:   案例分析、Spark新特性、总结
date:       2023-02-26
author:     ldf
header-img: img/post-bg-spark03.png
catalog: true
tags:
    - Spark
---

# 一、案例分析 - 销售统计分析

某公司在全国各省都有分店铺，店铺的销售数据都联网上传到后台，

**4个需求需开发**：

1. 各省销售指标每个省份的销售额统计
2. TOP3销售省份中,有多少家店铺日均销售额1000+

3. TOP3省份中各个省份的平均单单价

4. TOP3省份中,各个省份的支付类型比例

**2个操作需处理**：

1. 将需求结果写出到mysql
2. 将数据写入到Spark On Hive中

**数据来源：**

数据非常复杂，但是格式很统一：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230226161741.png)

看一下原始的数据格式：

```json
{"discountRate": 1, "dayOrderSeq": 8, "storeDistrict": "雨花区", "isSigned": 0, "storeProvince": "湖南省", "origin": 0, "storeGPSLongitude": "113.01567856440359", "discount": 0, "storeID": 4064, "productCount": 4, "operatorName": "OperatorName", "operator": "NameStr", "storeStatus": "open", "storeOwnUserTel": 12345678910, "corporator": "hnzy", "serverSaved": true, "payType": "alipay", "discountType": 2, "storeName": "杨光峰南食店", "storeOwnUserName": "OwnUserNameStr", "dateTS": 1563758583000, "smallChange": 0, "storeGPSName": "", "erase": 0, "product": [{"count": 1, "name": "百事可乐可乐型汽水", "unitID": 0, "barcode": "6940159410029", "pricePer": 3, "retailPrice": 3, "tradePrice": 0, "categoryID": 1}, {"count": 1, "name": "馋大嘴盐焗鸡筋110g", "unitID": 0, "barcode": "6951027300076", "pricePer": 2.5, "retailPrice": 2.5, "tradePrice": 0, "categoryID": 1}, {"count": 2, "name": "糯米锅巴", "unitID": 0, "barcode": "6970362690000", "pricePer": 2.5, "retailPrice": 2.5, "tradePrice": 0, "categoryID": 1}, {"count": 1, "name": "南京包装", "unitID": 0, "barcode": "6901028300056", "pricePer": 12, "retailPrice": 12, "tradePrice": 0, "categoryID": 1}], "storeGPSAddress": "", "orderID": "156375858240940641230", "moneyBeforeWholeDiscount": 22.5, "storeCategory": "normal", "receivable": 22.5, "faceID": "", "storeOwnUserId": 4082, "paymentChannel": 0, "paymentScenarios": "PASV", "storeAddress": "StoreAddress", "totalNoDiscount": 22.5, "payedTotal": 22.5, "storeGPSLatitude": "28.121213726311993", "storeCreateDateTS": 1557733046000, "payStatus": -1, "storeCity": "长沙市", "memberID": "0"}
{"discountRate": 1, "storeShopNo": "277551253310005", "dayOrderSeq": 6, "storeDistrict": "岳麓区", "isSigned": 1, "storeProvince": "湖南省", "origin": 0, "storeGPSLongitude": "112.95106", "discount": 0, "storeID": 718, "productCount": 1, "operatorName": "OperatorName", "operator": "NameStr", "storeStatus": "open", "storeOwnUserTel": 12345678910, "payType": "alipay", "discountType": 2, "storeName": "芙蓉兴盛汶强食品店", "storeOwnUserName": "OwnUserNameStr", "dateTS": 1546737450000, "smallChange": 0, "storeGPSName": "None", "erase": 0, "product": [{"count": 1, "name": "白沙", "unitID": 8, "barcode": "6901028191012", "pricePer": 7, "retailPrice": 7, "tradePrice": 0, "categoryID": 21}], "storeGPSAddress": "None", "orderID": "15467374316307186813", "moneyBeforeWholeDiscount": 7, "storeCategory": "normal", "receivable": 7, "faceID": "", "storeOwnUserId": 577, "paymentChannel": 0, "paymentScenarios": "PASV", "storeAddress": "StoreAddress", "totalNoDiscount": 7, "payedTotal": 7, "storeGPSLatitude": "28.158909", "storeCreateDateTS": 1534920455000, "storeCity": "长沙市", "memberID": "0"}
{"discountRate": 1, "storeShopNo": "None", "dayOrderSeq": 26, "storeDistrict": "芙蓉区", "isSigned": 0, "storeProvince": "湖南省", "origin": 0, "storeGPSLongitude": "113.03312", "discount": 0, "storeID": 1786, "productCount": 1, "operatorName": "OperatorName", "operator": "NameStr", "storeStatus": "open", "storeOwnUserTel": 12345678910, "payType": "cash", "discountType": 2, "storeName": "裕丰超市", "storeOwnUserName": "OwnUserNameStr", "dateTS": 1546478081000, "smallChange": 0, "storeGPSName": "None", "erase": 0, "product": [{"count": 1, "name": "双喜经典包装", "unitID": 8, "barcode": "6901028000826", "pricePer": 10, "retailPrice": 10, "tradePrice": 8.58, "categoryID": 21}], "storeGPSAddress": "None", "orderID": "154647808086017867144", "moneyBeforeWholeDiscount": 10, "storeCategory": "normal", "receivable": 10, "faceID": "", "storeOwnUserId": 1714, "paymentChannel": 0, "paymentScenarios": "OTHER", "storeAddress": "StoreAddress", "totalNoDiscount": 10, "payedTotal": 10, "storeGPSLatitude": "28.198982", "storeCreateDateTS": 1540802164000, "storeCity": "长沙市", "memberID": "0"}
```

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230226161022.png)

虽然子弹很多，但是我们只需要用到上述**5个字段**：

- storeDistrict：店铺所在行政区
- **storeProvince：店铺所在省份**（需求2）
- **storeID：店铺ID**（需求3）
- storeName：店铺名称
- **dateTS：订单日期**（需求2）
- orderID：订单ID
- **receivable：收款金额**（需求1、需求2）
- **payType：付款类型**（需求4）



1. 首先，创建好SparkSession对象，连接上thriftserver：

```python
SparkSession.builder.appname("Sales demand").\
	master("local[*]").\
	config("spark.sql.shuffle.partitions","2"),\
	config("spark.sql.warehouse.dir","hdfs://node1:8020/user/hive/warehouse").\
	config("hive.metastore.uris","thrift://node3:9083").\
enableHiveSupport().\
getOrCreate()
```

2. 从JSON文件中读取数据（处理缺省值,省份/订单金额过滤/列值裁剪）

```python
df = spark.read.format("json").load("../data/input/mini.json").\
dropna(thresh=1,subset=['storeProvince'].\		# 不填的要去掉
filter("storeProvince != 'null' ").\		# “null”字符串也视为空
filter("receivable < 10000").\		# 数据集中有的订单的金额是单笔超过10000的，这些是测试数据
select("storeProvince","storeID","receivable","dateTS","payType")		# 列值裁剪
```

## 1、各省销售指标每个省份的销售额统计

```python
# 按省份排名后，把销售额做sum
# 注意：import spark.sql.functions as F
sellProvince = df.groupBy("storeProvince").sum("receivable").\
	withColumnRenamed("sum(receivale)","sum_money").\
	withColumn("sum_money",F.round("money",2)).\		# 保留两位小数
	orderBy("sum_money",ascending=False)	# 降序排列
	
	sellProvince.show(truncate=False)	# 不要缺省输出
	
    
    # 输出到mysql
	sellProvince.write.mode("overwrite").\
    	format("jdbc").\
    	option("url","jdbc:mysql://node1:3306//bigdata?useSSL=false&useUnicode=true&characterEncoding=utf8").\
    option("dbtable","province_sale").\
    option("user","root").\
    option("password","2212072ok1").\
    option("encoding","utf-8").\
    save()
    
    # 写入到Hive,可以直接使用saveAsTable方法(但是要提前配置好Hive)
    sellProvince.write.mode("overwrite").\
    saveAsTable("default.province_sale","parquet")	# 表名"province_sale"
```



## 2、TOP3销售省份中,有多少家店铺日均销售额1000+

先过滤到TOP3销售省份（在需求1的基础上，直接找到TOP3），然后把过滤销售额

```python
# 先过滤到TOP3销售省份
top3Province = sellProvince.limit(3).\
select("storeProvince").withColumnRenamed("storeProvince","top3Province")			# 列值裁剪;并且换个名字，避免和下面表连接的时候产生同名的列
# 和原始的df表进行内连接，就是全部TOP3的销售数据
selectedProvince = df.join(top3Province,on=df['storeProvince' == top3Province['top3Province']])
# 按店铺来分组(避免重名：省份 + 店铺名; 日均销售： 时间戳要在同一天内)
# 调用from_unixtime转换时间
store_count_day = selectedProvince.groupBy("storeProvince","storeID",
                         F.from_unixtime(df['dateTS'].substr(0,10),'yyyy-mm-dd')"").alias("day").\		# from_unixtime的精度是秒级，数据的精度是 毫秒级，要对数据进行精度的裁剪 ubstr; alias并改一下名(F操作的返回时column对象)
sum("receivable").withColumnRenamed("sum(receivable)","sum_money").\		# 这里是df对象
filter("money>1000").\
dropDuplicates(subset = ["storeProvince","storeID"]).\			# 把同样的店铺名过滤掉,只需要统计一次即可(同一个店铺会在很多天都达到要求)
groupBy("storeProvince").count()		# 最后才按照省份统计这个省有多少家

# 后面也一样，把store_count_day写入到mysql和Hive（一样的代码，主要改一下表名
```

由于需求2-4都是要求TOP3省份，我们可以先缓存一下(<font color='red'>最后的时候，要记得清理缓存</font>):

```python
# from pyspark.storagelevel import StorageLevel
# 这里是DF数据，但是最底层其实是RDD数据，所以缓存是有效的
selectedProvince.persist(StorageLevel.MEMORY_AND_DISK)

# 最后的时候，要记得清理缓存
selectedProvince.unpersist()
```



## 3、TOP3省份中各个省份的平均单的单价

> 这个需求很简单了，直接把每个省的交易单的平均数

```python
price_unit = selectedProvince.groupBy("storeProvince").\
	avg("receivable").\
	withColumnRenamed("avg(selectedProvince)","price unit").\
	withColumn("price unit", F.round("price unit", "2")).\
	orderBy("price unit",ascending=False)	# 降序排列

# 同样的，也要输出到mysql和Hive
```



## 4、TOP3省份中,各个省份的支付类型比例

> 先按省份进行过滤（也是top3），然后把他们所有的 **支付类型/全部的支付数据** 都输出出来；这种贴在后面的列可以尝试用开窗函数解决
>
> 求百分比：常规操作就是先求一个total数据，然后和原表联合查询（或者说子查询）

```python
# 创建临时表
selectedProvince.createTempView("province_pay")

# 这里直接使用SQL风格
pay_type = spark.sql("""
	SELECT storeProvince, payType, (count(payType)/ total) AS percent
	FROM (
		SELECT storeProvince, payType, count(1) OVER(PARTITION BY storeProvince) AS total
		FROM province_pay
		) AS sub_table
	GROUP BY storeProvince, payType, total
""")	# 使用 开窗函数; 子查询的表要单独命名

# 但是这个时候“percent”列的数据是"0.5290805676312312",尾数很多的小数，我们用一个UDF函数来进一步处理
def  udf_func(percent):
    return str(round(percent*100,2) + "%")
# 注册这个UDF
# from pyspark.sql.types import StringType
F.udf(udf_func, StringType())

pay_type_final = 	pay_type.withColumn("percent",udf_func("percent"))	# 调用这个自定义函数

#后面写入到mysql和Hive

#去除top3省份的缓存
selectedProvince.unpersist()
```

注意：要去除top3省份的缓存



# 二、Spark新特性

## 1、Spark的shuffle流程

> shuffle过程，又叫“洗牌”过程

shuffle过程中，在本质上也是一个**Map-Reduce过程**：

- 提供数据的称为 Map端(shuffle write)
- 接收数据的称之为 Reduce(shuffle read)

如果不做优化的话，它的内存分布是比较混乱的：



所以，spark提供了两类shuffle优化器：

1. 分组优化器：hashShuffleManager
2. 排序优化器：SortShuffleManager

### 1). 分组优化器：hashShuffleManager

![未经优化的hashShuffleManager](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230226220235.png)

在executor内部task之间，数据不共享，有各自的内部buffer内存，也就相应地产生了各自的文件(block file)。

![优化后的hashShuffleManager](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230226221315.png)





基本和未优化的一致,不同点在于
1. 在一个Executor内, 不同Task是**共享Buffer缓冲区**
2. 这样**减少了缓冲区乃至写入磁盘文件的数量**, 提高性能



### 2). 排序优化器：SortShuffleManager

> SortShuffleManager的运行机制主要分成两种，一种是普通运行机制，另一种是bypass运行机制。

1. **普通运行机制**

数据来了之后，先用Map/Array结构来暂缓数据；达到一定阈值之后，就会进行排序；排序后的数据写入到内存缓冲区，进而写入到磁盘文件中（**最后会合并成一个磁盘文件**）；下游的task来拉取数据的时候，需要访问索引文件：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230226222357.png)

我们可以发现，一个task对应一个磁盘文件，那么有多少并行度就会有多少个磁盘文件，这里和hashShuffle有很大区别(hashShuffle的磁盘文件很多)

<font color='red'>优势：磁盘文件的减少，也就间接地减少了网络I/O</font>

> <font color='red'>思考几个问题：</font>
>
> 1. **排序怎么排？**
> 2. **为什么排序？随机不行吗？**
> 3. **为什么写入磁盘？直接由内存生成字节码进行网络传输可以吗？**





2. **bypass运行机制**

bypass运行机制的触发条件如下：

1. shuffle map task数量小于`spark.shuffle.sort.bypassMergeThreshold=200`参数的值
2. 不是聚合类的shuffle算子(比如reduceByKey)。

同普通机制基本类同,区别在于：

1. 写入磁盘临时文件的时候不会在内存中进行排序，而是直接写,最终合并为一个task一个最终文件
2. 所以和普通模式IDE区别在于:
   1. 磁盘写机制不同;
   2. 不会进行排序。也就是说，启用该机制的<font color='red'>最大好处在于</font>，**shuffle write过程中,不需要进行数据的排序操作,也就节省掉了这部分的性能开销**

### 3). SortShuffle和hashShuffle区别

1. SortShuffle对比HashShuffle可以减少很多的磁盘文件,以节省网络IO的开销
2. SortShuffle主要是对磁盘文件进行合并来进行文件数量的减少，同时两类Shuffle都需要经过内存缓冲区溢写磁盘的场景。所以可以得知，尽管Spark是内存迭代计算框架，但是**内存迭代主要在窄依赖中**。在宽依赖(Shuffle)中磁盘交互还是一个无可避免的情况。

所以,我们要尽量减少Shuffle的出现，不要进行无意义的Shuffle计算。



## 2、Spark3.0新特性

在Spark3.0的更新中，我们有下面的统计数据：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230226223248.png)

可以看到，SparkSQL的更新占了约50%。

### 1). Adaptive Query Execution自适应查询(SparkSQL模块)

> 由于缺乏或者不准确的数据统计信息(元数据，会加到sql的AST树上)和对成本的错误估算(执行计划调度)导致生成的初始执行计划不理想。

在Spark3.x版本提供Adaptive Query Execution（AQE）自适应查询技术，通过在”运行时”对查询执行计划进行优化, 允许Planner在运行时执行可选计划,这些可选计划将会基于运行时数据统计进行动态优化, 从而提高性能。

开启AQE方式：在spark的配置文件`spark-conf`中设置`set spark.sql.adaptive.enabled=true;`

AQE主要提供了三个自适应优化：

1. 动态合并：可以动态调整shuffle分区的数量。用户可以在开始时设置相对较多的shuffle分区数，AQE会在运行时将相邻的小分区合并为较大的分区。
2. 动态调整join策略：此优化可以在一定程度上避免由于缺少统计信息或着错误估计大小（当然也可能两种情况同时存在），而导致执行计划性能不佳的情况。这种自适应优化可以在运行时`sort merge join`转换成`broadcast hash join`，从而进一步提升性能
3. 动态优化倾斜join：`skew joins`可能导致负载的极端不平衡，并严重降低性能。
   - 在AQE从shuffle文件统计信息中检测到任何倾斜后，它可以将倾斜的分区分割成更小的分区，并将它们与另一侧的相应分区连接起来。这种优化可以并行化倾斜处理，获得更好的整体性能。

### 2). Dynamic Partition Pruning动态分区裁剪(SparkSQL模块)

当优化器在编译时无法识别可跳过的分区时，可以使用"动态分区裁剪"，即基于运行时推断的信息来进一步进行分区裁剪。

这在星型模型中很常见（星型模型是由一个或多个并且引用了任意数量的维度表的事实表组成）。在这种连接操作中，我们可以通过识别维度表过滤之后的分区来裁剪从事实表中读取的分区。在一个TPC-DS基准测试中，102个查询中有60个查询获得2到18倍的速度提升。

### 3). 增强的Python API: PySpark和Koalas

很多Python开发人员在数据结构和数据分析方面使用pandas API，但仅限于单节点处理。Databricks会持续开发Koalas——基于Apache Spark的pandas API实现，让数据科学家能够在分布式环境中更高效地处理大数据。

经过一年多的开发，Koalas实现对pandas API将近80%的覆盖率。Koalas每月PyPI下载量已迅速增长到85万，并以每两周一次的发布节奏快速演进。虽然Koalas可能是从单节点pandas代码迁移的最简单方法，但很多人仍在使用PySpark API，也意味着PySpark API也越来越受欢迎。



# 三、简单回顾一下~

Spark概念中有六大分支：

1. 部署模式
2. RDD算子
3. Spark执行流程
4. DataFrame概念
5. SparkSQL执行流程
6. Spark各个概念之间的层级关系



## 1、部署模式

4大角色：

1. 资源管理类型（对集群资源进行管理）：
   1. master
      - Local: Local进程本身
      - StandAlone: Master进程
      - YARN: YARN的ResourceManager进程
   2. worker（对单台服务器进行资源管理）
      - Local: Local进程本身
      - StandAlone Worker进程
      - YARN: YARN的NodeManager进程
2. 任务运行类型：
   1. Driver（Driver本身也是属于executor，在这里标记位Driver）
      - Local: Driver运行在Local进程内
      - StandAlone:运行在Worker进程内部
      - YARN：有两种模式：
        - YARN-Client：Driver运行在client进程中
        - YARN-Cluster：Driver和运行在ApplicationMaster在同一个容器内进程中
   2. Executor
      - Local:不存在，由Local模式内的工作线程来干活
      - StandAlone:运行在Worker进程中
      - YARN:运行在YARN提供的容器内部

## 2、RDD算子

**它的定义：**

1. 弹性的（resilient）：RDD的分区数量可以动态的增减
2. 分布式（distribute）：RDD的数据是分散在各个分区之上的,各个分区被
   托管在多个Executor上
3. 数据集（dataset）：存储数据的集合

**5大特性：**

1. RDD是有分区的
2. 算子作用在每一一个分区之上
3. RDD之间是有血缘关系
4. (可选)针对KV型RDD,可以自定义分区器一默认的分区规则是Hash规则
5. (可选)如有可能, RDD加载数据将会就近取值，在数据所在的机器上启动Executor来加载数据——**移动数据不如移动计算**

**2类算子：**

1. Transformation：返回值是一个RDD，还能继续调用。
2. Action：返回值不是一个RDD，用于将Transformation生成的蓝图进行调用来开启工作
   - 多数的算子都是直接将结果想Driver汇聚，但是有两个`foreach/saveAsText`是由RDD的分区(线程)直接执行，和Driver无交互

**RDD还提供了持久化机制(persist、cached、checkpoint)**

1. persist、cached缓存：保留血缘关系，可以存储在磁盘/内存（但是不能在HDFS中，所以没有副本来容错，不太鲁棒）
2. checkpoint：不保留血缘关系，可以存储在磁盘/HDFS（但是不能在内存中，所以比较慢）

## 3、Spark执行流程

大方向上：

1. 提交代码
2. 生成Driver 一 **DAGScheduler**规划逻辑任务
3. 生成Executor(被Driver生成)
4. Driver内**TaskScheduler**去监控整个Spark程序的执行

<font color='red'>**细粒度的执行流程：**</font>

1. 客户端提交代码到YARN
2. YARN构建第一个容器：
   1. 启动ApplicationMaster
   2. AM启动Driver
   3. Driver构建DAGScheduler规划任务
   4. Driver和AM通讯，AM知道了要申请多少容器资源
   5. Driver在申请的容器内部启动Executor
   6. Driver内的taskScheduler执行 任务（task）

这其中，会涉及到内存迭代（或者说，**内存管道、宽窄依赖、并行计算**）的过程：

1. Spark的任务都会产生DAG执行图
2. DAG会基于宽窄依赖划分阶段
   1. 阶段内部都是窄依赖
   2. 内部会基于并行划分出一个个的并行执行Task，这些Task每 一个就是一个内存迭代PipLine
3. 每一个Task由一个具体的线程执行
4. 每个线程处理的流水线内都可以在这个线程内部全部计算完成
5. **所以,多个Task(PipLine)并行执行，并行的内存迭代计算**



## 4、DataFrame概念

> 他其实就是相对于单机数据结构，是分布式数据结构（**区别于RDD，DataFrame仅储存结构化数据，并且以二维表格式进行保存**），所有有这么几个特性：
>
> 1. 分布式：多分区存储
> 2. 弹性：分区可以动态增减，他也随之增减
> 3. 数据集：存储数据
>
> 他也有RDD的那些特性~

他和RDD的区别：

1. DF是只能储存二维表结构：所以**也能提前优化**
2. RDD不限制数据结构



## 5、SparkSQL执行流程

它的提交代码的过程和spark本身是一致的，但是他有一个优化器——CataLyst优化器。

它生成SQL的过程（或者说，将SQL翻译成底层的RDD过程）：

1. 生成原始AST语法树
2. 标记AST打上元数据
3. 执行AST优化
   - 谓词下推\断言下推
   - 列值裁剪
4. 生成优化后的执行计划
5. 计划翻译成RDD代码执行



## 6、Spark各个概念之间的层级关系

1. 一个Spark环境中有很多Application
2. 一个Application中有很多Jobs
3. 一个job对应一个DAG（或者说，一个job对应一个Action）——**逻辑层面**
4. 一个job中有很多stage
   1. 每一个stage都有很多task