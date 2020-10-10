# 01 - Setup a single HDFS data node cluster

## Goal

In this tutorial, you will use the "plain" Hadoop 3.3.0 distribution and start it as a single node cluster.

The goal here is to identify the set of environment variables required to properly start your Hadoop instance:
 - JAVA_HOME
 - HADOOP_HOME
 - HADOOP_CONF_DIR
 - HADOOP_LOG_DIR
 - HADOOP_OPTS
 - PATH

If these environment variable are not configured, then default values will be applied (which works too), but the location of the generated files will be diverse on your system.

A series of files will also be configured to specify ports and file locations as well:
 - core-site: defines the Name Node port and the tmp directory

Before starting your single node cluster, you'll need to format the Name Node.

**<p style="text-align: center;"> <span style="color: red">In order to keep your environment similar to the one used to build the series of tutorials, you will need to do everything in the same directory as described in the tutorials (esigelec-ue-lsp-hdp).
PLEASE BE DON'T BE CREATIVE WITH FOLDER NAMES!!!!</span></p>**

> #### **Note:**
>Keep in mind that if you format your Name Node with existing data in your Data Node, you will receive an error because the cluster ids won't match.
>When formatting your Name Node you can specify the cluster id.

## Prerequisites - Not required when using Azure Labs

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

. ./scripts/git-restore.sh step-00
```

## Download the Hadoop 3.3.0 distribution - Not required when using Azure Labs

In your **Ubuntu** terminal, execute:

```sh
cd ~
wget -nc http://apache.crihan.fr/dist/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz
```

## Extract the Hadoop 3.3.0 distribution into the Git Folder

In your **Ubuntu** terminal, execute:

```sh
cd ~
tar xvf hadoop-3.3.0.tar.gz -C ~/esigelec-ue-lsp-hdp --exclude='hadoop-3.3.0/share/doc'
```

The documentation is not extracted as it is available online if needed.

## Configure your BASH environment with Hadoop environment variables

In your **Ubuntu** terminal, execute:

```sh
export ENV_FILE=~/esigelec-ue-lsp-hdp/env/set-hadoop-env.sh
echo \#\!/bin/bash > $ENV_FILE
echo "
export LSP_HOME=/home/\$USER/esigelec-ue-lsp-hdp

export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=\$LSP_HOME/hadoop-3.3.0

export HADOOP_BIN_PATH=\$HADOOP_HOME/bin
export HADOOP_SBIN_PATH=\$HADOOP_HOME/sbin

export HADOOP_CONF_DIR=\$HADOOP_HOME/etc/hadoop
export HADOOP_LOG_DIR=\$HADOOP_HOME/logs

export PATH=\$PATH:\$HADOOP_BIN_PATH
export PATH=\$PATH:\$HADOOP_SBIN_PATH
export PATH=\$PATH:\$JAVA_HOME/bin

export \"HADOOP_OPTS=\$HADOOP_OPTS -Dhadoop.home='\$HADOOP_HOME'\"
export set_hadoop_env=done
" >> $ENV_FILE

dos2unix $ENV_FILE
chmod ugo+x $ENV_FILE

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc

source $ENV_FILE
```

## Configure your Hadoop environment file

In your **Ubuntu** terminal, execute:

```sh
export LINE="export \"HADOOP_OPTS=\$HADOOP_OPTS -Dhadoop.home='\$HADOOP_HOME'\""
grep -qF "$LINE" $HADOOP_HOME/etc/hadoop/hadoop-env.sh || echo -e $LINE >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

export LINE="export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64"
grep -qF "$LINE" $HADOOP_HOME/etc/hadoop/hadoop-env.sh || echo -e $LINE >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

## Configure the Hadoop core-site.xml file

In your **Ubuntu** terminal, execute:

```sh
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

Paste in the following configuration:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>${hadoop.home}/tmp/hadoop</value>
    </property>
</configuration>
```

For more details about the configuration file, you can check the following link:

 - https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml

## Initialize HDFS

Before starting using the HDFS, you will need to format it.

In your **Ubuntu** terminal, execute the following command to format the Name Node:

```sh
rm -rf $HADOOP_HOME/tmp/* $HADOOP_HOME/data/* $HADOOP_HOME/logs/* $HADOOP_HOME/pid/*

hdfs namenode -format -force -clusterID local
```

This will format the name node `dfs`.

## Start HDFS daemons

In a command prompt, execute the following commands:

```sh
start-dfs.sh
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

You can check the processes are started by using the following command:

```sh
jps
```

It should return four processes like this:

```
16130 SecondaryNameNode
16274 Jps
15718 NameNode
15900 DataNode
```

The first column is the PID (process ID) that you can use if you want to kill a process.

You can also get details about HDFS and the Name Node using the following URL:

 - http://localhost:9870/

## Stop HDFS daemons

In a command prompt, execute the following commands:

```sh
stop-dfs.sh
```

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill -9 $(jps -mlV | awk '{ print $1 }')
```
