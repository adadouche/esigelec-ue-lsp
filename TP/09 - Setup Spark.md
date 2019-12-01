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
rm $ENV_FILE

echo -e "umask 022" > $ENV_FILE

echo -e "export LSP_HOME=$(echo ~)/esigelec-ue-lsp-hdp" >> $ENV_FILE

echo -e "export SPARK_HOME=\$LSP_HOME/spark-3.0.0" >> $ENV_FILE
echo -e "export HADOOP_HOME=\$LSP_HOME/hadoop-3.2.1" >> $ENV_FILE

echo -e "export PATH=\$PATH:\$SPARK_HOME/bin" >> $ENV_FILE
echo -e "export PATH=\$PATH:\$SPARK_HOME/sbin" >> $ENV_FILE

export LINE="export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop-client"
grep -qF "$LINE" $SPARK_HOME/conf/spark-env.sh || echo -e $LINE >> $SPARK_HOME/conf/spark-env.sh

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc

source $ENV_FILE
```

## Start Spark Master & Slave processes

You can check that your HDFS & Yarn processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

If the command returns the following, then you don't need to start the HDFS processes again:
 - 1 Name Node
 - 3 Data Node
 - 1 Resource Manager
 - 3 Node Manager

If you need to start the HDFS & Yarn processes, execute the following commands:

```sh
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

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon start resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon start nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm --daemon start nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-3-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm --daemon start nodemanager
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

You can also get details about HDFS processes using the following URL:

 - Spark Web UI : http://localhost:8080/
