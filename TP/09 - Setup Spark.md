# Setup Spark

## Goal

In this tutorial, you will setup Spark.

The main steps are:

  - Download and un-compress the Spark Tar ball
  - Start the Spark process

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

git pull
git reset --hard origin/step-08
git clean -dfq

./.setup.sh
```

## Download the Spark distribution

All Spark download links can be obtained from the following:

 - http://spark.apache.org/downloads.html

The version to be downloaded is the Spark 3.0.0-preview2 release for Hadoop 2.7.

In your **Ubuntu** terminal, execute:

```sh
cd ~

wget http://apache.crihan.fr/dist/spark/spark-3.0.0-preview2/spark-3.0.0-preview2-bin-hadoop3.2.tgz
```

Then execute the following command to move the extracted directory:

```sh
cd ~

rm -R ~/esigelec-ue-lsp-hdp/spark-*

tar xvf ~/spark-*.tgz -C ~/esigelec-ue-lsp-hdp

mv ~/esigelec-ue-lsp-hdp/spark-* ~/esigelec-ue-lsp-hdp/spark-3.0.0
```

## Configure your environment with Spark environment variables

In your **Ubuntu** terminal, execute:

```sh
export ENV_FILE=~/esigelec-ue-lsp-hdp/.set_spark_env.sh
export SPARK_HOME=$(echo ~)/esigelec-ue-lsp-hdp/spark-3.0.0

rm -f $ENV_FILE

echo -e "umask 022" > $ENV_FILE

echo -e "export LSP_HOME=/home/\$USER/esigelec-ue-lsp-hdp" >> $ENV_FILE

echo -e "export SPARK_HOME=$SPARK_HOME" >> $ENV_FILE
echo -e "export HADOOP_HOME=$HADOOP_HOME" >> $ENV_FILE

echo -e "export PATH=\$PATH:\$SPARK_HOME/bin" >> $ENV_FILE
echo -e "export PATH=\$PATH:\$SPARK_HOME/sbin" >> $ENV_FILE

export SPARK_DIST_CLASSPATH=$(hadoop classpath)

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

## Check the PySpark version

In your **Ubuntu** terminal, execute:

```sh
pyspark --version
```

This will give you the current version of the PySpark shell.

```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/esigelec-ue-lsp-hdp/spark-3.0.0/jars/slf4j-log4j12-1.7.16.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
2019-12-31 15:44:11,041 WARN util.Utils: Your hostname, W-PF0VF55M resolves to a loopback address: 127.0.0.1; using 172.23.79.153 instead (on interface eth0)
2019-12-31 15:44:11,042 WARN util.Utils: Set SPARK_LOCAL_IP if you need to bind to another address
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.0.0-preview2
      /_/

Using Scala version 2.12.10, OpenJDK 64-Bit Server VM, 1.8.0_232
Branch HEAD
Compiled by user yumwang on 2019-12-17T04:38:22Z
Revision bcadd5c3096109878fe26fb0d57a9b7d6fdaa257
Url https://gitbox.apache.org/repos/asf/spark.git
Type --help for more information.
```

Now, open a new PySpark session and execute the following code:

```sh
pyspark
```

Then you can paste the following code:

```python
input_file = sc.textFile("file:///home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/mr/wordcount/src/WordCount.java")
counts = input_file.flatMap(lambda line: line.split(" ")) \
             .map(lambda word: (word, 1)) \
             .reduceByKey(lambda a, b: a + b) \
             .toDF()
counts.show()
exit()
```

It assumes that you have the following local file available /home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/mr/wordcount/src/WordCount.java.

You can also check the Spark job execution at the following URL:

  - Spark Shell Jobs: http://localhost:4040/jobs/

## Stop Spark Master & Slave processes

You can check that your Spark Master & Slave processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

If the command returns any entries, then you can stop processes using the following commands:

```sh
stop-master.sh
stop-slaves.sh
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```
