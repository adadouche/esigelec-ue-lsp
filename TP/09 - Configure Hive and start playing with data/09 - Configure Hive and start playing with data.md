# Get Hadoop & Hive from GitHub on WSL

## Clone the Git Repository

A Git repository is available for you to download the base version of:

 - Hadoop 3.2.1
 - Hive 3.2.1

Open an Ubuntu command prompt and execute:

```sh
mkdir /mnt/c/hadoop-hive

git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git /mnt/c/hadoop-hive
```

Now checkout the current step branch:

```
cd /mnt/c/hadoop-hive

git fetch --all
git reset --hard origin/step-08
git clean -dfq
```

## Extend your BASH RC environment

In your Ubuntu terminal, execute the following commands:

First thing, you can do is to set the JAVA_HOME in the bashrc scripts so it will be initialized for every new bash session:

```sh
rm ~/.bashrc_hadoop_env

echo -e "umask 022" >> ~/.bashrc_hadoop_env
echo -e "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> ~/.bashrc_hadoop_env
echo -e "export HADOOP_HOME=/mnt/c/hadoop-hive/hadoop-3.2.1" >> ~/.bashrc_hadoop_env
echo -e "export HIVE_HOME=/mnt/c/hadoop-hive/hive-3.1.2" >> ~/.bashrc_hadoop_env

echo -e "export HIVE_CONF_DIR=\$HIVE_HOME/conf" >> ~/.bashrc_hadoop_env

echo -e "export HADOOP_BIN_PATH=\$HADOOP_HOME/bin" >> ~/.bashrc_hadoop_env
echo -e "export HADOOP_SBIN_PATH=\$HADOOP_HOME/sbin" >> ~/.bashrc_hadoop_env

echo -e "export HADOOP_CONF_DIR=\$HADOOP_HOME/etc/hadoop" >> ~/.bashrc_hadoop_env
echo -e "export HADOOP_LOG_DIR=\$HADOOP_HOME/logs" >> ~/.bashrc_hadoop_env

echo -e "export \"HADOOP_OPTS=\$HADOOP_OPTS -Dhadoop.home='\$HADOOP_HOME'\"" >> ~/.bashrc_hadoop_env
echo -e "export \"HADOOP_OPTS=\$HADOOP_OPTS -Dyarn.home='\$HADOOP_HOME'\"" >> ~/.bashrc_hadoop_env

echo -e "export PATH=\$PATH:\$HADOOP_BIN_PATH" >> ~/.bashrc_hadoop_env
echo -e "export PATH=\$PATH:\$HADOOP_SBIN_PATH" >> ~/.bashrc_hadoop_env
echo -e "export PATH=\$PATH:\$JAVA_HOME/bin" >> ~/.bashrc_hadoop_env
echo -e "export PATH=\$PATH:\$HIVE_HOME/bin" >> ~/.bashrc_hadoop_env

source ~/.bashrc_hadoop_env
```

## Start the Hadoop processes

If the Hadoop processes are not started yet, you will need to start.

Open an Ubuntu terminal, execute the following commands:

```sh
rm -rf $HADOOP_HOME/tmp
rm -rf $HADOOP_HOME/data
rm -rf $HADOOP_HOME/pid
rm -rf $HADOOP_HOME/logs

mkdir $HADOOP_HOME/tmp
mkdir $HADOOP_HOME/data
mkdir $HADOOP_HOME/pid
mkdir $HADOOP_HOME/logs

hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode -format -force

export HADOOP_PID_DIR=$HADOOP_HOME/pid/master-nn;  
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode  &> $HADOOP_HOME/logs/hadoop-master-nn.log &

export HADOOP_PID_DIR=$HADOOP_HOME/pid/slave-1-dn;
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn datanode &> $HADOOP_HOME/logs/hadoop-slave-1-dn.log &

export HADOOP_PID_DIR=$HADOOP_HOME/pid/slave-2-dn;
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn datanode &> $HADOOP_HOME/logs/hadoop-slave-2-dn.log &

export HADOOP_PID_DIR=$HADOOP_HOME/pid/slave-3-dn;
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn datanode &> $HADOOP_HOME/logs/hadoop-slave-3-dn.log &

export HADOOP_PID_DIR=$HADOOP_HOME/pid/master-rm;
yarn  --config $HADOOP_HOME/etc/hadoop-master-rm resourcemanager  &> $HADOOP_HOME/logs/hadoop-master-rm.log &

export HADOOP_PID_DIR=$HADOOP_HOME/pid/slave-1-nm;
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm nodemanager &> $HADOOP_HOME/logs/hadoop-slave-1-nm.log &

export HADOOP_PID_DIR=$HADOOP_HOME/pid/slave-2-nm;
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm nodemanager &> $HADOOP_HOME/logs/hadoop-slave-1-nm.log &

export HADOOP_PID_DIR=$HADOOP_HOME/pid/slave-3-nm;
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm nodemanager &> $HADOOP_HOME/logs/hadoop-slave-1-nm.log &
```

You can check the status of your cluster using the following links:

 - Name Node : http://localhost:9870/
 - Resource Manager	: http://localhost:8088/

You can also check that the processes are running using the following commands:

```sh
jps | grep NameNode
jps | grep DataNode

jps | grep ResourceManager
jps | grep NodeManager
```

## Create Hive Meta store and tmp directory

In your Ubuntu terminal, execute the following commands:

```sh
hadoop fs -mkdir -p /tmp
hadoop fs -mkdir -p /user/hive/warehouse
hadoop fs -chmod g+w /tmp
hadoop fs -chmod g+w /user/hive/warehouse
```

## Configure Hive

In order to configure Hive properly, execute the following commands:

```sh
echo -e "<configuration>" >  $HIVE_HOME/conf/hive-site.xml
echo -e "<property><name>hive.server2.enable.doAs</name><value>false</value></property>" >> $HIVE_HOME/conf/hive-site.xml
echo -e "</configuration>" >> $HIVE_HOME/conf/hive-site.xml
```

Then, make sure to start the next commands in the $HIVE_HOME directory:

```sh
cd $HIVE_HOME
```

On top, you can cleanup previous experiements using the following commands:

```sh
rm -rf $HIVE_HOME/metastore_db
```

## Initialize the Hive Meta store

The Schema Tool is a offline command line tool to manage the Hive Meta Store.

In your Ubuntu terminal, execute the following commands:

```sh
schematool -dbType derby -initSchema
```

You can then verify that the Meta store version:

```sh
schematool -dbType derby -info
```

## Start Hive

In order to start the Hive process, execute the following commands:

```
cd $HIVE_HOME
mkdir -p $HIVE_HOME/logs

hiveserver2 &> $HIVE_HOME/logs/hiveserver2.log &
```

You can check the status of your Hive instance using the following links:

 - Hive : http://localhost:10002/

You can also check that the process is running using the following commands:

```sh
jps -mlV | grep HiveServer2
```

## Connect to Hive using Beeline

HiveServer2  has its own CLI called Beeline.  

To run Beeline from shell, execute the following command:

```
beeline -u jdbc:hive2://localhost:10000/default
```

> If you receive the following error while connecting with beeline, you may need to wait a little more for the Hive process to get started:
> ```
Could not open connection to the HS2 server.
```
## Create Your First Table

Execute the following statements in your Beeline session:

```sql
CREATE TABLE pokes (foo INT, bar STRING);
CREATE TABLE invites (foo INT, bar STRING) PARTITIONED BY (ds STRING);
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
| invites   |
| pokes     |
+-----------+
```

Finally you can describe the table structure using the following statement:

```sql
DESCRIBE invites;
```

The response should look like this:

```
+--------------------------+------------+----------+
|         col_name         | data_type  | comment  |
+--------------------------+------------+----------+
| foo                      | int        |          |
| bar                      | string     |          |
| ds                       | string     |          |
|                          | NULL       | NULL     |
| # Partition Information  | NULL       | NULL     |
| # col_name               | data_type  | comment  |
| ds                       | string     |          |
+--------------------------+------------+----------+
```

## Load Your First Table Data

To load data into a Hive table, you can execute the followin statement in your beeline session:

```sql
LOAD DATA LOCAL INPATH './examples/files/kv1.txt' OVERWRITE INTO TABLE pokes;
LOAD DATA LOCAL INPATH './examples/files/kv2.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-15');
LOAD DATA LOCAL INPATH './examples/files/kv3.txt' OVERWRITE INTO TABLE invites PARTITION (ds='2008-08-08');
```

And naturally, you can now check the row count with:

```sql
select count(*) as cnt from pokes;
```

The output should look like this:

```
+------+
| cnt  |
+------+
| 500  |
+------+
```

You can also notice some additional logs that highlights the MapReduce Job execution:

```
INFO  : Compiling command(queryId=hadoop_20191116191536_2fee0b0d-f3a1-4842-87ab-2cbd3bae293b): select count(*) as cnt from pokes
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Semantic Analysis Completed (retrial = false)
INFO  : Returning Hive schema: Schema(fieldSchemas:[FieldSchema(name:cnt, type:bigint, comment:null)], properties:null)
INFO  : Completed compiling command(queryId=hadoop_20191116191536_2fee0b0d-f3a1-4842-87ab-2cbd3bae293b); Time taken: 0.107 seconds
INFO  : Concurrency mode is disabled, not creating a lock manager
INFO  : Executing command(queryId=hadoop_20191116191536_2fee0b0d-f3a1-4842-87ab-2cbd3bae293b): select count(*) as cnt from pokes
WARN  : Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
INFO  : Query ID = hadoop_20191116191536_2fee0b0d-f3a1-4842-87ab-2cbd3bae293b
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
INFO  : Submitting tokens for job: job_local1638650527_0002
INFO  : Executing with tokens: []
INFO  : The url to track the job: http://localhost:8080/
INFO  : Job running in-process (local Hadoop)
INFO  : 2019-11-16 19:15:38,024 Stage-1 map = 100%,  reduce = 100%
INFO  : Ended Job = job_local1638650527_0002
INFO  : MapReduce Jobs Launched:
INFO  : Stage-Stage-1:  HDFS Read: 23248 HDFS Write: 11624 SUCCESS
INFO  : Total MapReduce CPU Time Spent: 0 msec
INFO  : Completed executing command(queryId=hadoop_20191116191536_2fee0b0d-f3a1-4842-87ab-2cbd3bae293b); Time taken: 1.485 seconds
INFO  : OK
INFO  : Concurrency mode is disabled, not creating a lock manager
```

For more examples, you canck the following links:

 - [DDL Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DDLOperations)
 - [DML Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DMLOperations)
 - [SQL Operations](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SQLOperations)
 - [Simple Example Use Cases](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-SimpleExampleUseCases)
