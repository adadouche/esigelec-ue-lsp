# Import & Query Data in Hive

## Goal

In this tutorial, using your ***simulated*** a multi node Hadoop cluster and Hive server, you will start creating tables using HDFS data and start querying them.

> #### **Note:**
>
> - The Hive process takes some time to be up and running

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

. ./scripts/git-restore.sh step-07
```

## Stop HDFS & Yarn processes

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill -9 $(jps -mlV | awk '{ print $1 }')
```

## Start HDFS & Yarn processes

For this tutorial, you will use the following HDFS & Yarn processes :
 - 1 Name Node
 - 2 Data Node
 - 1 Resource Manager
 - 2 Node Manager

To start the HDFS & Yarn processes, execute the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn  
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-nn  
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon start namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon start datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-2-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn --daemon start datanode

hdfs dfsadmin -safemode leave

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon start resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon start nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm --daemon start nodemanager
```

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

You can also get details about HDFS processes using the following URL:

 - Name Node : http://localhost:9870/
 - Resource Manager	: http://localhost:8088/

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

## Connect to Hive using Beeline

HiveServer2 has its own CLI called Beeline.  

To run Beeline from shell, execute the following command:

```
beeline -u jdbc:hive2://localhost:10000/default --hivevar lsp.home=$(echo ~)/esigelec-ue-lsp-hdp
```

> If you receive the following error while connecting with beeline, you may need to wait a little more for the Hive process to get started:
> ```
> Could not open connection to the HS2 server.
> ```

**To quit beeline CLI, you will need to type ```! q```.**

## Create Your First Table

Execute the following statements in your Beeline session:

```sql
CREATE TABLE pokes (foo INT, bar STRING);
```

You can notice that the tables are now also present in HDFS but empty:

 - http://localhost:9870/explorer.html#/user/hive/warehouse

Then execute the following statement:

```sql
SHOW TABLES;
```

The response should look like this:

```
+-----------+
| tab_name  |
+-----------+
| pokes     |
+-----------+
```

Finally you can describe the table structure using the following statement:

```sql
DESCRIBE pokes;
```

The response should look like this:

```
INFO  : Compiling command(queryId=hadoop_20191130130745_e5df88e3-9ef7-42dd-9435-3e059b2df55b): DESCRIBE pokes
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:col_name, type:string, comment:from deserializer), FieldSchema(name:data_type, type:string, comment:from deserializer), FieldSchema(name:comment, type:string, comment:from deserializer)], properties:null)
INFO  : Completed compiling command(queryId=hadoop_20191130130745_e5df88e3-9ef7-42dd-9435-3e059b2df55b); Time taken: 0.043 seconds
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Executing command(queryId=hadoop_20191130130745_e5df88e3-9ef7-42dd-9435-3e059b2df55b): DESCRIBE pokes
INFO  : Starting task [Stage-0:DDL] in serial mode
INFO  : Completed executing command(queryId=hadoop_20191130130745_e5df88e3-9ef7-42dd-9435-3e059b2df55b); Time taken: 0.031 seconds
INFO  : OK
INFO  : Concurrency mode is disabled, not creating a lock manager
+-----------+------------+----------+
| col_name  | data_type  | comment  |
+-----------+------------+----------+
| foo       | int        |          |
| bar       | string     |          |
+-----------+------------+----------+
```

## Load Your First Table Data

To load data into a Hive table, you can execute the following statement in your beeline session:

```sql
LOAD DATA LOCAL INPATH "file:///${lsp.home}/hive-3.1.2/examples/files/kv1.txt" OVERWRITE INTO TABLE pokes;
```

The data will be loaded from a local file (not stored in HDFS).

And naturally, you can now check the row count with:

```sql
select count(*) as cnt from pokes;
```

The output should look like this:

```
INFO  : Compiling command(queryId=hadoop_20191130130759_822bfe51-0dab-4d0e-9050-8251eb69a256): select count(*) as cnt from pokes
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:cnt, type:bigint, comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hadoop_20191130130759_822bfe51-0dab-4d0e-9050-8251eb69a256); Time taken: 0.11 seconds
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Executing command(queryId=hadoop_20191130130759_822bfe51-0dab-4d0e-9050-8251eb69a256): select count(*) as cnt from pokes
WARN  : Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
INFO  : Query ID = hadoop_20191130130759_822bfe51-0dab-4d0e-9050-8251eb69a256
INFO  : Total jobs = 1
INFO  : Launching Job 1 out of 1
INFO  : Starting task [Stage-1:MAPRED] in serial mode
INFO  : Number of reduce tasks determined at compile time: 1
INFO  : In order to change the average load for a reducer (in bytes):
INFO  :   set hive.exec.reducers.bytes.per.reducer=<number>
INFO  : In order to limit the maximum number of reducers:
INFO  :   set hive.exec.reducers.max=<number>
INFO  : In order to set a constant number of reducers:
INFO  :   set mapreduce.job.reduces=<number>
INFO  : number of splits:1
INFO  : Submitting tokens for job: job_local1823431497_0001
INFO  : Executing with tokens: []
INFO  : The url to track the job: http://localhost:8080/
INFO  : Job running in-process (local Hadoop)
INFO  : 2019-11-30 13:08:00,909 Stage-1 map = 100%,  reduce = 0%
INFO  : 2019-11-30 13:08:01,914 Stage-1 map = 100%,  reduce = 100%
INFO  : Ended Job = job_local1823431497_0001
INFO  : MapReduce Jobs Launched:
INFO  : Stage-Stage-1:  HDFS Read: 12642 HDFS Write: 81949535 SUCCESS
INFO  : Total MapReduce CPU Time Spent: 0 msec
INFO  : Completed executing command(queryId=hadoop_20191130130759_822bfe51-0dab-4d0e-9050-8251eb69a256); Time taken: 2.577 seconds
INFO  : OK
INFO  : Concurrency mode is disabled, not creating a lock manager
+------+
| cnt  |
+------+
| 500  |
+------+
```

If you check the Resource Manager (http://localhost:8088/cluster/apps), you will notice that there is no job submitted.

This is because the data set is really small, and the execution would have been slower compared to a local job execution.

You can now quit beeline using the following command (with a space between the ! and q):

```
! q
```

## Create A Table With a Larger Dataset

Download and unzip a local copy of the following link:

- http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

You can use the following commands to download:

```sh
cd ~/esigelec-ue-lsp-hdp

7z e 1500000_Sales_Records.7z

mv 1500000\ Sales\ Records.csv 1500000_Sales_Records.csv
```

And finally, let's import it with inline configuration properties:

```sh
hdfs dfs -Ddfs.replication=2 -Ddfs.block.size=10m -put $(echo ~)/esigelec-ue-lsp-hdp/1500000_Sales_Records.csv  /1500000_Sales_Records.csv
```

Open a new Beeline session using the following command:

```
beeline -u jdbc:hive2://localhost:10000/default --hivevar lsp.home=$(echo ~)/esigelec-ue-lsp-hdp
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
+-----------------+---------------+----------+
|    col_name     |   data_type   | comment  |
+-----------------+---------------+----------+
| region          | string        |          |
| country         | string        |          |
| item_type       | string        |          |
| sales_channel   | string        |          |
| order_priority  | string        |          |
| order_date      | string        |          |
| order_id        | bigint        |          |
| ship_date       | string        |          |
| units_sold      | int           |          |
| unit_price      | decimal(8,2)  |          |
| unit_cost       | decimal(8,2)  |          |
| total_revenue   | decimal(8,2)  |          |
| total_cost      | decimal(8,2)  |          |
| total_profit    | decimal(8,2)  |          |
+-----------------+---------------+----------+
```

To load data into a Hive table, you can execute the following statement in your beeline session:

```sql
LOAD DATA INPATH '/1500000_Sales_Records.csv' OVERWRITE INTO TABLE sales_records;
```

And naturally, you can now check the row count with:

```sql
select count(*) as cnt from sales_records;
```

The count result should be:

```
INFO  : Compiling command(queryId=hadoop_20191130130825_afa83768-901f-4857-a2c3-67b8fc95939f): select count(*) as cnt from sales_records
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:cnt, type:bigint, comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hadoop_20191130130825_afa83768-901f-4857-a2c3-67b8fc95939f); Time taken: 0.162 seconds
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Executing command(queryId=hadoop_20191130130825_afa83768-901f-4857-a2c3-67b8fc95939f): select count(*) as cnt from sales_records
WARN  : Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
INFO  : Query ID = hadoop_20191130130825_afa83768-901f-4857-a2c3-67b8fc95939f
INFO  : Total jobs = 1
INFO  : Launching Job 1 out of 1
INFO  : Starting task [Stage-1:MAPRED] in serial mode
INFO  : Number of reduce tasks determined at compile time: 1
INFO  : In order to change the average load for a reducer (in bytes):
INFO  :   set hive.exec.reducers.bytes.per.reducer=<number>
INFO  : In order to limit the maximum number of reducers:
INFO  :   set hive.exec.reducers.max=<number>
INFO  : In order to set a constant number of reducers:
INFO  :   set mapreduce.job.reduces=<number>
INFO  : Cannot run job locally: Input Size (= 187182221) is larger than hive.exec.mode.local.auto.inputbytes.max (= 134217728)
INFO  : number of splits:1
INFO  : Submitting tokens for job: job_1575110135653_0006
INFO  : Executing with tokens: []
INFO  : The url to track the job: http://localhost:8088/proxy/application_1575110135653_0006/
INFO  : Starting Job = job_1575110135653_0006, Tracking URL = http://localhost:8088/proxy/application_1575110135653_0006/
INFO  : Kill Command = /home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/bin/mapred job  -kill job_1575110135653_0006
INFO  : Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
INFO  : 2019-11-30 13:08:34,324 Stage-1 map = 0%,  reduce = 0%
INFO  : 2019-11-30 13:08:42,493 Stage-1 map = 100%,  reduce = 0%, Cumulative CPU 4.32 sec
INFO  : 2019-11-30 13:08:49,632 Stage-1 map = 100%,  reduce = 100%, Cumulative CPU 6.38 sec
INFO  : MapReduce Total cumulative CPU time: 6 seconds 380 msec
INFO  : Ended Job = job_1575110135653_0006
INFO  : MapReduce Jobs Launched:
INFO  : Stage-Stage-1: Map: 1  Reduce: 1   Cumulative CPU: 6.38 sec   HDFS Read: 187268972 HDFS Write: 107 SUCCESS
INFO  : Total MapReduce CPU Time Spent: 6 seconds 380 msec
INFO  : Completed executing command(queryId=hadoop_20191130130825_afa83768-901f-4857-a2c3-67b8fc95939f); Time taken: 25.721 seconds
INFO  : OK
INFO  : Concurrency mode is disabled, not creating a lock manager
+----------+
|   cnt    |
+----------+
| 1500000  |
+----------+
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
kill -9 $(jps -mlV | grep -E 'Node|Manager|NodeManager' | awk '{ print $1 }')
```

## More Examples

For more examples, you can check the following links:

 - [DDL Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DDLOperations)
 - [DML Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DMLOperations)
 - [SQL Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SQLOperations)
 - [Simple Example Use Cases](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SimpleExampleUseCases)


<div style="background-color: #D3D3D3; padding: 20px;  border: 1px solid black;" >

## Quiz

Now that you have run successfully your first Hive SQL queries on the Sales Record data, let's try to write a Hive SQL query that will:

- Query 1 : count the number of rows where the country is equal to `France`.

  The result should have 1 column which count the number of entries where the country is equal to `France`.

- Query 2 : sum the total profit for item type "Baby Food" per region per country

  The result should have 3 columns which returns:

   - region
   - country
   - the total profit sum only for item type "Baby Food" for the region &  country of the row

- Query 3 : sum the total profit for item type "Baby Food" & "Beverages" per region per country

  The result should have 4 columns which returns:

   - region
   - country
   - the total profit sum only for item type "Baby Food" for the region & country of the row
   - the total profit sum only for item type "Beverages" for the region & country of the row

  The goal is to compare if, in certain country, we are making more profit on Beverages than for Baby Food

- Query 4 : sum the total profit for item type "Baby Food" per region per country, sum the total profit for item type "Baby Food" per region & the count of countries in the region

   The result should have 5 columns which returns:

    - region
    - country
    - the total profit sum only for item type "Baby Food" for the region & country of the row
    - the total profit sum only for item type "Baby Food" for the region of the row
    - the count of countries for the region of the row

### Hint

For the last query you will need to take the following elements into consideration:
  - use window aggregate function
    - count(distinct col1) over (partition by col2)
    - sum(col3) over (partition by col2)
  - use the distinct keyword to remove duplicate

### Query errors

If your queries fails because with an error code 2:

  - ***FAILED: Execution Error, return code 2 from org.apache.hadoop.hive.ql.exec.mr.MapRedTask***

You can execute the following statement to limit the memory used by a reducer, and therefore potentially increase the number of reducer instead of having a bigger one:

```sql
set hive.exec.reducers.bytes.per.reducer=33554432;
```

If you want to debug your errors, you can also check the hive log file locate in:

   /tmp/hadoop/hive.log

You can also enable the explain mode using the following sql statement:

```sql
set hive.log.explain.output=true;
```

</div>
