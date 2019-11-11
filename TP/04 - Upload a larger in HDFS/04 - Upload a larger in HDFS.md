# Upload a larger in HDFS

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **C:\hadoop**.

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop

git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git %HADOOP_HOME%
```

Now checkout the current step branch:

```
cd %HADOOP_HOME%

git fetch --all
git reset --hard origin/step-03
git clean -dfq
```

## Set your Hadoop & Java Home

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_221
```

## Start HDFS

If the HDFS processes are not started yet, you will need to start.

In your DOS command prompt, execute the following commands to set the environment variables:

```
%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd
```

Then, execute the following commands:

```
rd /S /Q %HADOOP_HOME%\tmp
rd /S /Q %HADOOP_HOME%\data

hdfs --config %HADOOP_HOME%\etc\hadoop-master-nn namenode -format -force

start "hdfs - master namenode" hdfs --config %HADOOP_HOME%\etc\hadoop-master-nn namenode

start "hdfs - slave-1 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-1-dn datanode
start "hdfs - slave-2 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-2-dn datanode
start "hdfs - slave-3 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-3-dn datanode
```

## Interact with the File System

Download and unzip a local copy of the following link:

- http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

You can use the following commands to download:

```sh
powershell Invoke-WebRequest -OutFile 1500000_Sales_Records.7z http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z
```

Now, you need to extract it and rename the output file like **1500000_Sales_Records.csv** instead of **1500000 Sales Records.csv**.

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
