---
layout:     post
title:     SparkSQL-函数式编程API(1)
subtitle:   基础概念、DataFrame
date:       2023-02-20
author:     ldf
header-img: img/post-bg-spark02.png
catalog: true
tags:
    - Spark
---

# 一、SparkSQL框架模块

Spark SQL是Spark用来处理 海量 **结构化数据**的一个模块（严格来说，RDD可以处理结构化、非结构化、半结构化数据），它提供了一个编程抽象叫做DataFrame并且作为**分布式SQL查询/计算 引擎**的作用。

企业大面积使用 SparkSQL 处理业务数据，包括：

- 离线开发
- 数仓搭建
- 科学计算
- 数据分析

Spark一开始是模仿着Hive，推出了Shark框架。但是这个框架核心是Spark(RDD)，而Hive的优化项是针对MR设计的，导致这个分布式SQL引擎效果不佳。后来，开始引进SparkSQL：

1. 2014年，1.0版本Spark整合了SparkSQL发布，并且定了SchemaRDD为其基本的数据结构
2. 2015年，Spark1.3.0版本发布，提出了**DataFrame**数据结构（DataFrame这个数据结构与pandas库对应的数据结构很类似，内部都是存储的**二维表数据**）
3. 2016年1月，Spark1.6.0发布，更新了Dataset数据结构(带泛型的DataFrame)，用于支持带泛型的语言（Java/Scala）
   - 同年7月，Spark2.0发布，统一了Dataset和DataFrame，以后只有Dataset。Python用的DataFrame就是没有泛型的Dataset。（因为**Python没有泛型这个概念**）
4. 2019年，Spark3.0发布，性能提升，但是SparkSQL变化不大





## 1、SparkSQL 特点

SparkSQL 支持 SQL语言/性能强/可以针对SQL进行自动优化/API简单/兼容Hive等特点。

1. 融合性：SQL可以集成在代码中，随时用SQL处理数据

```Python
result = spark.sql(
	"SELECT * FROM people")		# SQL语句
name = result.map(lambda p:p.name)	# RDD算子
```

2. 统一的数据访问模型：有一个标准API，可读写不同数据源。JDBC、JSON、Hive、parquet文件（一种列式存储文件，是SparkSQL默认的数据源）都支持：`SparkSession.read.该数据类型的方法名(该格式数据的路径)`

3. 兼容Hive：可以使用 SparkSQL 直接计算并生成Hive数据集；Hive表可以直接交给SparkSQL处理

4. 标准化连接：支持标准化JDBC/ODBC连接，方便和各种数据库进行数据交互



## 2、SparkSQL 和 Hive 的比较

> 为什么要学习SparkSQL?
>
> SparkSQL 一开始是模仿着Hive去实现的，目的是解决MapReduce这种计算模型执行效率比较慢的问题（Hive 将Hive SQL转换成MapReduce然后提交到集群上执行）。
>
> SparkSQL 是将Spark SQL转换成RDD，然后提交到集群执行，执行效率非常快！
>
> Hive、impala、presto、Drill、SparkSQL 等都是 MPP(Massively parallel processing) 系统（一种不共享架构，每个节点运行自己的操作系统和数据库等）。
>
> - 在**数据库<font color='red'>非</font>共享集群**中，每个节点都有独立的磁盘存储系统和内存系统，业务数据根据数据库模型和应用特点划分到各个节点上，每台数据节点通过专用网络或者商业通用网络互相连接，彼此协同计算，作为整体提供数据库服务。
> - 非共享数据库集群有完全的可伸缩性、高可用、高性能、优秀的性价比、资源共享等优势。

|          | SparkSQL         | Hive          |
| -------- | ---------------- | ------------- |
| 集群化   | 支持YARN         | 支持YARN      |
| 开发语言 | SQL/代码混合执行 | 仅支持SQL开发 |
| 底层运行 | Spark RDD        | MapReduce     |
| 数据管理 | 无元数据管理     | Metasotre     |
| 计算位置 | 内存             | 磁盘          |

### 1). SparkSQL的数据抽象

| 名称                  | 数据抽象  | 描述                                                   |
| --------------------- | --------- | ------------------------------------------------------ |
| Pandas                | DataFrame | 二维表数据结构<br />单机(本地)集合                     |
| SparkCore             | RDD       | 无统一数据结构，存储各种数据结构<br />分布式集合(分区) |
| SparkSQL              | DataFrame | 二维表数据结构<br />分布式集合(分区)                   |
| SparkSQL for JVM      | Dataset   | 支持泛型<br />可用于Java、Scala语言                    |
| SparkSQL for Python/R | DataFrame | 二维表数据结构<br />分布式集合(分区)                   |

- **DataFrame和RDD之间的差别：**

|              | RDD                | DataFrame            |
| ------------ | ------------------ | -------------------- |
| 分布式       | 是                 | 是                   |
| 分区         | 是                 | 是                   |
| 弹性         | 是                 | 是                   |
| **数据结构** | 可存储任意数据结构 | 只存储二维表结构数据 |

**DataFrame和RDD谁更适合用SQL**？

- DataFrame更适合

看一个例子就知道了，对于人员信息存储，Dataframe是这样的：

| id   | name     | age  |       |
| ---- | -------- | ---- | ----- |
| 1    | zhangsan | 11   | 分区1 |
| 2    | lisi     | 11   | 分区1 |
| 3    | wangwu   | 18   | 分区2 |

RDD是这样的：

| [1,zhangsan,11]<br />[2,lisi,11] | 分区1 |
| -------------------------------- | ----- |
| [3,wangwu,18]                    | 分区2 |

很明显，DataFrame这种二维表的数据结构更适合SQL处理，因为RDD里面存一个字符串，不好做切割。

## 3、SparkSession对象

> 在RDD阶段，程序的执行入口对象是: **SparkContext**
>
> 在Spark 2.0后，推出了SparkSession对象，作为Spark编码的统一入口对象。
>
> SparkSession = RDD编程 + SparkSQL编程

SparkSession对象可以:

- 用于SparkSQL编程作为入口对象
- 用于SparkCore编程，可以通过SparkSession对象中获取到SparkContext

**构建SparkSession核心代码：**

```python
# coding:utf8
# SparkSQL 中的入口对象是SparkSession对象
from pyspark.sql import SparkSession
if __name__ == '__main__':
    # 构建SparkSession对象, 这个对象是构建器模式通过builder方法来构建
    spark = SparkSession.builder.\
    appName("local[*]").\
    config("spark.sql.shuffle.partitions", "4").\
    getOrCreate()
    # appName 设置程序名称, config设置一些常用属性
    # 最后通过getOrCreate()方法创建SparkSession对象
    
    # 通过SparkSession对象，获取 SparkContext 对象
    sc = spark.sparkContext
    
    # 读取数据文件,是一份成绩数据
    df = spark.read.csv("../data/input/stu_score.txt",seq=',',header=False)
    # 把数据处理一下，加上列名
    df2 = df.toDF("id","name","score")
    df2.printSchema()	# 打印数据模式;打印的是表的结构
    df2.show()	# 打印数据
    
    df2.createTempView("score")	#注册临时视图：创建一个score表，进行sql处理
    #处理方式一: SQL 风格
    spark.sql("""
    	SELECT * FROM score WHERE name='语文' LIMIT 5
    """).show()
    
    #处理方式二: DSL 风格(调用API)
    df2.where("name='语文'").limit(5).show()

'''
txt数据:
1,语文,99
2,语文,45
3,语文,67
4,语文,78
5,语文,0
6,语文,101
7,数学,99
...
'''
```



# 二、SparkSQL DataFrame

在Spark中，DataFrame是一种以RDD为基础的分布式数据集，类似于传统数据库的二维表格。

DataFrame可以从很多数据源构建

- 比如：已经存在的RDD、结构化文件、外部数据库、Hive表

DataFrame = RDD + schema元信息(对数据的结构描述信息)

## 1、DataFrame组成

DataFrame可以看成是一张mysql表，那么他肯定需要有这么几个属性：

- 行信息：row对象记录一行数据
- 列信息：column对象记录一列数据并包含**列的信息**（即StructField + 列数）
- 表结构描述：
  - StructType：描述整个DataFrame的表结构
  - StructField：描述一个列的信息

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230221171416.png)

### 1). DataFrame和RDD的优缺点

<font size='5'>**RDD算子：**</font>

**优点：**

1. 编译时类型安全：会进行类型检查，在编译的时候及时发现错误
2. 具有面向对象编程的风格

**缺点：**

1. 构建大量的java对象占用了大量heap堆空间，导致频繁的GC
   - 由于数据集RDD它的数据量比较大，后期都需要存储在heap堆中，这里有heap堆中的内存空间有限，出现频繁的垃圾回收（GC）。而程序在进行垃圾回收的过程中，所有的任务都是暂停，效率低。
2. 数据的序列化和反序列性能开销很大
   - 在分布式程序中，对象(对象的内容和结构)是先进行序列化，发送到其他服务器，进行大量的网络传输，然后接受到这些序列化的数据之后，再进行反序列化来恢复该对象

<font size='5'>**DataFrame：**</font>

DataFrame引入了schema元信息和off-heap(堆外)

**优点：**

1. DataFrame引入off-heap，大量的对象构建直接使用操作系统层面上的内存，不在使用heap堆中的内存，这样一来heap堆中的内存空间就比较充足，不会导致频繁GC，程序的运行效率比较高，解决`RDD频繁的GC`问题。
2. DataFrame引入了schema元信息——就是数据结构的描述信息，后期spark程序中的大量对象在进行网络传输的时候，**只需要把数据的内容本身进行序列化就可以**，数据结构信息可以省略掉。这样一来数据网络传输的数据量是有所减少，数据的序列化和反序列性能开销就不是很大了。它是解决了RDD数据的序列化和反序列性能开销很大这个缺点

**缺点：**（其实就是损失了RDD的优点）

1. 编译时类型不安全：无法在编译的时候发现错误，只有在运行的时候才会发现
2. 不再具有面向对象编程的风格



## 2、DataFrame创建

### 1). 基于RDD构建

DataFrame对象和RDD算子其实都是分布式数据集，所以只需要转换一下内部存储结构，将串行数据转换为结构化的二维表数据。

1. 通过SparkSession.createDataFrame()方法

```python
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType



if __name__ == '__main__':
	# 构建执行环境入口对象SparkSession
    spark = SparkSession.builder.\
    appName("createDF").\
    master("local[*]").\
    getOrCreate()
    
    # 构建RDD入口对象
    sc = spark.sparkContxt
    
    # # 首先构建一个RDD
    sc.textFile("../data/people.txt").map(lambda x:x.split(',')).map(lambda x:[x[0],int(x[1])])	# 做类型转换, 文件读取的时候数据全是字符串类型的
    
    # 构建DF方式1：直接用缺省的方式调用createDataFrame(粗粒度控制)
    # 参数1：被转换的RDD
    # 参数2：指定列名，通过list的形式指定
    # schema参数只传入列名称，类型从RDD中进行推断，是否允许为空默认为允许(True)
    df = spark.createDataFrame(rdd, schema = ['name', 'age'])
    
    
    # 构建DF方式2：通过StructType对象来定义DataFrame的“表结构”(细粒度控制)
    schema = StructType().\
    add("name", StringType(), nullable=True).\
    add("age", IntegerType(), nullable=False)
    df = spark.createDataFrame(rdd,schema = schema)
    
    # 构建DF方式3：使用RDD的toDF方法转换RDD
    # 方式3.1适用于类型不敏感数据: 只传列名, 类型靠推断, 是否允许为空是true
    df = rdd.toDF(['name', 'age'])
    # 方式3.2: 传入完整的Schema描述对象StructType
    df = rdd.toDF(schema)
    
    
    # 打印 DataFrame表结构
    df.printSchema()
    # 打印df数据
    # 参数1：展示数据条数，默认是20
    # 参数2：是否对数据进行截断，超过20就截断
    
    # 将DF对象注册一个临时表，供sql查询
    df.createOrReplaceTempView("people")
    spark.sql("SELECT * FROM people WHERE age < 30").show()
    
    
```

`people.txt`文件内容：

```txt
Michael,29
Andy,30
Justin,19

```



### 2). 基于Pandas的DataFrame

```python
# 构建Pandas的DF
pdf = pd.DataFrame({
"id": [1, 2, 3],
"name": ["张大仙", '王晓晓', '王大锤'],
"age": [11, 11, 11]
})
# 将Pandas的DF对象转换成Spark的DF
df = spark.createDataFrame(pdf)
```



### 3). 基于外部文件构建

```python
sparksession.read.format("text|csv|json|parquet|orc|avro|jdbc|......")
.option("K", "V") # option可选
.schema(StructType | String) # STRING的语法如.schema("name STRING", "age INT")
.load("被读取文件的路径, 支持本地文件系统和HDFS")


```

示例代码：

- **读取text数据源：**

```python
# 读取到的DataFrame只会有一个列，列名默认称之为：value; 这里我们改为"data"

schema = StructType().add("data", StringType(), nullable=True)
df = spark.read.format("text")\
.schema(schema)\
.load("../data/sql/people.txt")

# 如果需要继续处理，我们需要调用option函数来做切分; .option("sep", ";")\# 列分隔符
'''
输出：
+-----------+
|       data|
+-----------+
|Michael, 29|
|   Andy, 30|
| Justin, 19|
+-----------+

'''
```

- **读取Json数据源：**

```python
df = spark.read.format("json").\
load("../data/sql/people.json")
# JSON 类型一般不用写.schema, json自带, json带有列名和列类型(字符串和数字)
df.printSchema()
df.show()

'''
json文件格式：
{"name":"Michael"}
{"name":"Andy","age":30}
{"name":"Justin","age":19}

输出：
+----+-------+
|  age|  name|
+----+-------+
|null|Michael|
|  30|   Andy|
|  19| Justin|
+----+-------+
'''

```

- **读取csv数据源：**

```python
df = spark.read.format("csv")\
    .option("sep", ";")\# 列分隔符
    .option("header", False)\# 是否有CSV标头
    .option("encoding", "utf-8")\# 编码
    .schema("name STRING, age INT, job STRING")\# 指定列名和类型
    .load("../data/sql/people.csv")# 路径
df.printSchema()
df.show()


```

- **读取parquet数据源**

```python
# parquet 自带schema, 直接load啥也不需要了
# 想要在pycharm中直接看这个文件，需要安装插件`parquet vierwer`
df = spark.read.format("parquet").\
load("../data/sql/users.parquet")
df.printSchema()
df.show()
```

**parquet:** 是Spark中常用的一种列式存储文件格式；和Hive中的ORC差不多, 他俩都是**列存储格式**。

parquet对比普通的文本文件的区别:

- parquet 内置schema (列名\列类型\是否为空)
- 存储是以列作为存储格式
- 存储是序列化存储在文件中的(有压缩属性体积小)





## 3、操作DataFrame

DataFrame操作也称为无类型的Dataset操作。

DataFrame支持两种风格进行编程，分别是：

- SQL风格
  - 用SQL语句处理DataFrame数据
  - 比如，spark.sql("SELECT * FROM xxx")
- DSL风格
  - 领域特定语言（domain specific language）
  - 以调用API的方式来处理Data，是DataFrame特有的API
  - 比如，df.where().limit()

### 1). SQL语句编程

DataFrame的一个强大之处就是我们可以将它看作是一个关系型数据表，然后可以通过在程序中使用spark.sql() 来执行SQL语句查询，结果返回的仍然是一个DataFrame对象。

需要先将DataFrame注册成表,采用如下的方式：

```python
df.createTempView("score1")	# 注册一个临时视图(表)
df.createOrReplaceTempView("score2")	# 注册一个临时视图(表)，如果已存在就进行替换
df.createGlobalTempView("score3")	# 注册一个全局视图(表)
```

所有的表都只能在SparkSession对象的生命周期中使用：

```python
df1 = spark.sql("SELECT * FROM score1 WHERE score < 90").show()
df2 = spark.sql("SELECT subject COUNT(*) FROM score2 WHERE score < 90").show()
df3 = spark.sql("SELECT subject COUNT(*) AS cnt FROM global_temp.score3 GROUP BY subject").show()
```



### 2). DSL语句编程

- **show 方法：**

```
df.show(参数1, 参数2)
-参数1: 默认是20, 控制展示多少条
-参数2: 是否阶段列, 默认只输出20个字符的长度, 过长不显示, 要显示的话请填入truncate = True
```

- **printSchema方法：**

打印输出df的schema信息

```python
df.printSchema()
'''
root
  |-- name: string (nullable = true)
  |-- age: integer(nullable = true)
  |-- job: string(nullable = true)
'''
```

- **select方法：**

选择DataFrame中的指定列（通过传入参数进行指定）

它的传参有两种方式：

1. **可变**参数的cols对象，cols对象可以是Column对象来指定列或者字符串列名来指定列：`df.select("id","age").show();df.select(["id","age"]).show()`
2. List[Column]对象或者List[str]对象，用来选择多个列：`df.select(df['id'],df['age']).show()`

- **filter()和where()函数过滤数据：**

它的传参有两种形式：

1. string字符串类型：`df.filter("score<90").show();df.where("score<90").show()`
2. 传column形式：`df.filter(df['score']<90).show();df.where(df['score']<90).show()`

- **groupBy 分组：**

它的作用是，按照指定的列进行数据的分组，**返回值是GroupedData对象**（这个对象还有很多扩展函数）

它也有两种传参形式：

1. string字符串类型：`df.groupBy("subject").show()`
2. 传column形式：`df.groupBy(df['subject']).show()`

我们可以进一步处理GroupedData对象：

比如使用

- `.count()`统计数目，
- `.min()`求最小值，
- `.max()`求最大值，
- `.avg()`求平均值，
- `.sum()`求和值，等

这个API和SQL里面的分组操作基本一致

### 3). WordCount案例分析

1. 先把RDD数据转换成DataFrame数据
2. 再通过 SQL/DSL 语句两种方案进行读取

```python
# coding:utf8
# 演示sparksql wordcount
from pyspark.sql import SparkSession
# 导入StructType对象
from pyspark.sql.types import StructType, StringType, IntegerType
import pandas as pd
from pyspark.sql import functions as F
if __name__ == '__main__':
    spark = SparkSession.builder.\
    appName("create df").\
    master("local[*]").\
    getOrCreate()
    sc = spark.sparkContext
    # TODO 1: SQL风格处理, 以RDD为基础做数据加载
    rdd = sc.textFile("../input/words.txt").\
    flatMap(lambda x: x.split(" ")).\		# flatMap处理后是一个一维数据
    map(lambda x: [x])		# 因为DataFrame格式只能处理带嵌套的数组
    # 转换RDD到df，列名是word
    df = rdd.toDF(["word"])
    # 注册df为表
    df.createTempView("words")
    # 使用sql语句处理df注册的表
    spark.sql("""SELECT word, COUNT(*) AS cnt FROM words GROUP BY word ORDER BY cnt DESC""").show()
    
    
    # TODO 2: DSL风格处理, 纯sparksql api做数据加载
    # df当前只有一个列，默认叫做value
    df = spark.read.format("text").load("../input/words.txt")

    # 通过withColumn方法对一个列进行操作
    # 方法功能: 对老列执行操作, 返回一个全新列, 如果列名一样就替换, 不一样就拓展一个列
    df2 = df.withColumn("value", F.explode(F.split(df['value'], " ")))
    df2.groupBy("value").\
    count().\
    withColumnRenamed("count", "cnt").\
    orderBy('cnt', ascending=False).\
    show()
```



### 4). 电影评分案例分析

#### 数据来源与格式：

来源：*GroupLens*研究所的开源数据：https://files.grouplens.org/datasets/movielens/ml-100k/ 中的 `u.data`数据，它由四列组成：

| 用户ID | 电影ID | 评分 | 时间      |
| ------ | ------ | ---- | --------- |
| 196    | 242    | 3    | 881250949 |
| 286    | 1014   | 5    | 879781125 |

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230222205220.png)





#### 需求：

1. 查询用户平均分
2. 查询电影平均分
3. 查询大于平均分的电影的数量
4. 查询高分电影中(>3)打分次数最多的用户, 并求出此人打的平均分
5. 查询每个用户的平均打分, 最低打分, 最高打分
6. 查询 被评分超过100次的电影的平均分,以及他们排名的TOP10



#### 代码实现：

1. 导入库：

```python
# coding:utf8
# 演示sparksql 电影评分数据
import time
from pyspark.sql import SparkSession
# 导入StructType对象
from pyspark.sql.types import StructType, StringType, IntegerType
import pandas as pd
from pyspark.sql import functions as F
```

2. 创建SparkSession和SparkContext：

```
spark = SparkSession.builder.\
	appName("create df").\
	master("local[*]"). \
	config("spark.sql.shuffle.partitions", "2").\
	getOrCreate()
sc = spark.sparkContext
```

3. 读取数据：

```python
schema = StructType().add("user_id", StringType(), nullable=True).\
    add("movie_id", IntegerType(), nullable=True).\
    add("rank", IntegerType(), nullable=True).\
    add("ts", StringType(), nullable=True)
df = spark.read.format("csv").\
    option("sep", "\t").\
    option("header", False).\
    option("encoding", "utf-8").\
    schema(schema).\
    load("../data/sql/u.data")

# 2. 注册成一个临时表, 方便sql处理
df.createTempView("movie")
```

4. **需求1：用户平均分**

```python
df.groupBy("user_id").\
	avg("rank").\		# 求完平均数后，这一列名变为"avg(rank)"
	withColumnRenamed("avg(rank)", "avg_rank").\		# 改名
	withColumn("avg_rank", F.round("avg_rank", 2)).\	# 确定精度为小数点后2位
	orderBy("avg_rank", ascending=False).\	# 排序
	show()
```

5. **需求2：查询电影平均分**

```python
# 要么和需求1用户平均分一样的DSL操作，需要把groupBy("user_id")替换成groupBy("movie_id")
# 也可以用SQL风格处理
spark.sql("""
	SELECT movie_id, ROUND(AVG(rank), 2) AS avg_rank 	FROM movie GROUP BY movie_id ORDER BY avg_rank DESC
	""").show()
```

6. **需求3：查询大于平均分的电影的数量**

```python
# DSL风格
avg_rank = df.select(F.avg(df['rank'])).first()['avg(rank)']	# 通过 F.avg求出平均分，并通过select语句取出这一列所有数据，.first()操作取出第一行，然后第一行的'avg(rank)'列就肯定是平均数(因为它也只有这一个列)
print("大于平均分的电影数量: ", df.where(df['rank'] > avg_rank).count())	# count计数

# SQL风格
spark.sql("""
	SELECT COUNT(distinct m1.movie_id) FROM movie m1,
		(
		SELECT movie_id, ROUND(AVG(rank), 2) AS avg_rank 	FROM movie GROUP BY movie_id ORDER BY avg_rank DESC
		) AS m2
		where m1.movie_id = m2.movie_id and m1.rank > avg_rank
""")
```

7. **需求4：查询高分电影中(>3)打分次数最多的用户, 并求出此人打的平均分**

```python
# 先找到这个人的id
userId = df.where("rank>3").\	# 打高分的行
	groupBy("user_id").\
	count().\		# 按用户分组后，求一下这些用户的数量
	withColumnRenamed("count","cnt").\
	orderBy("cnt",ascending=False).\
	limit(1).\		# 找到这个cnt次数最多的列，这时候还是DF数据，没办法.show()
	first()['user_id']	# 和上面一样，找到他的第一行的'user_id'列
# 然后再根据id找到他对应的打分平均分
df.filter(df['user_id']==userId).\
	select(F.round(F.avg("rank"),2))
```

8. **需求5：查询每个用户的平均打分, 最低打分, 最高打分**

```python
# DSL风格
df.groupBy("user_id").\
	agg(
		F.round(F.avg('rank'), 2).alias("avg_rank"),
		F.min('rank').alias("min_rank"),
		F.max('rank').alias("max_rank")
).show()

# SQL风格
spark.sql("""
	SELECT user_id, avg(rank) as avg_rank, max(rank) as max_rank, min(rank) as min_rank
	FROM movie
	GROUP BY user_id
""")
```

9. **需求6：查询被评分超过100次的电影, 的平均分排名TOP10**

```python
# DSL 风格
df.groupBy("movie_id").\
	agg(
		F.count("movie_id").alias("cnt"),
		F.round(F.avg("rank"), 2).alias("avg_rank")
	).where("cnt > 100").\
	orderBy("avg_rank", ascending=False).\
	limit(10).show()

# SQL 风格
spark.sql("""
	SELECT movie_id, count(movie_id) as cnt, avg(rank) as avg_rank
	FROM movie
    GROUP BY movie_id
    HAVING cnt > 100
    ORDER BY avg_rank desc
    limit 10
""")
```

## 4、DataFrame数据写出

SparkSQL提供了统一的API来写出DataFrame：`df.write.mode().format().option(K, V).save(PATH)`

- mode，传入模式字符串可选: append 追加，overwrite 覆盖，ignore 忽略，error 重复就报异常(默认的)
- format, 传入格式字符串，可选: text， CsV, json, parquet, orc, avro, jdbc
  - <font color='red'>注意：text文件只支持单列df写出</font>
- option 设置属性，如: . option("sep", ",")
- save写出的路径，支持本地文件和HDFS

**代码示例：**

```python
# text 写出，只能写出一个单列数据
df.select(F.concat_ws("---"，, "user_ id", "movie_ id", "rank", "ts")).\	# text只支持单列数据，所以需要把列先拼接起来
	write.\
	mode("overwrite").\
	format("text").\
	save(". ./data/output/sql/text")

# CSV写出
df.write.mode("overwrite").\
	format("csv").\
	option("sep", ",").\
	option("header", True).\
	save("../data/output/sql/csv")

# Json写出
df.write.mode("overwrite").\
	format("json").\
	save("../data/output/sql/json")

# Parquet 写出
df.write.mode("overwrite").\
	format("parquet").\
	save("../data/output/sql/ parquet'")

#不给format, 默认以parquet写出
df.write.mode("overwrite").save("../data/ output/sql/default")

```



## 5、DataFrame 通过JDBC读写数据库（MySQL示例）

读取JDBC是需要有驱动的,我们读取的是`jdbc:mysql://`这个协议,也就是读取的是mysql的数据；需要有mysql的驱动jar包给spark程序用。如果不给驱动jar包，会提示: `No suitable Driver`

<font color='red'>注意：针对不同版本mysql，有不同的驱动包：</font>

- mysql5版本用：`mysql-connector-java-5.x.xx-bin.jar`
- mysql8版本用：`mysql-connector-java-8.x.xx-bin.jar`

<font color='red'>注意：针对不同版本操作系统，有不同的导入方式：</font>

- windows系统：将jar包放在`Anaconda3安全路径下\envs\虚拟环境名称\lib\site-packages\pyspark\jars`
- Linux系统：将jar包放在`Anaconda3安全路径下/envs/虚拟环境名称/lib/python3.x/site-packages/pyspark/jars`

利用上面的DataFrame数据统一读写API：

```python
# 通过JDBC读取数据
# 读出来是自带schema，不需要额外设置
df.read.format("jdbc").\
	option("url", "jdbc:mysql://node1:3306/test?useSSL=false&useUnicode=true").
	option("dbtable", "u_data").\	# dbtable属性：指定读取的表名
	option("user", "root").\
	option("password", "123456").\
	load()	# 无需参数，没有路径，直接读取数据库


# 通过JDBC写入数据
df.write.mode("overwrite").\
	format("jdbc").\
	option("url", "jdbc:mysql://node1:3306/test?useSSL=false&useUnicode=true").
	option("dbtable", "u_data").\	# dbtable属性：指定写出的表名
	option("user", "root").\
	option("password", "123456").\
	save()	# 无需参数，没有路径，直接写入数据库
```



## 6、SparkSQL数据清洗API

1. **dropDuplicates 去重**

```python
# 功能：对DF数据进行去重
# 无参数是对数据进行整体去重
df.dropDuplicates().show()
# API同样可以针对字段进行去重，如下传入age字段，表示只要年龄一样就认为你是重复数据
df.dropDuplicates(['age','job']).show()
```

2. **dropna 删除有缺失值(空值null)的行**

他有三个参数：`def dropna(self, how='any', thresh=None, subset=None)`

```python
#无参数为how=any执行，只要有一个列是null数据整行删除，如果填入how='all' 表示全部列为空才会删除，how参数默认是any
df.dropna().show( )
#指定阀值进行删除, thresh=3表示, 有效的列最少有3个, 这行数据才保留, 否则就删除
#设定thresh后, how参数无效了
df.dropna(thresh=3).show()
#可以指定阀值以及配合指定列进行工作
# thresh=2,subset=['name', 'age'] 表示针对这2个列，有效列最少为2个才保留数据
df.dropna(thresh=2，subset=['name','age']).show()
```

3. **fillna 填充/替换 缺失值数据**

```python
# 功能：根据参数的骨子额，来进行 null 的替换
#将所有的空， 按照你指定的值进行填充，不理会列的任何空都被填充
df.fillna("loss"). show()
#指定列进行填充 
df.fillna("loss", subset=['job']).show()
#给定字典设定各个列的填充规则
df.fillna({"name": "未知姓名"，"age": 1，"job": "worker"}).show()
```



# 三、总结

1. DataFrame 在结构层面上由StructField组成列描述，由StructType构造表描述。在数据层面上，Column对象记录列数据，Row对象记录行数据

2. DataFrame可以从 RDD转换、Pandas DF转换、读取文件、读取JDBC等方法构建

3. spark.read.format()和df.write.format() 是DataFrame读取和写出的统一化标准API，支持各种数据类型的读写，包括 `text/csv/json/parquet`，也支持连接外部数据库，如`JDBC`。

4. SparkSQL默认在Shuffle阶段200个分区，可以修改参数获得最好性能
5. **dropDuplicates**可以去重、**dropna**可以删除缺失值、**fillna**可以填充缺失值







