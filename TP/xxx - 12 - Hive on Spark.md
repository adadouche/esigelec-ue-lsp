# Hive on Spark

## Goal

In this tutorial, you will use Hive with Spark as an execution engine.

Hive supports MapReduce/Yarn and Tez as execution as well.

The main steps are:

  - Configure Hive to use Spark execution engine
  - Start HDFS/YARN processes
  - Start Spark processes
  - Use BeeLine to execute queries

Most of the configuration steps are also detailed here:

 - https://cwiki.apache.org/confluence/display/Hive/Hive+on+Spark%3A+Getting+Started

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

git reset --hard origin/new-step-11
git clean -dfq

./.setup.sh
```

## Start HDFS & YARN processes

In order to reduce the resource footprint, you will start using a ***smaller** Hadoop cluster:

- 1 Name Node
- 1 Data Node
- 1 Resource Manager
- 1 Node Manager

You can kill your HDFS & YARN processes using the following command:

```sh
kill -9 $(jps -mlV | grep -E 'Node|Manager|NodeManager' | awk '{ print $1 }')
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

sleep 30

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon start resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon start nodemanager
```

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

You can also get details about HDFS processes using the following URL:

 - Name Node : http://localhost:9870/
 - Resource Manager	: http://localhost:8088/

## Configure Hive - Create Hive Warehouse and temporary directory in HDFS

In your Ubuntu terminal, execute the following commands:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /tmp
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /user/hive/warehouse
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -chmod -R g+w /tmp
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -chmod -R g+w /user/hive/warehouse
```

## Configure Spark - Jar Cache

Create a new spark-defaults.conf file in the Hive configuration folder using the following command:

```sh
nano $SPARK_HOME/conf/spark-defaults.conf
```

Then add the following content

```sh
spark.yarn.jars                     hdfs://localhost:9000/spark-jars/*
spark.sql.hive.metastore.version    2.3.0
spark.sql.hive.metastore.jars       ~/esigelec-ue-lsp-hdp/hive-*/lib:~/esigelec-ue-lsp-hdp/hadoop-*/share/hadoop/client/*
```

## Configure Spark - Cache Spark Jars in HDFS

In order to work efficiently, you should cache all Spark Jars in HDFS and provide the cache location to Spark for YARN to use it.

To do execute the following to upload all Spark Jars in HDFS:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /spark-jars

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put $SPARK_HOME/jars/* /spark-jars/
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -rm /spark-jars/*hive*1.2.1*
```

## Configure Hive - Initialize the Hive Meta store

You can cleanup previous experiments using the following commands:

```sh
rm -rf $HIVE_HOME/metastore_db
```

The Schema Tool is a offline command line tool to manage the Hive Meta Store.

Then, make sure to start the next commands in the $HIVE_HOME directory as described below.

In your Ubuntu terminal, execute the following commands:

```sh
schematool -dbType derby -initSchema
```

## Configure Hive - Link Spark Jars in Hive

In order to properly work, you will need to add Spark Jars to the Hive lib folder using the following command:

```sh
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/scala-library*.jar                   ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/spark-core*.jar                      ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/spark-network-common*.jar            ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/spark-unsafe*.jar                    ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/chill*.jar                           ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/jackson-module-paranamer*.jar        ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/jackson-module-scala*.jar            ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/jersey-container-servlet-core*.jar   ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/jersey-server*.jar                   ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/json4s-ast*.jar                      ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/kryo-shaded*.jar                     ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/minlog*.jar                          ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/scala-xml*.jar                       ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/spark-launcher*.jar                  ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/spark-network-shuffle*.jar           ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/spark-unsafe*.jar                    ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
ln -sf ~/esigelec-ue-lsp-hdp/spark-*/jars/xbean-asm7-shaded*.jar               ~/esigelec-ue-lsp-hdp/hive-3.1.2/lib/
```

## Start Spark Master & Slave processes

You can check that your Spark Master & Slave processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

If the command returns the following, then you don't need to start the Spark Master & Slave processes again:
- 1 Master
- 1 Worker

If you need to start the Spark Master & Slave processes, execute the following commands:

```sh
start-master.sh
start-slaves.sh
```

> ### **Note:**
> if you receive the following error message:
```
localhost: ssh: connect to host localhost port 22: Connection refused
```
> then execute the following commands to restart ssh:
```sh
sudo service ssh restart
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

You can also get details about HDFS processes using the following URL:

- Spark Web UI : http://localhost:8080/

## Start Hive

In your **Ubuntu** terminal, execute the following commands:

```sh
mkdir -p $HIVE_HOME/logs

export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop-client

hiveserver2 &> $HIVE_HOME/logs/hiveserver2.log &
```

<b><span style="color: red">
As you can notice, we set the `HADOOP_CONF_DIR` before starting the Hiver server in order to make it a new client to the HDFS.
If you omit this, then the default `etc/hadoop/core-site` and `hdfs-site.xml` files will be loaded.
</span></b>

You can check the status of your Hive instance using the following links:

- Hive : http://localhost:10002/

> ### **Note:**
> The above URL might take a while to become available, so be patient.

You can also check that the process is running using the following command in your other Ubuntu terminal:

```sh
jps -mlV | grep HiveServer2
```

## Create A Table With a Larger Dataset

Download and unzip a local copy of the following link:

- http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

You can use the following commands to download:

```sh
cd ~
wget http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

7z e '1500000 Sales Records.7z'

mv 1500000\ Sales\ Records.csv 1500000_Sales_Records.csv
```

And finally, let's import it with inline configuration properties:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put ~/1500000_Sales_Records.csv  /1500000_Sales_Records.csv
```

Open a new Beeline session using the following command:

```
beeline -u jdbc:hive2://localhost:10000/default
```

Then, you can create the table

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
You can notice that the tables are now also present in HDFS but empty:

 - http://localhost:9870/explorer.html#/user/hive/warehouse

Then to load data into a Hive table, you can execute the following statement in your beeline session:

```sql
LOAD DATA INPATH '/1500000_Sales_Records.csv' OVERWRITE INTO TABLE sales_records;
```

And naturally, you can now check the row count with:

```sql
select count(*) as cnt from sales_records;
```

This however uses the "Hive-on-MR" execution engine.

To switch to the Spark execution you can execute the following command:

```sql
set hive.execution.engine=spark;
```

Now, let's sum the total revenue & total cost per region:

```sql
select region, sum(total_revenue) as total_revenue, sum(total_cost) as total_cost from sales_records group by region;
```

The result should be:

```
INFO  : Compiling command(queryId=hadoop_20191130191652_4fee5220-c194-4d03-acd1-e7e828e7c40a): select region, sum(total_revenue) as total_revenue, sum(total_cost) as total_cost from sales_records group by region
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:region, type:string, comment:null), FieldSchema(name:total_revenue, type:decimal(18,2), comment:null), FieldSchema(name:total_cost, type:decimal(18,2), comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hadoop_20191130191652_4fee5220-c194-4d03-acd1-e7e828e7c40a); Time taken: 0.127 seconds
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Executing command(queryId=hadoop_20191130191652_4fee5220-c194-4d03-acd1-e7e828e7c40a): select region, sum(total_revenue) as total_revenue, sum(total_cost) as total_cost from sales_records group by region
WARN  : Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
INFO  : Query ID = hadoop_20191130191652_4fee5220-c194-4d03-acd1-e7e828e7c40a
INFO  : Total jobs = 1
INFO  : Launching Job 1 out of 1
INFO  : Starting task [Stage-1:MAPRED] in serial mode
INFO  : Number of reduce tasks not specified. Estimated from input data size: 1
INFO  : In order to change the average load for a reducer (in bytes):
INFO  :   set hive.exec.reducers.bytes.per.reducer=<number>
INFO  : In order to limit the maximum number of reducers:
INFO  :   set hive.exec.reducers.max=<number>
INFO  : In order to set a constant number of reducers:
INFO  :   set mapreduce.job.reduces=<number>
INFO  : Cannot run job locally: Input Size (= 187182221) is larger than hive.exec.mode.local.auto.inputbytes.max (= 134217728)
INFO  : number of splits:1
INFO  : Submitting tokens for job: job_1575135834364_0005
INFO  : Executing with tokens: []
INFO  : The url to track the job: http://localhost:8088/proxy/application_1575135834364_0005/
INFO  : Starting Job = job_1575135834364_0005, Tracking URL = http://localhost:8088/proxy/application_1575135834364_0005/
INFO  : Kill Command = /home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/bin/mapred job  -kill job_1575135834364_0005
INFO  : Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
INFO  : 2019-11-30 19:17:00,821 Stage-1 map = 0%,  reduce = 0%
INFO  : 2019-11-30 19:17:08,993 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 6.37 sec
INFO  : 2019-11-30 19:17:14,110 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 8.59 sec
INFO  : MapReduce Total cumulative CPU time: 8 seconds 590 msec
INFO  : Ended Job = job_1575135834364_0005
INFO  : MapReduce Jobs Launched:
INFO  : Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 8.59 sec   HDFS Read: 187270536 HDFS Write: 509 SUCCESS
INFO  : Total MapReduce CPU Time Spent: 8 seconds 590 msec
INFO  : Completed executing command(queryId=hadoop_20191130191652_4fee5220-c194-4d03-acd1-e7e828e7c40a); Time taken: 23.804 seconds
INFO  : OK
INFO  : Concurrency mode is disabled, not creating a lock manager
+------------------------------------+-----------------+-----------------+
|               region               |  total_revenue  |   total_cost    |
+------------------------------------+-----------------+-----------------+
| Asia                               | 47788808588.10  | 52192373548.10  |
| Australia and Oceania              | 26590908783.11  | 29017960230.04  |
| Central America and the Caribbean  | 35463565263.58  | 38601733155.42  |
| Europe                             | 85285130661.07  | 92523183021.75  |
| Middle East and North Africa       | 40848153289.21  | 44424455983.84  |
| North America                      | 7070067658.23   | 7691466663.48   |
| Sub-Saharan Africa                 | 85235092189.84  | 92890575433.98  |
+------------------------------------+-----------------+-----------------+
```

Or even :

```sql
select region, sum(total_revenue) as total_revenue, sum(total_cost) as total_cost from sales_records where region = 'Asia' group by region;
```

The result should be:

```
INFO  : Compiling command(queryId=hadoop_20191130191740_48db9270-fea6-4f2f-9130-38e2523f92bd): select region, sum(total_revenue) as total_revenue, sum(total_cost) as total_cost from sales_records where region = 'Asia' group by region
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:region, type:string, comment:null), FieldSchema(name:total_revenue, type:decimal(18,2), comment:null), FieldSchema(name:total_cost, type:decimal(18,2), comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hadoop_20191130191740_48db9270-fea6-4f2f-9130-38e2523f92bd); Time taken: 0.12 seconds
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Executing command(queryId=hadoop_20191130191740_48db9270-fea6-4f2f-9130-38e2523f92bd): select region, sum(total_revenue) as total_revenue, sum(total_cost) as total_cost from sales_records where region = 'Asia' group by region
WARN  : Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
INFO  : Query ID = hadoop_20191130191740_48db9270-fea6-4f2f-9130-38e2523f92bd
INFO  : Total jobs = 1
INFO  : Launching Job 1 out of 1
INFO  : Starting task [Stage-1:MAPRED] in serial mode
INFO  : Number of reduce tasks not specified. Estimated from input data size: 1
INFO  : In order to change the average load for a reducer (in bytes):
INFO  :   set hive.exec.reducers.bytes.per.reducer=<number>
INFO  : In order to limit the maximum number of reducers:
INFO  :   set hive.exec.reducers.max=<number>
INFO  : In order to set a constant number of reducers:
INFO  :   set mapreduce.job.reduces=<number>
INFO  : Cannot run job locally: Input Size (= 187182221) is larger than hive.exec.mode.local.auto.inputbytes.max (= 134217728)
INFO  : number of splits:1
INFO  : Submitting tokens for job: job_1575135834364_0006
INFO  : Executing with tokens: []
INFO  : The url to track the job: http://localhost:8088/proxy/application_1575135834364_0006/
INFO  : Starting Job = job_1575135834364_0006, Tracking URL = http://localhost:8088/proxy/application_1575135834364_0006/
INFO  : Kill Command = /home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/bin/mapred job  -kill job_1575135834364_0006
INFO  : Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
INFO  : 2019-11-30 19:17:49,169 Stage-1 map = 0%,  reduce = 0%
INFO  : 2019-11-30 19:17:59,373 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 6.64 sec
INFO  : 2019-11-30 19:18:04,470 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 9.09 sec
INFO  : MapReduce Total cumulative CPU time: 9 seconds 90 msec
INFO  : Ended Job = job_1575135834364_0006
INFO  : MapReduce Jobs Launched:
INFO  : Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 9.09 sec   HDFS Read: 187271893 HDFS Write: 134 SUCCESS
INFO  : Total MapReduce CPU Time Spent: 9 seconds 90 msec
INFO  : Completed executing command(queryId=hadoop_20191130191740_48db9270-fea6-4f2f-9130-38e2523f92bd); Time taken: 24.675 seconds
INFO  : OK
INFO  : Concurrency mode is disabled, not creating a lock manager
+---------+-----------------+-----------------+
| region  |  total_revenue  |   total_cost    |
+---------+-----------------+-----------------+
| Asia    | 47788808588.10  | 52192373548.10  |
+---------+-----------------+-----------------+
```

You can now quit beeline using the following command (with a space between the ! and q):

```
! q
```

## Stop Hive processes

You can check that your Hive processes are started using the following command:

```sh
jps -mlV | grep HiveServer2$
```

If the command returns any entries, then you need to stop the Hive processes.

Unfortunately, there is no graceful way of doing it so you will need to kill the process using the following commands:

```sh
kill -9 $(jps -mlV | grep HiveServer2$ | awk '{ print $1 }')
```

You can check that your HDFS processes are stopped using the following command which should return no results:

```sh
jps | grep HiveServer2$
```

## Stop HDFS & YARN processes

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

If the command returns any entries, then you need to stop the HDFS & YARN processes using the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon stop namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon stop resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon stop nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm --daemon stop nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm --daemon stop nodemanager
```

You can check that your HDFS & YARN processes are stopped using the following command which should return no results:

```sh
jps | grep -E 'Node|Manager'$
```

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill $(jps -mlV | grep -E 'Node|Manager|NodeManager' | awk '{ print $1 }')
```
