

# 1、介绍

Pivotal Greenplum-Spark Connector 支持在 Greenplum 数据库和 Spark 集群之间高速并行传输数据，它使用：

- Spark’s Scala API 编程接口访问（包括 spark-shell）

Greenplum-Spark Connector 支持：

- 交互数据分析
- 内存分析处理
- 批量 ETL
- 持续的 ETL 管道（流式处理）

下载链接：<https://network.pivotal.io/products/pivotal-gpdb/>


支持的平台：

| Greenplum-Spark Connector Version | Greenplum Version | Spark Version | Scala Version | PostgreSQL JDBC Driver Version |
| :--- | :--- | :--- | :--- | :--- |
| 1.7.0 | 4.3.x, 5.x, 6.x | 2.3.1 and above | 2.11 | 9.4.1209 |
| 1.6.2, 1.6.1 | 4.3.x, 5.x, <=6.7 | 2.3.1 and above | 2.11 | 9.4.1209 |
| 1.6.0, 1.5.0 | 4.3.x, 5.x | 2.1.2 and above | 2.11 | 9.4.1209 |
| 1.4.0, 1.3.0, 1.2.0, 1.1.0, 1.0.0 | 4.3.x, 5.x | 2.1.1 | 2.11 | 9.4.1209 |



# 2、系统要求



## 2.1 前置条件

在使用 Greenplum-Spark Connector之前确保以下：

- 你有**管理员权限**运行 Greenplum 数据库集群。
- 你有权限运行 Spark 集群。
- 在  Greenplum Database master节点和 Spark driver 节点和每个 worker 节点之前存在网络连接。
- 在 Spark worker 节点和每个 Greenplum Database segment 主机之间存在网络连接。 

## 2.2 内存要求

参考 Spark [内存要求](https://spark.apache.org/docs/2.3.1/hardware-provisioning.html#memory)。 

## 2.3 端口要求

Greenplum Database master的端口是可以配置的，默认是5432，要确保该端口对于Spark driver 和所有 Spark worker 节点是可以访问的。 

# 3、架构

![](../assets/a477fde5fb7d1ad5a805e4cee8de4e13/gscarch.png) 

# 4、使用



## 4.1 Greenplum 配置和维护



### 4.1.1 Greenplum 配置



#### 客户端访问权限

修改 pg\_hba.conf 确保 Spark 节点有权限访问 Greenplum 集群。 

#### 角色权限

当前用户或角色对要读或者写的表所在的非公开的 schema 要有 USAGE、CREATE 权限。

```sqlite
GRANT USAGE, CREATE ON SCHEMA <schema_name> TO <user_name>;
```

当前用户或角色对于 Greenplum 中要读入到 spark 的表要有 SELECT 权限。

```sql
GRANT SELECT ON <schema_name>.<table_name> TO <user_name>;
```

要把 Greeplum 表读入到 spark，当前用户或角色要有使用 gpfdist 协议的创建有写权限的外部表。

```sql
ALTER USER <user_name> CREATEEXTTABLE(type = 'writable', protocol = 'gpfdist');
```

如果写 Greeplum 数据库的用户或者角色不是数据库或者表所有者，当前角色必须有 SELECT、INSERT 权限

    GRANT SELECT, INSERT ON <schema_name>.<table_name> TO <user_name>;

为了把 Spark 数据写入到 Greeplum，当前用户或者角色要有权限使用 gpfdist 创建一个可读的外部表。

    ALTER USER <user_name> CREATEEXTTABLE(type = 'readable', protocol = 'gpfdist');



### 4.1.2 Greenplum 维护任务

Greenplum-Spark Connector 使用 Greenplum 外部表加载 Greenplum 数据到 Spark，对这些外部表的维护包括：

- VACUUM
- ANALYZE
- 清理外部表 

## 4.2 使用 Greenplum-Spark Connector



### 4.2.1 下载 Spark

下载地址：<https://spark.apache.org/downloads.html>，下载的 spark 版本为2.4.7，scala 版本为2.12，带hadoop，下载链接为：[spark-2.4.7-bin-hadoop2.7.tgz](https://mirrors.bfsu.edu.cn/apache/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.7.tgz) 。

    sudo yum install wget -y
    wget https://mirrors.bfsu.edu.cn/apache/spark/spark-2.4.7/spark-2.4.7-bin-hadoop2.7.tgz
    tar zxvf spark-2.4.7-bin-hadoop2.7.tgz
    mv spark-2.4.7-bin-hadoop2.7 ~/soft/spark-2.4.7

配置环境变量：

```bash
cat >> ~/.bash_profile<<EOF 
#spark
export SPARK_HOME=~/soft/spark-2.4.7
export PATH=$PATH:$SPARK_HOME/bin
EOF
source ~/.bash_profile
```

安装jdk：

    sudo yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel

进入spark-shell：

    spark-shell



### 4.2.2 下载 Connector JAR

下载链接：<https://network.pivotal.io/products/pivotal-gpdb/> ，当前最新版本文件为 greenplum-spark\_2.11-1.7.0.jar ，将其保存在本地一个路径，例如：~/soft/。 

### 4.2.3 使用 spark-shell

配置环境变量：

```bash
cat >> ~/.bash_profile<<EOF 
export GSC_JAR=~/soft/greenplum-spark_2.11-1.7.0.jar
EOF
source ~/.bash_profile
```

运行spark-shell：

```bash
spark-shell --jars $GSC_JAR
```



### 4.2.4 Spark 从 Greenplum 读数据

方法1：

```scala
val gscReadOptionMap = Map(
      "url" -> "jdbc:postgresql://192.168.1.131:5432/tutorial",
      "user" -> "user2",
      "password" -> "pivotal",
      "dbschema" -> "faa",
      "dbtable" -> "otp_c",
      "partitionColumn" -> "airlineid"
)
val gpdf = spark.read.format("greenplum").options(gscReadOptionMap).load()
```

方法2：

```scala
val url = "jdbc:postgresql://192.168.1.131:5432/tutorial"
val tblname = "flightdate"
val jprops = new Properties()
jprops.put("user", "user2")
jprops.put("password", "pivotal")
jprops.put("partitionColumn", "otp_c")
val gpdf = spark.read.greenplum(url, tblname, jprops)
```



### 4.2.5 Spark 写数据到 Greenplum

```scala
import org.apache.spark.sql.SaveMode
val gscWriteOptionMap = Map(
      "url" -> "jdbc:postgresql://192.168.1.131:5432/testdb",
      "user" -> "user2",
      "password" -> "pivotal",
      "dbschema" -> "faa",
      "dbtable" -> "flightdate",
)
dfToWrite.write.format("greenplum")
      .options(gscWriteOptionMap)
		  .mode(SaveMode.Append)
      .save()
```



### 4.2.6 示例程序

**读greenplum数据**


给user2授权：

    psql -d tutorial
    tutorial=# GRANT USAGE, CREATE ON SCHEMA faa TO user2;
    tutorial=# GRANT SELECT ON faa.otp_c TO user2;
    tutorial=# ALTER USER user2 CREATEEXTTABLE(type = 'writable', protocol = 'gpfdist');
    tutorial=# \q

验证user2是否可以访问：

    psql -d tutorial -h `hostname` -U user2
    tutorial=> \dt faa.*
    tutorial=> \d faa.otp_c

运行spark-shell：

```bash
spark-shell --jars $GSC_JAR
```

读数据：

    val gscReadOptionMap = Map(
          "url" -> "jdbc:postgresql://192.168.1.131:5432/tutorial",
          "user" -> "user2",
          "password" -> "pivotal",
          "dbschema" -> "faa",
          "dbtable" -> "otp_c",
          "partitionColumn" -> "airlineid"
    )
    val gpdf = spark.read.format("greenplum").options(gscReadOptionMap).load()
    gpdf.count()
    gpdf.select("origincityname", "flt_month", "airlineid", "carrier").filter("cancelled = CAST(1 as SMALLINT)").filter("flt_month = CAST(12 as SMALLINT)").orderBy("airlineid", "origincityname").show()
               
    gpdf.groupBy("flt_dayofweek").agg(avg("depdelayminutes")).sort("flt_dayofweek").show()
    gpdf.select("origincityname", "destcityname", "flightnum", "carrier", "airlineid", "flt_month").filter("cancelled = CAST(1 as SMALLINT)").filter("flt_month = CAST(12 as SMALLINT)").filter($"origincityname".startsWith("Mi")).orderBy("origincityname", "destcityname").show()

**写数据到greenplum**


给用户授权：

```bash
psql -d tutorial
 tutorial=# ALTER USER user2 CREATEEXTTABLE(type = 'readable', protocol = 'gpfdist');
```

spark操作：将上面表中数据写入到greenplum：

```scala
val delaydf = gpdf.groupBy("flt_dayofweek").agg(avg("depdelayminutes")).sort("flt_dayofweek")
```

创建一个Map参数，将 delaydf 写入到 avgdelay 中：

```scala
val gscWriteMap = Map(      "url" -> "jdbc:postgresql://192.168.1.131:5432/tutorial",      "user" -> "user2",      "password" -> "pivotal",      "dbschema" -> "faa",      "dbtable" -> "avgdelay")import org.apache.spark.sql.SaveModedelaydf.write.format("greenplum").options(gscWriteMap).mode(SaveMode.Append).save()
```

去 greenplum查询数据：

```bash
psql -d tutorialtutorial=# select * from faa.avgdelay; flt_dayofweek | avg(depdelayminutes)---------------+----------------------             4 |      12.056892575386             1 |     14.7384915697799             3 |     11.1981982562523             2 |     11.2372720240202             6 |     12.6958636127181             5 |      12.455024249522             7 |     14.8182711926037(7 rows)
```
