# Setup Spark

## Goal

In this tutorial, using your ***simulated*** a multi node Hadoop cluster, you will setup Spark.

The main steps are:

  - Download and un-compress the Spark Tar ball
  - Start your HDFS/YARN/MapReduce/Hive processes
  - Create the Hive Meta store and Temp directory in HDFS
  - Configure the hive-site file
  - Start the Hive process

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

git reset --hard origin/new-step-08
git clean -dfq

export ENV_FILE=~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh
source $ENV_FILE

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc

export ENV_FILE=~/esigelec-ue-lsp-hdp/.set_hive_env.sh
source $ENV_FILE

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc
```

## Download the Spark 3.0.0 preview distribution

All Spark download links can be obtained from the following:

 - http://spark.apache.org/downloads.html

The version to be downloaded is the Spark 3.0.0 preview release for Hadoop 3.2 or later.

In your **Ubuntu** terminal, execute:

```sh
cd ~
wget http://apache.mirrors.benatherton.com/spark/spark-3.0.0-preview/spark-3.0.0-preview-bin-hadoop3.2.tgz
```

## Extract the Spark 3.0.0 preview distribution into the Git Folder

In your **Ubuntu** terminal, execute:

```sh
cd ~
tar xvf spark-3.0.0-preview-bin-hadoop3.2.tgz -C ~/esigelec-ue-lsp-hdp

mv ~/esigelec-ue-lsp-hdp/spark-3.0.0-preview-bin-hadoop3.2 ~/esigelec-ue-lsp-hdp/spark-3.0.0
```

## Configure the your environment with Spark environment variables

In your **Ubuntu** terminal, execute:

```sh
export ENV_FILE=~/esigelec-ue-lsp-hdp/.set_spark_env.sh
export SPARK_HOME=$(echo ~)/esigelec-ue-lsp-hdp/spark-3.0.0

rm -f $ENV_FILE

echo -e "umask 022" > $ENV_FILE

echo -e "export LSP_HOME=$(echo ~)/esigelec-ue-lsp-hdp" >> $ENV_FILE

echo -e "export SPARK_HOME=\$LSP_HOME/spark-3.0.0" >> $ENV_FILE
echo -e "export HADOOP_HOME=\$LSP_HOME/hadoop-3.2.1" >> $ENV_FILE

echo -e "export PATH=\$PATH:\$SPARK_HOME/bin" >> $ENV_FILE
echo -e "export PATH=\$PATH:\$SPARK_HOME/sbin" >> $ENV_FILE

cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
export LINE="export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop-client"
grep -qF "$LINE" $SPARK_HOME/conf/spark-env.sh || echo -e $LINE >> $SPARK_HOME/conf/spark-env.sh

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc

source $ENV_FILE
```

## Start Spark Master & Slave processes

You can check that your Spark Master & Slave processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

If the command returns the following, then you don't need to start the Spark Master & Slave processes again:
 - 1 Master
 - 1 Worker

If you need to start the Spark Master & Slave processes, execute the following commands:

```sh
start-master.sh
start-slaves.sh
```

> ### **Note:**
> if you receive the following error message:
```
localhost: ssh: connect to host localhost port 22: Connection refused
```
> then execute the following commands to restart ssh:
```sh
sudo service ssh restart
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

You can also get details about HDFS processes using the following URL:

 - Spark Web UI : http://localhost:8080/

## Test PySpark shell

In your **Ubuntu** terminal, execute:

```sh
pyspark --version
```

This will give you the current version of the PySpark shell.

```
2019-12-01 16:28:58,279 WARN util.Utils: Your hostname, W-PF0VF55M resolves to a loopback address: 127.0.0.1; using 192.168.0.22 instead (on interface eth0)
2019-12-01 16:28:58,281 WARN util.Utils: Set SPARK_LOCAL_IP if you need to bind to another address
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.0.0-preview
      /_/

Using Scala version 2.12.10, OpenJDK 64-Bit Server VM, 1.8.0_222
Branch HEAD
Compiled by user ubuntu on 2019-10-31T02:05:23Z
Revision 007c873ae34f58651481ccba30e8e2ba38a692c4
Url https://gitbox.apache.org/repos/asf/spark.git
Type --help for more information.
```

Now, open a new PySpark session and execute the following code:

```python
input_file = sc.textFile("/wordcount/input/WordCount.java")
counts = input_file.flatMap(lambda line: line.split(" ")) \
             .map(lambda word: (word, 1)) \
             .reduceByKey(lambda a, b: a + b) \
             .toDF()
counts.show()
```

It assumes that you have uploaded the WordCount.java in HDFS.

## Stop Spark Master & Slave processes

You can check that your Spark Master & Slave processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

If the command returns any entries, then you can stop processes using the following commands:

```sh
stop-slaves.sh
stop-master.sh
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

You can also get details about HDFS processes using the following URL:

- Spark Web UI : http://localhost:8080/
