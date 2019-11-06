# Upload your first file in HDFS

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

set HADOOP_BIN_PATH=%HADOOP_HOME%\bin
set HADOOP_CONF_DIR=%HADOOP_HOME%\etc\hadoop

set JAVA_HOME=C:\Progra~1\Java\jdk1.8.0_221
set PATH=%PATH%;%HADOOP_BIN_PATH%

cd %HADOOP_HOME%

.\etc\hadoop\hadoop-env.cmd
```

## Interact with the File System

Let's first create a directory:

```sh
.\bin\hdfs dfs -mkdir /config
```

Now check the directory is created :

```sh
.\bin\hdfs dfs -ls /
```

Now let's copy the local configuration file into it

```sh
.\bin\hdfs dfs -put %HADOOP_HOME%\etc\hadoop\*.xml /config
```

And finally, let's check the files are here:

```sh
.\bin\hdfs dfs -ls /config
```

And visualize one:

```sh
hdfs dfs -cat /config/yarn-site.xml
```

## Check the NameNode Server

You can also get details about HDFS and the NameNode using the following URL:

 - http://localhost:50070/

And access the file system:

 - http://localhost:50070/explorer.html#/

You can also explore the folders stored in **C:\MyWork\hadoop\tmp**
