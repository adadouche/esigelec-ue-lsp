# Upload a larger in HDFS

## Prerequisites

You will need to have a Hadoop tar ball extracted and properly configured in:

```sh
C:\MyWork\hadoop
```

The DFS process must be started.

## Set the environment variables

Open a command prompt, then execute the following commands to setup the enironment variables :

```sh
set HADOOP_HOME=C:\MyWork\hadoop

cd %HADOOP_HOME%

.\etc\hadoop\hadoop-env.cmd
```

## Interact with the File System

Download and unzip a local copy of the following link:

- http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

Rename the output file like **1500000_Sales_Records.csv** instead of **1500000 Sales Records.csv**.

Now, let's import it:

```sh
.\bin\hdfs dfs -put %HADOOP_HOME%\1500000_Sales_Records.csv /1500000_Sales_Records.default.csv
```

Here you can notice that the full path is required and no configuration file is provided.

Now, let's import it with a configuration file:

```sh
.\bin\hdfs dfs -conf .\etc\hadoop\hdfs-site.client.xml -put %HADOOP_HOME%\1500000_Sales_Records.csv /1500000_Sales_Records.client.csv
```

And finally, let's import it with inline configuration properties:

```sh
.\bin\hdfs dfs -Ddfs.replication=3 -Ddfs.block.size=10m -put %HADOOP_HOME%\1500000_Sales_Records.csv /1500000_Sales_Records.param.csv
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

You can also explore the folders stored in **C:\MyWork\hadoop\tmp**
