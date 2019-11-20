# Upload a larger in HDFS

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

git reset --hard origin/new-step-03
git clean -dfq
```

## Set your Hadoop environment variables

In your **Ubuntu** terminal, execute:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh
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

## Interact with the File System

Download and unzip a local copy of the following link:

- http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

You can use the following commands to download:

```sh
cd ~
wget http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

7z e '1500000 Sales Records.7z'

mv 1500000\ Sales\ Records.csv 1500000_Sales_Records.csv
```

Now, let's import it:

```sh
hdfs dfs -put ~/1500000_Sales_Records.csv /1500000_Sales_Records.default.csv
```

Here you can notice that the full path is required and no configuration file is provided.

Now, let's import it with a configuration file:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put ~/1500000_Sales_Records.csv /1500000_Sales_Records.client.csv
```

And finally, let's import it with inline configuration properties:

```sh
hdfs dfs -Ddfs.replication=3 -Ddfs.block.size=10m -put ~/1500000_Sales_Records.csv  /1500000_Sales_Records.param.csv
```

Now, let's check the files are here:

```sh
hdfs dfs -ls /
```

Finally, let's check the associated blocks and replica using the following command:

```sh
hdfs fsck / -files -blocks
```

You can also get these details using the following URL:

 - http://localhost:50070/explorer.html#/

You can notice that the files have different settings for the replica and the block size.

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
