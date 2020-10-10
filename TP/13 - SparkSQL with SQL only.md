# SparkSQL with SQL only

## Goal

In this tutorial, you will start using SparkSQL to create / load / query table using SQL only.

You will be using different type of tables loaded from various format/sources.

The main steps are:

 - create and load tables from a text, csv, parquet, json files
 - use the Hive format (doen't require a Hive setup)

In this mode, the process will be executed locally, therefore you don't need to start the Spark cluster.
But you can also use the Spark Cluster or YARN mode if you want as well.

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **esigelec-ue-lsp-hdp** in your Ubuntu home directory (**~**).

If you didn't clone the repository yet, you can do so using the following command:

```sh
cd ~
git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git
```

Now checkout the current step branch:

```sh
cd ~/esigelec-ue-lsp-hdp

. ./scripts/git-restore.sh step-12
```

## Start HDFS processes

In order to reduce the resource footprint, you will start using a ***smaller** Hadoop cluster:

- 1 Name Node
- 1 Data Node

You can kill your HDFS & YARN processes using the following command:

```sh
kill -9 $(jps -mlV | grep -E 'Node' | awk '{ print $1 }')
```

Then start the HDFS & YARN processes using the following commands:

```sh
# First we reformat the Name Node to avoid inconsistency issues
rm -rf $HADOOP_HOME/tmp/* $HADOOP_HOME/data/* $HADOOP_HOME/logs/* $HADOOP_HOME/pid/*
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode -format -force -clusterID local

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn  
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-nn  
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon start namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon start datanode
```

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node'$
```

You can also get details about HDFS processes using the following URL:

 - Name Node : http://localhost:9870/

## Upload data files in HDFS

In our example, you will be using the 1500000_Sales_Records and the WordCount java program as input files.

First, download the 1500000_Sales_Records file locally and extract it:

```sh
cd $LSP_HOME
rm  $LSP_HOME/'1500000_Sales_Records.7z'

wget -nc http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

mv 1500000\ Sales\ Records.7z 1500000_Sales_Records.7z

7z e 1500000_Sales_Records.7z

mv 1500000\ Sales\ Records.csv 1500000_Sales_Records.csv
```

And finally, let's import the data files:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /wordcount/input

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put $HADOOP_HOME/mr/wordcount/src/WordCount.java /wordcount/input

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -Ddfs.block.size=10m -put $(echo ~)/esigelec-ue-lsp-hdp/1500000_Sales_Records.csv /1500000_Sales_Records.csv
```

## Connect to Spark using Spark-SQL shell with Hive variables

Spark has its own interactive shell called Spark-SQL.  

To run Spark-SQL from shell, execute the following command:

```
spark-sql --hivevar lsp.home=$(echo ~)/esigelec-ue-lsp-hdp
```

Once started, you can check the Spark local Web UI for information:

 - http://localhost:4040/jobs/

## Create your first tables using Hive format

Execute the following statements in your Beeline session:

```sql
CREATE TABLE pokes (foo INT, bar STRING) USING HIVE;
```

You can notice that the tables are now also present in HDFS but empty:

 - http://localhost:9870/explorer.html#/user/hive/warehouse

Then execute the following statement:

```sql
SHOW TABLES;
```

The response should look like this:

```
default pokes   false
```

Finally you can describe the table structure using the following statement:

```sql
DESCRIBE pokes;
```

The response should look like this:

```
foo     int     NULL
bar     string  NULL
```

To load data into a Hive table in Spark, you can execute the following statement in your session:

```sql
LOAD DATA LOCAL INPATH "file:///${lsp.home}/hive-3.1.2/examples/files/kv1.txt" OVERWRITE INTO TABLE pokes;
```

The data will be loaded from a local file (not stored in HDFS).

And naturally, you can now check the row count with:

```sql
select count(*) as cnt from pokes;
```

If you check the Spark local Web UI (http://localhost:4040/jobs/), you will notice that there are job submitted.

This is because the data set is really small, and the execution would have been slower compared to a local job execution.

You can now quit your session using the following command (with a space between the ! and q):

```
exit;
```

You will notice that  if you restart your spark-sql shell session, the table will still be there.

Open a new spark-sql session using the following command:

```
spark-sql --hivevar lsp.home=$(echo ~)/esigelec-ue-lsp-hdp
```

Then, you can execute the following statement to check that your table still exist and contains data:

```sql
select count(*) as cnt from pokes;
```

Now, you can create the Sale Record table

```sql
create table sales_records (
  region string,
  country string,
  item_type string,
  sales_channel string,
  order_priority string,
  order_date string,
  order_id bigint,
  ship_date string,
  units_sold int,
  unit_price decimal(8,2),
  unit_cost decimal(8,2),
  total_revenue decimal(8,2),
  total_cost decimal(8,2),
  total_profit decimal(8,2)
)
row format delimited fields terminated by ',' tblproperties('skip.header.line.count'='1');
```

You can notice that the tables is not using the ***USING HIVE*** construct, because the associated syntax is assoicated to Hive.

Then execute the following statement:

```sql
SHOW TABLES;
```

Finally you can describe the table structure using the following statement:

```sql
DESCRIBE sales_records;
```

which should provide the following output:

```
region  string  NULL
country string  NULL
item_type       string  NULL
sales_channel   string  NULL
order_priority  string  NULL
order_date      string  NULL
order_id        bigint  NULL
ship_date       string  NULL
units_sold      int     NULL
unit_price      decimal(8,2)    NULL
unit_cost       decimal(8,2)    NULL
total_revenue   decimal(8,2)    NULL
total_cost      decimal(8,2)    NULL
total_profit    decimal(8,2)    NULL
```

To load data into a Hive table, you can execute the following statement in your beeline session:

```sql
LOAD DATA LOCAL INPATH 'file:///${lsp.home}/1500000_Sales_Records.csv' OVERWRITE INTO TABLE sales_records;
```

And naturally, you can now check the row count with:

```sql
select count(*) as cnt from sales_records;
```

The count result should be:

```
1500001
```

**As you can notice, this is 1 more than expected because the Hive engine is not fully respected here.**

Let's see if using a different format can solve the issue.

You can now quit your session using the following command (with a space between the ! and q):

```
exit;
```
## Create your tables using the spark-csv connector/format

Let's now stat Spark-SQL with the spark-csv package

Execute the following command:

```
spark-sql --hivevar lsp.home=$(echo ~)/esigelec-ue-lsp-hdp --packages com.databricks:spark-csv_2.11:1.5.0
```

Now, let's recreate the table using the following command:

```sql
create table sales_records_spark_csv (
  region string,
  country string,
  item_type string,
  sales_channel string,
  order_priority string,
  order_date string,
  order_id bigint,
  ship_date string,
  units_sold int,
  unit_price decimal(8,2),
  unit_cost decimal(8,2),
  total_revenue decimal(8,2),
  total_cost decimal(8,2),
  total_profit decimal(8,2)
)
USING com.databricks.spark.csv
OPTIONS (path "file:///${lsp.home}/1500000_Sales_Records.csv", header "true");
```

And now, you can now check the row count with:

```sql
select count(*) as cnt from sales_records_spark_csv;
```

You can also now execute the queries from the [08 - Import & Query Data in Hive](https://github.com/adadouche/esigelec-ue-lsp/tree/master/TP/08%20-%20Import%20&%20Query%20Data%20in%20Hive.md) tutorial like the following one:

```sql
select region, sum(total_revenue) as total_revenue, sum(total_cost) as total_cost from sales_records_spark_csv group by region;
```

## Stop HDFS processes

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node'$
```

If the command returns any entries, then you need to stop the HDFS & YARN processes using the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon stop namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon stop datanode
```

You can check that your HDFS & YARN processes are stopped using the following command which should return no results:

```sh
jps | grep -E 'Node'$
```

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill -9 $(jps -mlV | grep -E 'Node' | awk '{ print $1 }')
```
