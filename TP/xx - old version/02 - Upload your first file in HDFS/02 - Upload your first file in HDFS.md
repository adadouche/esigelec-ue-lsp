# Upload your first file in HDFS

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
git reset --hard origin/step-01
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

hdfs namenode -format -force

start-dfs
```

## Interact with the File System

Let's first create a directory:

```sh
hdfs dfs -mkdir /config
```

Now check the directory is created :

```sh
hdfs dfs -ls /
```

Now let's copy the local configuration file into it

```sh
hdfs dfs -put %HADOOP_HOME%\etc\hadoop\*.xml /config
```

And finally, let's check the files are here:

```sh
hdfs dfs -ls /config
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

You can also explore the folders stored in **%HADOOP_HOME%\tmp**
