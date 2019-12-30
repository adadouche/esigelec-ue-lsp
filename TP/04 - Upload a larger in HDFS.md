# Upload a larger in HDFS

## Goal

In this tutorial, using your ***simulated*** a multi node Hadoop cluster, you will start uploading a larger file.

In this tutorial, you will start your cluster using ***daeomn*** processes (background processes) instead of separate shell windows as in the last tutorial.

You can find the full list of commands in the documentation :

- https://hadoop.apache.org/docs/r3.2.1/hadoop-project-dist/hadoop-common/FileSystemShell.html

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

git reset --hard origin/new-step-03
git clean -dfq

./.setup.sh
```

## Start HDFS processes

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
```

If the command returns the following, then you don't need to start the HDFS processes again:
 - 1 Name Node
 - 3 Data Node

If you need to start the HDFS & YARN processes, execute the following commands:

```sh
# First we reformat the Name Node to avoid inconsistency issues
rm -rf $HADOOP_HOME/tmp/* $HADOOP_HOME/data/* $HADOOP_HOME/logs/* $HADOOP_HOME/pid/*
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode -format -force -clusterID local

# Then we start the processes as daemons
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

 - Name Node : http://localhost:9870/

## Interact with the File System

Download and unzip a local copy of the following link:

- http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

You can use the following commands to download:

```sh
cd $LSP_HOME
rm  $LSP_HOME/'1500000_Sales_Records.7z'

wget http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

mv 1500000\ Sales\ Records.7z 1500000_Sales_Records.7z

7z e 1500000_Sales_Records.7z

mv 1500000\ Sales\ Records.csv 1500000_Sales_Records.csv
```

Now, let's import it:

```sh
hdfs dfs -put $LSP_HOME/1500000_Sales_Records.csv /1500000_Sales_Records.default.csv
```

Here you can notice that the full path is required and no configuration file is provided.

Now, let's import it with a configuration file:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put $LSP_HOME/1500000_Sales_Records.csv /1500000_Sales_Records.client.csv
```

And finally, let's import it with inline configuration properties:

```sh
hdfs dfs -Ddfs.replication=3 -Ddfs.block.size=10m -put $LSP_HOME/1500000_Sales_Records.csv  /1500000_Sales_Records.param.csv
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

 - http://localhost:9870/explorer.html#/

You can notice that the files have different settings for the replica and the block size.

## Stop HDFS processes

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
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
```

You can check that your HDFS processes are stopped using the following command which should return no results:

```sh
jps | grep Node$
```

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill $(jps -mlV | grep Node$ | awk '{ print $1 }')
```

<div style="background-color: #D3D3D3; padding: 20px;  border: 1px solid black;" >

## Quiz

Now that you have uploaded your large file and check the blocks and replica count, let's try to answer the following questions:

#### Replica

If your cluster has only 2 slaves data node running, and you upload a file requesting 3 replica.

You can stop one data node using the following command:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon stop datanode
```

How many replica will be stored on you cluster?

Now, start the 3rd slave data node using the following command:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon start datanode
```

Will the 3rd replica be created? check the Name Node Explorer: http://localhost:9870/explorer.html#/

#### Block size

If your cluster has 3 slaves data node running, and you upload a file of about a 180MB requesting 2 replica and a block size of 10MB.

How many blocks will be created? How many replica of each block will be created? How much space in total will your 180MB file be using in reality?

Now, stop one of the data node using the following command:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon stop datanode
```

Will the complete file be served? Why?

You can check the 1500000_Sales_Records.param.csv uploaded file using the following command:

```sh
hdfs fsck /1500000_Sales_Records.param.csv -files -blocks
```

#### Hint

You will need to wait 5 minutes before the Name Node realize that the Data Node is down.

So for the time being, if you try to get the file, you will potentially receive a list of Data Nodes that are down for your file.

You can adjust this setting in your hdfs-site.xml (dfs.namenode.checkpoint.period and others) or be patient.

</div
