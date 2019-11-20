# Import & Query Data in Hive

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **esigelec-ue-lsp-hdp** in your Ubuntu home directory (**~**).

Open an **Ubuntu** terminal and execute:

```sh
cd ~
git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git
```

Now checkout the current step branch:

```sh
cd ~/esigelec-ue-lsp-hdp

git reset --hard origin/new-step-07
git clean -dfq
```

## Start HDFS processes

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
```

If the command returns 1 NameNode and 3 DataNode entries, then you don't need to start the HDFS processes again.

If you need to start the HDFS processes, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

rm -rf $HADOOP_HOME/tmp
rm -rf $HADOOP_HOME/data
rm -rf $HADOOP_HOME/pid
rm -rf $HADOOP_HOME/logs

hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode -format -force

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn  
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-nn  
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon start namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon start datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-2-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn --daemon start datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-3-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon start datanode
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
```

You can also get details about HDFS processes using the following URL:

 - http://localhost:9870/

## Start YARN processes

You can check that your YARN processes are started using the following command:

```sh
jps | grep Manager$
```

If the command returns 1 ResouceManager and 3 NodeManager entries, then you don't need to start the YARN processes again.

If you need to start the YARN processes, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon start resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon start nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm --daemon start nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-3-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm --daemon start nodemanager
```

You can check that the processes are started by using the following command:

```sh
jps | grep Manager$
```

You can also get details about HDFS processes using the following URL:

 - Resource Manager	: http://localhost:8088/

## Start Hive

In your **Ubuntu** terminal, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hive_env.sh

cd $HIVE_HOME

mkdir -p $HIVE_HOME/logs

hiveserver2 &> $HIVE_HOME/logs/hiveserver2.log &
```

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
beeline -u jdbc:hive2://localhost:10000/default
```

> If you receive the following error while connecting with beeline, you may need to wait a little more for the Hive process to get started:
> ```
Could not open connection to the HS2 server.
```


> **To quit beeline CLI, you will need to type ```! q``` with a space in between the ! and q.**

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
INFO  : Compiling command(queryId=hadoop_20191120085246_ef3a9305-2f62-4dcf-882c-c43c4b2a3c67): DESCRIBE pokes
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:col_name, type:string, comment:from deserializer), FieldSchema(name:data_type, type:string, comment:from deserializer), FieldSchema(name:comment, type:string, comment:from deserializer)], properties:null)
INFO  : Completed compiling command(queryId=hadoop_20191120085246_ef3a9305-2f62-4dcf-882c-c43c4b2a3c67); Time taken: 0.037 seconds
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Executing command(queryId=hadoop_20191120085246_ef3a9305-2f62-4dcf-882c-c43c4b2a3c67): DESCRIBE pokes
INFO  : Starting task [Stage-0:DDL] in serial mode
INFO  : Completed executing command(queryId=hadoop_20191120085246_ef3a9305-2f62-4dcf-882c-c43c4b2a3c67); Time taken: 0.038 seconds
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
LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
```

The data will be laoded from a local file (not stored in HDFS).

And naturally, you can now check the row count with:

```sql
select count(*) as cnt from pokes;
```

The output should look like this:

```
INFO  : Compiling command(queryId=hadoop_20191120085009_376df27b-6a29-4250-ab01-6dde0c4362dc): select count(*) as cnt from pokes
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:cnt, type:bigint, comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hadoop_20191120085009_376df27b-6a29-4250-ab01-6dde0c4362dc); Time taken: 0.103 seconds
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Executing command(queryId=hadoop_20191120085009_376df27b-6a29-4250-ab01-6dde0c4362dc): select count(*) as cnt from pokes
WARN  : Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
INFO  : Query ID = hadoop_20191120085009_376df27b-6a29-4250-ab01-6dde0c4362dc
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
INFO  : Submitting tokens for job: job_local1642501879_0003
INFO  : Executing with tokens: []
INFO  : The url to track the job: http://localhost:8080/
INFO  : Job running in-process (local Hadoop)
INFO  : 2019-11-20 08:50:10,653 Stage-1 map = 100%,  reduce = 100%
INFO  : Ended Job = job_local1642501879_0003
INFO  : MapReduce Jobs Launched:
INFO  : Stage-Stage-1:  HDFS Read: 58120 HDFS Write: 0 SUCCESS
INFO  : Total MapReduce CPU Time Spent: 0 msec
INFO  : Completed executing command(queryId=hadoop_20191120085009_376df27b-6a29-4250-ab01-6dde0c4362dc); Time taken: 1.382 seconds
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
cd ~
wget http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

7z e '1500000 Sales Records.7z'

mv 1500000\ Sales\ Records.csv $HIVE_HOME/1500000_Sales_Records.csv
```

And finally, let's import it with inline configuration properties:

```sh
hdfs dfs -Ddfs.replication=3 -Ddfs.block.size=10m -put ~/1500000_Sales_Records.csv  /1500000_Sales_Records.csv
```

Open a new Beeline session using the following command:

```
cd $HIVE_HOME

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
+----------+
|   cnt    |
+----------+
| 1500000  |
+----------+
```

Now, let's sum the total revenue & total cost per region:

```sql
select region, sum(total_revenue), sum(total_cost) from sales_records group by region;
```

The result should be:

```
+------------------------------------+-----------------+-----------------+
|               region               |       _c1       |       _c2       |
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
select region, sum(total_revenue), sum(total_cost) from sales_records where region = 'Asia' group by region;
```

The result should be:

```
+---------+-----------------+-----------------+
| region  |       _c1       |       _c2       |
+---------+-----------------+-----------------+
| Asia    | 47788808588.10  | 52192373548.10  |
+---------+-----------------+-----------------+
```

You can now quit beeline using the following command (with a space between the ! and q):

```
! q
```


## Stop HDFS processes

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
```

If the command returns any entries, then you need to stop the HDFS processes using the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon stop namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon stop datanode
```

You can check that your HDFS processes are stopped using the following command which should return no results:

```sh
jps | grep Node$
```

## Stop YARN processes

You can check that your YARN processes are started using the following command:

```sh
jps | grep Manager$
```

If the command returns any entries, then you need to stop the YARN processes using the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon stop resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon stop nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm --daemon stop nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm --daemon stop nodemanager
```

You can check that your YARN processes are stopped using the following command which should return no results:

```sh
jps | grep Manager$
```


## Stop Hive processes

You can check that your Hive processes are started using the following command:

```sh
jps -mlV | grep HiveServer2$
```

If the command returns any entries, then you need to stop the Hive processes.

Unfortunately, there is no gracefull way of doing it so you will need to kill the process using the following commands:

```sh
kill $(jps -mlV | grep HiveServer2$ | awk '{ print $1 }')
```

You can check that your HDFS processes are stopped using the following command which should return no results:

```sh
jps | grep HiveServer2$
```

## More Examples

For more examples, you can check the following links:

 - [DDL Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DDLOperations)
 - [DML Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DMLOperations)
 - [SQL Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SQLOperations)
 - [Simple Example Use Cases](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SimpleExampleUseCases)
