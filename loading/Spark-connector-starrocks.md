# 从 Apache Spark™ 导入

StarRocks 提供 Apache Spark™ 连接器 (StarRocks Connector for Apache Spark™)，可以通过 Spark 导入数据至 StarRocks。
基本原理是对数据攒批后，通过 [Stream Load](./StreamLoad.md) 批量导入StarRocks。Connector 导入数据基于Spark DataSource V2 实现，
可以通过 Spark DataFrame 或 Spark SQL 创建 DataSource，支持 Batch 和 Structured Streaming。

## 版本要求

| Connector | Spark           | StarRocks | Java  | Scala |
|----------|-----------------|-----------|-------| ---- |
| 1.1.0    | 3.2, 3.3, 3.4   | 2.4 及以上   | 8     | 2.12 |


## 获取 Connector

您可以通过以下方式获取 connector jar 包
* 直接下载已经编译好的jar
* 通过 Maven 添加 connector 依赖
* 通过源码手动编译

connector jar包的命名格式如下

`starrocks-spark-connector-${spark_version}_${scala_version}-${connector_version}.jar`

比如，想在 Spark 3.2 和 scala 2.12 上使用 1.1.0 版本的 connector，可以选择 `starrocks-spark-connector-3.2_2.12-1.1.0.jar`。

> **_注意:_** 一般情况下最新版本的 connector 只维护最近3个版本的 Spark。

### 直接下载

可以在 [Maven Central Repository](https://repo1.maven.org/maven2/com/starrocks/starrocks-stream-load-sdk) 获取不同版本的 connector jar。

### Maven 依赖

依赖配置的格式如下，需要将 `spark_version`、`scala_version` 和 `connector_version` 替换成对应的版本。

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-${spark_version}_${scala_version}</artifactId>
  <version>${connector_version}</version>
</dependency>
```
比如，想在 Spark 3.2 和 scala 2.12 上使用 1.1.0 版本的 connector，可以添加如下依赖

```xml
<dependency>
  <groupId>com.starrocks</groupId>
  <artifactId>starrocks-spark-connector-3.2_2.12</artifactId>
  <version>1.1.0</version>
</dependency>
```

### 手动编译

1. 下载 [Spark 连接器代码](https://github.com/StarRocks/starrocks-connector-for-apache-spark)。

2. 通过如下命令进行编译，需要将 `spark_version` 替换成相应的 Spark 版本

      ```shell
      sh build.sh <spark_version>
      ```
   
   比如，在 Spark 3.2 上使用，命令如下

      ```shell
      sh build.sh 3.2
      ```

3. 编译完成后，`target/` 目录下会生成 connector jar 包，比如 `starrocks-spark-connector-3.2_2.12-1.1.0.jar`。


## 参数说明

| 参数                                            | 是否必填   | 默认值 | 描述                                                                                                                                                                                                                    |
| ---------------------------------------------- |-------- | ---- |-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| starrocks.fenodes                              | 是      | 无 | FE 的 HTTP 地址，支持输入多个FE地址，使用逗号 , 分隔。格式为 <fe_host1>:<fe_http_port1>,<fe_host2>:<fe_http_port2>。                                                                                                                          |
| starrocks.fe.jdbc.url                          | 是      | 无 | FE 的 MySQL Server 连接地址。格式为 jdbc:mysql://<fe_host>:<fe_query_port>。                                                                                                                                                    |
| starrocks.table.identifier                     | 是      | 无 | StarRocks 目标表的名称，格式为 <database_name>.<table_name>。                                                                                                                                                                    |
| starrocks.user                                 | 是      | 无 | StarRocks 集群账号的用户名。                                                                                                                                                                                                   |
| starrocks.password                             | 是      | 无 | StarRocks 集群账号的用户密码。                                                                                                                                                                                                  |
| starrocks.write.label.prefix                   | 否      | spark- | 指定Stream Load使用的label的前缀。                                                                                                                                                                                             |
| starrocks.write.enable.transaction-stream-load | 否      | true | 是否使用Stream Load的事务接口导入数据，详见[Stream Load事务接口](https://docs.starrocks.io/zh-cn/latest/loading/Stream_Load_transaction_interface)，该功能需要StarRocks 2.4及以上版本。                                                               |
| starrocks.write.buffer.size                    | 否      | 104857600 | 数据攒批的内存大小，达到该阈值后数据批量发送给 StarRocks。增大该值能提高导入性能，但会带来写入延迟。                                                                                                                                                               |
| starrocks.write.flush.interval.ms              | 否      | 300000 | 数据攒批发送的间隔，用于控制数据写入StarRocks的延迟。                                                                                                                                                                                       |
| starrocks.columns                              | 否      | 无 | 支持向 StarRocks 表中写入部分列，通过该参数指定列名，多个列名之间使用逗号 (,) 分隔，例如"c0,c1,c2"。                                                                                                                                                       |
| starrocks.write.properties.*                   | 否      | 无 | 指定Stream Load 的参数，控制导入行为，例如使用starrocks.write.operties.format选择Stream load使用csv或json格式。支持的参数和说明，请参见[Stream Load](https://docs.starrocks.io/zh-cn/latest/sql-reference/sql-statements/data-manipulation/STREAM%20LOAD)。 |
| starrocks.write.properties.format              | 否      | CSV | 指定stream load导入数据使用的格式，取值为CSV 和 JSON。connector会将每批数据转换成相应的格式发送给StarRocks。                                                                                                                                             |
| starrocks.write.properties.row_delimiter       | 否      | \n | 使用CSV格式导入时，用于指定行分隔符。                                                                                                                                                                                                  |
| starrocks.write.properties.column_separator    | 否      | \t | 使用CSV格式导入时，用于指定列分隔符。                                                                                                                                                                                                  |
| starrocks.write.num.partitions                 | 否      | 无 | Spark用于并行写入的分区数。数据量小时可以通过减少分区数降低导入并发和频率。默认分区数由Spark决定。                                                                                                                                                                |

## 数据类型映射

| StarRocks 数据类型 | Spark 数据类型           |
|----------------|----------------------|
| BOOLEAN        | BooleanType          |
| TINYINT        | ByteType             |
| SMALLINT       | ShortType            |
| INT            | IntegerType          |
| BIGINT         | LongType             |
| LARGEINT       | StringType           |
| FLOAT          | FloatType            |
| DOUBLE         | DoubleType           |
| DECIMAL        | DecimalType          |
| CHAR           | CharType             |
| VARCHAR        | VarcharType |
| STRING         | StringType            |
| DATE           | DateType |
| DATETIME       | TimestampType |


## 使用示例

通过一个简单的例子说明如何使用 connector 写入 StarRocks 表，包括使用 Spark DataFrame 和 Spark SQL，其中 DataFrame 包括 Batch 和 Structured Streaming 两种模式。

更多示例请参考 [Spark Connector Examples](https://github.com/StarRocks/starrocks-connector-for-apache-spark/tree/main/src/test/java/com/starrocks/connector/spark/examples)，将持续补充更多例子。

### 创建StarRocks表

创建数据库 `test`，并在其中创建名为 `score_board` 的主键表。

```SQL
CREATE DATABASE `test`;

CREATE TABLE `test`.`score_board`
(
    `id` int(11) NOT NULL COMMENT "",
    `name` varchar(65533) NULL DEFAULT "" COMMENT "",
    `score` int(11) NOT NULL DEFAULT "0" COMMENT ""
)
ENGINE=OLAP
PRIMARY KEY(`id`)
COMMENT "OLAP"
DISTRIBUTED BY HASH(`id`)
PROPERTIES (
    "replication_num" = "1"
);
```

### 使用 Spark DataFrame 写入数据

下面分别介绍在 Batch 和 Structured Streaming 下如何写入数据，示例的项目构建、编译和运行请参考 [Spark Application](https://spark.apache.org/docs/3.2.0/quick-start.html#self-contained-applications)。

#### Batch

该例子演示了在内存中构造数据并写入 StarRocks 表，编译后运行即可，然后在 StarRocks 中查询数据进行验证。

```java
// 1. create a spark session
SparkSession spark = SparkSession
        .builder()
        .master("local[1]")
        .appName("SimpleWrite")
        .getOrCreate();

// 2. create a source DataFrame from a list of data, and define
// the schema which is mapped to the StarRocks table
List<Row> data = Arrays.asList(
        RowFactory.create(1, "row1", 1),
        RowFactory.create(2, "row2", 2)
);
StructType schema = new StructType(new StructField[] {
        new StructField("id", DataTypes.IntegerType, false, Metadata.empty()),
        new StructField("name", DataTypes.StringType, true, Metadata.empty()),
        new StructField("score", DataTypes.IntegerType, true, Metadata.empty())
});
Dataset<Row> df = spark.createDataFrame(data, schema);

// 3. create starrocks writer with the necessary options.
// The format for the writer is "starrocks"

Map<String, String> options = new HashMap<>();
// FE http url like "127.0.0.1:11901"
options.put("starrocks.fenodes", "xxxxxx");
// FE jdbc url like "jdbc:mysql://127.0.0.1:11903"
options.put("starrocks.fe.jdbc.url", "xxxxxx");
// table identifier
options.put("starrocks.table.identifier", "test.score_board");
// starrocks username
options.put("starrocks.user", "root");
// starrocks password
options.put("starrocks.password", "");

// The format should be "starrocks"
df.write().format("starrocks")
        .mode(SaveMode.Append)
        .options(options)
        .save();

spark.stop();
```

#### Structured Streaming

该例子演示了通过 socket 持续接收数据，并写入 StarRocks，该示例参考 [Spark WordCount](https://spark.apache.org/docs/3.2.0/structured-streaming-programming-guide.html#overview)。
运行步骤如下
1. 在一个终端上运行 `Netcat` 工具，启动一个 server 用来接收数据，命令如下
```shell
$ nc -lk 9999
```

2. 运行示例，其中的 `socket` DataSource 将连接到 `Netcat` server，并接收数据
3. 在 `Netcat` 的终端输入多行 csv 格式的数据
```text
3,row3,3
4,row4,4
```
4. 停止 server，并在 StarRocks 中查询验证结果。 


```java
 // 1. create a spark session
SparkSession spark = SparkSession
        .builder()
        .master("local[1]")
        .appName("SimpleWrite")
        .getOrCreate();

 // 2. create a streaming source DataFrame from a server, it will receive
 // text lines continuously. Each line is a row in starrocks table, and in
 // csv format such as "3,row3,3"
 Dataset<Row> lines = spark.readStream()
        .format("socket")
        .option("host", "localhost")
        .option("port", 9999)
        .load();

// 3. split each csv line to a row, and create a DataFrame with the schema
// mapped to the StarRocks table
StructType schema = new StructType(new StructField[] {
        new StructField("id", DataTypes.IntegerType, false, Metadata.empty()),
        new StructField("name", DataTypes.StringType, false, Metadata.empty()),
        new StructField("score", DataTypes.IntegerType, false, Metadata.empty())
});
Dataset<Row> df = lines.as(Encoders.STRING())
        .map((MapFunction<String, Row>) line -> RowFactory.create(line.split(",")))
        .schema(schema);

// 3. create starrocks writer with the necessary options.
// The format for the writer is "starrocks"

Map<String, String> options = new HashMap<>();
// FE http url like "127.0.0.1:11901"
options.put("starrocks.fenodes", "xxxxxx");
// FE jdbc url like "jdbc:mysql://127.0.0.1:11903"
options.put("starrocks.fe.jdbc.url", "xxxxxx");
// table identifier
options.put("starrocks.table.identifier", "xxxxxx");
// starrocks username
options.put("starrocks.user", "xxxxxx");
// starrocks password
options.put("starrocks.password", "xxxxxx");

StreamingQuery query = df.writeStream()
        // The format should be "starrocks"
        .format("starrocks")
        .outputMode(OutputMode.Append())
        .options(options)
        // set your checkpoint location
        .option("checkpointLocation", "xxxxxx")
        .start();

query.awaitTermination();

spark.stop();
```

### 使用 Spark SQL 写入数据

该例子演示使用 `INSERT INTO` 写入数据，可以通过 [Spark SQL CLI](https://spark.apache.org/docs/latest/sql-distributed-sql-engine-spark-sql-cli.html) 运行该示例。

```SQL
-- 1. create a table using datasource "starrocks"
CREATE TABLE `score_board`
USING starrocks
OPTIONS(
   "starrocks.fenodes"="127.0.0.1:11901",
   "starrocks.fe.jdbc.url"="jdbc:mysql://127.0.0.1:11903",
   "starrocks.table.identifier"="test.score_board",
   "starrocks.user"="xxxxxx",
   "spark.starrocks.password"="xxxxxx"
);

-- 2. insert two rows into the table
INSERT INTO `score_board` VALUES (5, "row5", 5), (6, "row6", 6);
```