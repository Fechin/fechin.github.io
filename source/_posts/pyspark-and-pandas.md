---
title: Pandas 和 PySpark 的 DataFrame 相互转换
date: 2018-06-26 16:00:00
categories:
    - Python
tags:
    - Pandas
    - PySpark
cover: /static/images/cover/pyspark-and-pandas.jpeg
mp3: /static/mp3/That%20Girl.mp3
---

![Pandas to PySpark](/static/images/PandasToSparkDataFrame/pyspark-pandas_cover5.jpg)

Pandas DataFrame 是一个二维数组，跟数据库中的 Table 或 Excel 很相似，底层使用 Numpy 和 array 存储，Numpy 使用 C 语言编写，运行速度很快。在 Python 语言中，它们是必不可少的机器学习库。

因为 Pandas 的高效，实际开发中，我更多的会借助它做抽样数据分析，然而生产环境的数据量巨大，需要集群部署，所以生产环境用 Spark 全量运行。Spark 和 Pandas 都可以集成 SQL 能力，但它们支持的 SQL 规范不一致，为保持操作统一，我们可能会选择其中的一种规范，也就有了 Pandas 和 PySpark 数据结构的转换需求。

#### PySpark DataFrame 转 Pandas DataFrame

获取或创建 SparkSession

```python
from pyspark.sql import SparkSession

# 在集群 UI 上展示的应用程序名称
appName = 'Spark2Pandas'
# 集群的 master URL，本地测试可以是“local”
master = 'local'
spark = SparkSession.builder.appName(appName).master(master).getOrCreate()
```


初始化 Spark 上下文

```python
from pyspark.sql import SQLContext
sc = spark.sparkContext
```

生成测试数据

```python
spf = sc.parallelize([(1, "foo"), (2, "bar")]).toDF(["id", "value"])
spf.show()

+---+-----+
| id|value|
+---+-----+
|  1|  foo|
|  2|  bar|
+---+-----+
```

PySpark 提供了 toPandas 方法，返回一个 Pandas DataFrame 对象，需要注意，不是所有的类型都可以 toPandas，目前尚不支持的数据类型有 ArrayType、 MapType，跟进问题处理进度请参考Jira [SPARK-21187](https://issues.apache.org/jira/browse/SPARK-21187)。

```python
pdf = spf.toPandas()
print pdf.head()

	id value
0   1   foo
1   2   bar
```

##### 空类型处理
在 PySpark 转 Pandas 之前，可能需要对数据做一些预处理，如修正数据，数据类型兼容等。下面这个例子是把空的 ArrayType、MapType 转换为 None。读者可举一反三，自行定制。另外，我们还可以通过Spark udf函数达到这个目的。
```python
from pyspark.sql.types import *
from pyspark.sql.functions import when, size, col, lit
for i, f in enumerate(spf.schema.fields):
    if isinstance(f.dataType, ArrayType) \
    or isinstance(f.dataType, MapType):
        spf = spf.withColumn(f.name, when(size(col(f.name)) == 0 , lit(None)).otherwise(col(f.name) ) )
```

#### Pandas DataFrame 转 PySpark DataFrame
PySpark 的 `createDataFrame(data, schema=None, samplingRatio=None)` 非常强大，它不仅支持 RDD、Python 元组和列表作为输入，还可以是 Pandas DataFrame，其内部会自动进行转换，其原理可以参考下文PySpark实现原理部分。
```
sqlContext.createDataFrame(pdf)
```

#### 性能测试
下面我们来看看 Pandas 和 PySpark 的 DataFrame 转换的性能，笔者使用的个人开发机，配置如下：
> MacBook Pro (Retina, 13-inch, Late 2013)
> 处理器：2.4 GHz Intel Core i5
> 内存：8 GB 1600 MHz DDR3

```python
n_rows = 500000
pdf = pd.DataFrame(np.random.randn(n_rows, 20))
%time spf = sqlContext.createDataFrame(pdf)

CPU times: user 1min 32s, sys: 1.08 s, total: 1min 33s
Wall time: 1min 37s
```
当 Pandas DataFrame 行数为 n_rows ，列固定 20 ，将其转换为 PySpark DataFrame 耗时如下：

![PySpark to Pandas](/static/images/PandasToSparkDataFrame/PandasToSparkDataFrame_withoutarrow_mini.jpg)
随着行数的增加，越来越耗时，100w 条数据时耗时已经超过 200 秒，笔者也测试了从 PySpark DataFrame 到 Pandas DataFrame 的转换，耗时基本一致，时间都去哪儿了？


#### PySpark 实现原理
Spark 是由 JVM 语言实现，并且程序会运行在 JVM 之上，无论是 Python、R 还是其它语言实现的 Spark，都会间接的和 JVM 进行通信，获得处理海量数据的能力。
![Pandas to PySpark](/static/images/PandasToSparkDataFrame/PySpark2_mini.jpg)

Python 借助 Py4j 实现和 Java 的交互，一个 PySpark 程序启动时，会实例化 Python 版的 SparkContext 对象，这也是一个比较耗时的过程，其内部实现：
1. 实例化 Py4j GatewayClient，连接 JVM 中的 Py4j GatewayServer
2. 通过 Py4j Gateway 在 JVM 中实例化 SparkContext 对象

以上，大概解释了为什么创建 SparkContext 慢的原因。那么，Pandas DataFrame 转 PySpark DataFrame 慢是为什么呢？原因有三：
1. PySpark 并没有直接使用 Pandas DataFrame 的数据类型，而是试图推断数据。
2. PySpark 不能使用高效的 NumPy 库，并且会迭代每一条记录，将记录中的值转换为 Python 对象。
3. 通过 Py4j 发送到 JVM 时，必须序列化成 pickle 格式，Spark 会通过单独的线程读取文件并转换成 Scala 类型。

数据量大时，这一系列步骤将是非常耗时的过程。

#### 使用 Arrow 提高转换速度
从 Spark 2.3 开始，集成了 Apache Arrow 支持，Pandas DataFrame 可以高效地转换为 Arrow 数据并直接传输到 JVM 以创建 Spark DataFrame，性能会更好，不在需要类型推断，参考 [SPARK-20791](https://issues.apache.org/jira/browse/SPARK-20791)。开启 Arrow 后的执行原理：
1. 根据默认并行数将 Pandas DataFrame 切片
2. 将每个 Pandas 切片转换为 Arrow RecordBatch
3. 将 schema 从 Arrow 转换为 Spark
4. 将 RecordBatchES 发送给 JVM，作为 JavaRDD[Array[Byte]]
5. 使用 Spark schema 包装 JavaRDD 并创建 DataFrame

代码实现：
```python
# 也可以在 spark-defaults.conf 全局配置
spark.conf.set("spark.sql.execution.arrow.enabled", "true")

spf = spark.createDataFrame(pdDF)
```
如图，使用 Arrow 之后，测试 100w 条数据仅仅用了 1.2 秒
![PySpark to Pandas](/static/images/PandasToSparkDataFrame/PandasToSparkDataFrame_witharrow_mini.jpg)

跟不使用 Arrow 相比，差得不是一点半点啊，下图中的红线是使用 Arrow 的耗时，基本与 X 轴重合了，这么高的反差，笔者感到很惊讶。
![PySpark to Pandas](/static/images/PandasToSparkDataFrame/PandasToSparkDataFrame_twoline_mini.jpg)
