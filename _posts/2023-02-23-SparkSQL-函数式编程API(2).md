---
layout:     post
title:     SparkSQL-函数式编程API(2)
subtitle:   运行流程、与Hive的集成
date:       2023-02-23
author:     ldf
header-img: img/post-bg-spark02.png
catalog: true
tags:
    - Spark
---

# 一、SparkSQL函数定义

## 1、UDF(User Define Function)函数

> 无论Hive还是SparkSQL分析处理数据时，往往需要使用函数，SparkSQL 模块本身自带很多实现公共功能的函数，在pyspark.sql.functions中。
>
> SparkSQL与Hive一样支持自定义函数: UDF和UDAF,尤其是UDF函数在实际项目中使用最为广泛。

在Hive中，有三种类型自定义函数：

1. UDF (User-Defined-Function) 函数
   - **一对一的关系**，输入一个值经过函数以后输出一个值;
   - 在Hive中继承UDF类， 方法名称为evaluate， 返回值不能为void， 其实就是实现一个方法;
2. UDAF(User-Defined Aggregation Function)聚合函数
   - **多对一的关系**，输入多个值输出一个值，通常与groupBy联合使用;
3. UDTF (User-Defined Table- Generating Functions) 函数
   - **多对多的关系**，输入一个值输出多个值(- -行变为多行) ;
   - 用户自定义生成函数，有点像flatMap;



**在SparkSQL中，目前仅仅支持UDF函数和UDAF函数,目前Python仅支持UDF。**

UDF定义支持2种方式，

1. 使用SparkSession对象构建： 
   - 可用DSL和SQL风格

```python
udf对象 = sparksession.udf.register(参数1，参数2，参数3)
# 参数1：UDF名称，仅可用于SQL风格
# 参数2：被注册成UDF的方法名
# 参数3：声明UDF的返回值类型
# udf对象是一个UDF对象，仅可用于DSL风格
# 通过对 参数1 和 udf对象 的调用实现DSL和SQL风格综合使用
```

2. 使用functions包中提供的UDF API构建：

- 仅可用于DSL风格

```python
udf对象 = pyspark.sql.functions.udf(参数1，参数2)
# 参数1：被注册成UDF的方法名
# 参数2：声明UDF的返回值类型
# udf对象是一个UDF对象，仅可用于DSL风格
```

### 1). 代码示例1 —— Float返回

```python
# TODO方式1注册
# 注册一个UDF函数
def num_ride_10(num):
	return num * 10	# 将数字都乘以10 
# 返回值用于DSL风格 内部注册的名称用于SQL(字符串表达式)风格
udf2 = spark.udf.register("udf1", num_ride_10, IntegerType())
# udf2(UDF对象)只能DSL风格处理
df.select(udf2(df['num'])).show()
# 函数名只能用SQL风格处理
df.selectExpr("udf1(num)").show()


# TODO方式2(functions包)注册，仅能用DSL风格
# 参数1: UDF的本体方法(处理逻辑)
# 参数2: 声明返回值类型
udf3 = F.udf(num_ride_10, IntegerType())
df.select(udf3(df['num'])).show( )
# df.selectExpr("udf3(num)").show()	# 这种方式就会报错
```

### 2). 代码示例2 —— ArrayType返回

注册一个ArrayType(数字\list)类型的返回值

```python
rdd = sc. parallelize([["hadoop spark flink"], ["hadoop flink java"]])
df = rdd.toDF(["line"])
# TODO方式1注册
def split_line(line): 
	return line.split(" ")

#返回值用于DSL风格内部注册的名称用于SQL (字符串表达式)风格
udf2 = spark. udf. register("udf1", split_line, ArrayType(StringType())) 
df.select(udf2(df['line'])).show()
df.selectExpr("udf1(line)").show()

# TODO方式2注册，仅能用DSL风格
udf3 = spark.sql.functions.udf(split_line, ArrayType(StringType()))	#Array的类型需要描述清楚StringType
df.select(udf3(df['line'])).show(truncate=False)	# truncate=False 表示不要截断
```

### 3). 代码示例3 —— Dict返回

```python
rdd = sc. parallelize([[1], [2], [3]])
df = rdd.toDF(["num"])

# 注册UDF
def process(data):
	return {"num":data, "letters": string.ascii_letters[data]}	# ascii_letters方法：根据数字生成字母
	
"""
UDF的返回值是字典的话，需要用StructType来接收
"""
udf1 = spark.udf.register("udf1", process, StructType().\
             add("num",IntegerType(),nullable=True).\
             add("letters",StringType(), nullable=True))
# DSL 风格
df.select(udf1(df['num'])).show(truncate=False)
# SQL风格
df.selectExpr("udf1(num)").show(truncate=False)
```



## 2、开窗函数

> SparkSQL支持窗口函数使用, 常用SQL中的窗口函数均支持, 如`聚合窗口\排序窗口\NTILE分组窗口`等
>
> 但是注意，<font color='red'>窗口函数只用于SQL风格</font>

开窗函数的引入是为了既显示聚集前的数据，又显示聚集后的数据。即在每一行的最后一列添加聚合函数的结果（相当于加了一列，并且这一列是聚合后的结果，类似于GROUP BY的效果）。

开窗用于为行定义一个窗口(这里的窗口是指运算将要操作的行的集合)，它对一组值进行操作，**不需要**使用GROUP BY子句对数据
行分组，就能够在同一行中<font color='red'>同时</font>返回**基础行的列**和**聚合列**。

开窗函数有三类：

1. 聚合开窗函数
   - OVER(选项)：选项可以是 `PARTITION BY 子句`，但不可以是 `ORDER BY 子句`
   - 聚合类型：`SUM/MIN/MAX/AVG/COUNT`
   - 例子：`sum() OVER([PARTITION BY xx][ORDER BY xxx [DESC]])`
2. 排序开窗函数
   - OVER(选项)：现象可以是 ORDER BY 子句，也可以是 `OVER(PARTITION BY 子句, ORDER BY 子句)`，但是不可以是 `PARTITION BY 子句`
   - 排序类型：`ROW_NUMBER/RANK/DENSE_RANK`
   - 例子：`ROW_NUMBER() OVER([PARTITION BY xx][ORDER BY xxx [DESC]])`    # 先执行 over括号里面操作，再执行排序函数
3. 分区类型NTILE的窗口函数
   - 例子：`NTILE(number) OVER([PARTITION BY xx][ORDER BY xxx [DESC]])`

### 1). 代码示例：对DF数据使用聚合窗口函数、排序窗口函数

```python
from pyspark.sql import SparkSession
# 导入StructType对象
from pyspark.sql.types import ArrayType, StringType, StructType, IntegerType
import pandas as pd
from pyspark.sql import functions as F

if __name__ == '__main__':
spark = SparkSession.builder.\
appName("create df").\
master("local[*]").\
config("spark.sql.shuffle.partitions", "2").\
getOrCreate()
sc = spark.sparkContext		# 构建SparkContext对象

# 创建RDD对象
rdd = sc.parallelize([
	('张三', 'class_1', 99),
	('王五', 'class_2', 35),
	('王三', 'class_3', 57),
	('王久', 'class_4', 12),
	('王丽', 'class_5', 99),
	('王娟', 'class_1', 90),
	('王军', 'class_2', 91),
	('王俊', 'class_3', 33),
	('王君', 'class_4', 55),
	('王珺', 'class_5', 66)])

# 根据RDD构建DataFrame
schema = StructType().add("name",StringType()).\
	add("class",StringType()).\
	add("score",IntegerType()).\
df = rdd.toDF(schema)

# 创建临时表，窗口函数只能用于SQL风格
df.createTempView("stu")

# 聚合窗口函数
# 这里的 OVER(PARTITION BY class) 相当于 GROUP BY class
spark.sql("""
	SELECT *, AVG(score) OVER(PARTITION BY class) AS avg_score
	FROM stu
""").show()

# 排序窗口函数
spark.sql("""
	SELECT *,
	ROW_NUMBER() OVER(ORDER BY score DESC) AS row_number_rank,		# 按score的降序排列后，增加一个序号
	DENSE_RANK() OVER(PARTITION BY class ORDER BY score DESC) AS dense_rank,		# 按班级为单位，分数的降序排序后，增加一个序号
	RANK() OVER(ORDER BY score) AS rank		# 按分数的升序排列后，增加一个序号
	FROM stu
""").show()

# 分区窗口函数
spark.sql("""
	SELECT *, 
	NTILE(6) OVER(ORDER BY score DESC)	# 将学生的成绩降序排列后，分成六份；增加序号
	FROM stu
""").show()
```





# 二、SparkSQL运行流程

对于一个RDD而言，它的执行流程需要经历：`代码-> DAG调度器逻辑任务-> Task调度器任务分配和管理监控-> Worker干活`。

RDD的运行会完全按照开发者的代码执行，如果开发者水平有限，RDD的执行效率也会受到影响。

而SparkSQL会对写完的代码，执行“**自动优化**"，以提升代码运行效率,避免开发者水平影响到代码执行效率。

本质上，之所以SparkSQL能优化，还是得益于它统一的数据结构DataFrame，RDD由于内含的数据类型不限格式和结构，实在无从下手。

SparkSQL的自动优化，依赖于：Catalyst优化器。

## 1、Spark SQL核心 - Catalyst查询编译器

Catalyst框架由以下几个重要组成部分：

![Catalyst框架](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230224113050.png)

- **Parser**：

将SQL/Dataset/DataFrame转化成一棵未经解析（Unresolved）的树，在Spark中称为**未解析的逻辑计划**（Unresolved Logical Plan），它是用户程序的一种抽象。

- **Analyzer**：

利用目录（Catalog）中的信息，对Parser中生成的树进行解析。

Analyzer有一系列规则（Rule）组成，每个规则负责某项检查或者转换操作，如解析SQL中的表名、列名，同时判断它们是否存在。

经过Analyzer之后的数据称之为 逻辑计划(Logical Plan)，也可以理解为抽象语法树(AST)。

- **Optimizer**：

对解析完的逻辑计划进行树结构的优化，以获得更高的执行效率。

优化过程也是通过一系列的规则来完成，常用的规则如谓词下推（Predicate Pushdown）、列裁剪（Column Pruning）、连接重排序（Join Reordering）等。

此外，Spark SQL中还有一个基于成本的优化器（Cost-based Optmizer），是由DLI内部开发并贡献给开源社区的重要组件。该优化器可以基于数据分布情况，自动生成最优的计划。

- **Planner**：

将优化后的逻辑计划转化成物理执行计划（Physical Plan）。由一系列的**策略**（Strategy）组成，每个策略将某个逻辑算子转化成对应的物理执行算子，并**最终变成RDD的具体操作**。



**对于上面的这些步骤，可以通过`.explain(True)`方式来看一下执行计划（和SQL操作是一样的）：**

```python
spark.sql("""
	SELECT name, age
	FROM people
	WHERE age<19
""").explain(True)
```

输出：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230224164524.png)



**用一个SQL例子来说明AST是怎么构造的：**

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230224162003.png)

原始的AST如右图所示，catalyst优化器提供了下面两种优化方案

### 1). 断言(谓词)下推

> 本质上，可以理解为**行过滤**，提前执行where条件

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230224163655.png)

如上，谓词下推以后，会先过滤age，然后再JOIN，减少JOIN的数据量，提高性能；如果数据量大，还能减少shuffle的数据量，也是加快速度的一点



### 2). 列值裁剪

> 本质上，可以理解为**列过滤**，提前规划select的字段数量

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230224163838.png)

如上，由于people表之上的操作只用了id列，所以可以把其他列先裁剪掉，这样可以减少处理的数据量（减少被处理数据的“宽度”），优化处理速度。



> 所以，虽然JSON、CSV这类格式的数据不支持切割，但是**parquet**这个格式本身就带有schema，是非常适合和SparkSQL配合使用的存储系统



## 2、SparkSQL的执行流程

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230224112325.png)

1. 提交SparkSQL 代码
2. **catalyst优化**
  - 生成原始AST语法数
  - 标记AST元数据
  - 进行**断言(谓词)下推**和**列值裁剪**以及其它方面的优化作用在AST上
  - 将最终AST得到，生成**执行计划**
  - 将执行计划翻译为**RDD代码**

3. Driver执行环境入口构建(SparkSession)
4. DAG调度器规划逻辑任务
5. TASK调度区分配逻辑任务到具体Executor.上工作并监控管理任务
6. Worker干活





# 三、SparkSQL和Hive集成 - Spark on Hive

其实没有官方的 Spark on Hive 说法，属于大家习惯性称呼。结合网上资料，将其对应为 **SparkSQL 读写 Hive 表**特定场景。（反倒是有`Hive-on-Spark`的说法，是Hive中发布的版本）

- SparkSQL 对 Hive 为非必须依赖，SparkSQL 可以创建自己的metastore_db，但两者结合使用为目前常态
- SparkSQL 作为 Spark 生态的一员继续发展，而不再受限于 Hive，只是兼容 Hive

他们之间的关系，可以通过这张图来反映：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20230224171952.png)

可见，要实现Spark on Hive，只需要连接上Hive的MetaStore服务：

1. 保证MetaStore存在并开机
2. Spark知道MetaStore的位置(IP:端口号)

## 1、Spark on Hive配置

首先要准备几个预装的环境：

1. 准备 Hadoop 和 Hive 环境
2. 准备 Spark on Yarn 环境



配置：

1. Spark修改 hive-site.xml 配置文件：在 3 台 Spark 服务器上都操作

```
# 进入 Spark 安装目录
cd /opt/server/spark/conf

# 增加 hive-site.xml 配置文件
vim hive-site.xml
```

- hive-site.xml：

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  
    <property>
      <name>hive.metastore.warehouse.dir</name>
      <value>/user/hive/warehouse</value>
    </property>
  
    <property>
      <name>hive.metastore.local</name>
      <value>false</value>
    </property>
  
    <property>
      <name>hive.metastore.uris</name>
      <value>thrift://node1:9083</value>
    </property>
  
</configuration>
```

2. 将mysql驱动jar包放入spark的jars目录

> 因为要连接元数据，会有部分功能连接到mysql库，需要mysql驱动包

- python环境，就物理引入`mysql-connector-java-8.0.13.jar` 到`spark安装目录/jars/`

- java环境的话就直接导入即可：

```xml
<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.48</version>
        </dependency>
```

2. Hive配置MetaStore服务

- 在conf目录下的hive-site.xml中**增加配置**：

```xml
<configuration>
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://node1:9083</value>
    </property>
</configuration>
```

4. 启动 Hive 的 metastore 服务

```shell
# 进入 Hive 安装目录
cd /opt/server/hive-2.1.0
nohup bin/hive --service metastore 2>&1 >> /var/log/metastore.log & 
# nohup关闭当前session不会中断程序，可以通过kill等命令终止
# 2>&1是用来将标准错误2重定向到标准输出1中。1前面的&是为了让bash将1解释成标准输出而不是文件1。而最后一个&是为了让bash在后台执行。
```

5. 检查服务是否正常启动

```
netstat -anp|grep 9083
```

这样，我们就可以去Spark执行环境中去使用SparkSQL直接调用MetaStore访问HDFS了。

## 2、调用Spark on Hive

1. 通过shell命令行执行：`bin/pyspark`进入后，`spark.sql(create table sparkinhive(id int))`；然后进入`bin/hive`执行环境，`show tables;`
   - 也可以`bin/spark-sql`执行环境，直接编写sql语句`create table sparkinhive2(id int))`
2. 在代码中集成：

```python
spark = SparkSession.builder.\
appName("create df").\
master("local[*]"). \
config ("spark. sql. shuffle. paratitions", "4"). \
config("spark. sql.warehouse.dir", "hdfs: //node1: 8020/user/hive/warehouse"). \
config("hive. metastore.uris", "thrift://node1:9083"). \
enableHiveSupport().\ 
getOrCreate()

# 直接使用sql语句，不需要创建临时表TempView
spark.sql(""'SELECT * FROM test. studen"). show()
```

如上加入3条语句：

1. 告知Spark默认创建表存到哪里
   - `config("'spark. sql.warehouse.dir", "hdfs://node1: 8020/user/hive/warehouse")`
2. 告知Spark Hive的MetaStore在哪
   - `config ("hive. metastore.uris", "thrift://node1:9083")`
3. 告知Spark开启对Hive的支持
   - `enableHiveSupport()`



## 3、分布式SQL执行引擎 - Spark Thrift Server

**Spark Thrift Server**是Spark社区基于**HiveServer2**实现的一个Thrift服务，同样具备`和Hive Metastore进行交互，获取到hive的元数据`的能力。

- **那什么是HiveServer2？**

hiveserver2其实就是Hive启动了一个server，客户端可以直接使用JDBC协议，通过IP+ Port的方式对其进行访问，达到**并发访问**的目的。（方便开发人员直接编写SQL语句，操作Hive；而不需要了解底层的执行原理）

Spark安装包中已经提供了启动thriftServer的sh文件：`sbin/start-thriftserver.sh`

启动方式为：`sbin/start-thriftserver.sh --hiveconf hive.server2.thrift.port=10000 --hiveconf hive.server2.thrift.bind.host=node1 --master local[*] `

<font color='red'>本质上，</font>就是启动了一个ThriftServer守护进程，监听在`10000`端口上，持续对外提供服务。

### 1). 测试 - 客户端/代码

1. **客户端**的可视化JDBC连接工具：
   - **DBeaver**配置好JDBC url、主机名、用户名等
   - DataGrip
   - HeidiSQL
2. **代码**中连接ThriftServer —— 需要使用pyhive包

```shell
# 为了安装pyhive包，需要先安装一些linux软件
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel libffi-devel gcc make gcc-c++ python-devel cyrus-sasl-devel cyrus-sasl-plain cyrus-sasl-gssapi -y

# 安装pyhive包
/export/server/anaconda3/envs/pyspark/bin/python -m pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyhive pymysql sasl thrift thrift_sasl
```

代码测试：

```python
from pyhive import hive
if __name__ == '__main__' :
#获取到Hive(Spark ThriftServer) 的连接
conn = hive.Connection(host="node1", port=10000, username= 'hadoop')
#获取一个游标对象，用来执行sql
cursor = conn.cursor()

#执行sql使用executor API
cursor.execute("SELECT * FROM test")
#执行后，使用fetchall API获取全部的返回值，返回值是一个List对象
result = cursor.fetchall()
#打印输出
print(result)
```

SQL提交后，底层运行的就是Spark任务。

相当于构建了一个**以MetaStore服务为元数据，Spark为执行引擎的数据库服务**，像操作数据库那样方便的操作SparkSQL进行分布式的SQL计算。
