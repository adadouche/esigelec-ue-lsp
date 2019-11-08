# Upload a larger in HDFS

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **C:\hadoop**.

Open a DOS command prompt and execute:

```sh
git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git C:\hadoop

set HADOOP_HOME=C:\hadoop

cd %HADOOP_HOME%
git checkout --track step-03
```

## Set your Hadoop Home

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop
```

## Start HDFS

If the HDFS processes are not started yet, you will need to start.

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop

%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd

start "Apache Hadoop Distribution - namenode" hdfs --config %HADOOP_HOME%\etc\hadoop-master namenode

start "Apache Hadoop Distribution - slave-1" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-1 datanode
start "Apache Hadoop Distribution - slave-2" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-2 datanode
start "Apache Hadoop Distribution - slave-3" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-3 datanode
```

## Interact with the File System

Download and unzip a local copy of the following link:

- http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

Rename the output file like **1500000_Sales_Records.csv** instead of **1500000 Sales Records.csv**.

Now, let's import it:

```sh
hdfs dfs -put %HADOOP_HOME%\1500000_Sales_Records.csv /1500000_Sales_Records.default.csv
```

Here you can notice that the full path is required and no configuration file is provided.

Now, let's import it with a configuration file:

```sh
hdfs --config %HADOOP_HOME%\etc\hadoop-client dfs -put %HADOOP_HOME%\1500000_Sales_Records.csv /1500000_Sales_Records.client.csv
```

And finally, let's import it with inline configuration properties:

```sh
hdfs dfs -Ddfs.replication=3 -Ddfs.block.size=10m -put %HADOOP_HOME%\1500000_Sales_Records.csv /1500000_Sales_Records.param.csv
```

And finally, let's check the files are here:

```sh
.\bin\hdfs dfs -ls /
```

## Access the NameNode

You can also get details about HDFS and the NameNode using the following URL:

 - http://localhost:50070/

And access the file system:

 - http://localhost:50070/explorer.html#/

You can notice that the files have different settings for the replica and the block size.

You can also explore the folders stored in **%HADOOP_HOME%\tmp** & **%HADOOP_HOME%\data**
