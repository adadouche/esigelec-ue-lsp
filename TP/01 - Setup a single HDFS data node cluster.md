# 01 - Setup a single HDFS data node cluster

## Goal

In this tutorial, you will use the "plain" Hadoop 3.2.1 distribution and start it as a single node cluster.

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
>When formatting, you Name Node you can specify the cluster id.

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **esigelec-ue-lsp-hdp** in your Ubuntu home directory (**~**).

Open an **Ubuntu** terminal and execute:

```sh
cd ~
git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git
```

## Download the Hadoop 3.2.1 distribution

In your **Ubuntu** terminal, execute:

```sh
cd ~
wget http://apache.crihan.fr/dist/hadoop/common/hadoop-3.2.1/hadoop-3.2.1.tar.gz
```

## Extract the Hadoop 3.2.1 distribution into the Git Folder

In your **Ubuntu** terminal, execute:

```sh
cd ~
tar xvf hadoop-3.2.1.tar.gz -C ~/esigelec-ue-lsp-hdp --exclude='hadoop-3.2.1/share/doc'
```

The documentation is not extracted as it is available online if needed.

## Configure the your environment with Hadoop environment variables

In your **Ubuntu** terminal, execute:

```sh
export ENV_FILE=~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh
export HADOOP_HOME=$(echo ~)/esigelec-ue-lsp-hdp/hadoop-3.2.1

rm $ENV_FILE

echo -e "umask 022" > $ENV_FILE

echo -e "export LSP_HOME=$(echo ~)/esigelec-ue-lsp-hdp" >> $ENV_FILE

echo -e "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> $ENV_FILE
echo -e "export HADOOP_HOME=\$LSP_HOME/hadoop-3.2.1" >> $ENV_FILE

echo -e "export HADOOP_BIN_PATH=\$HADOOP_HOME/bin" >> $ENV_FILE
echo -e "export HADOOP_SBIN_PATH=\$HADOOP_HOME/sbin" >> $ENV_FILE

echo -e "export HADOOP_CONF_DIR=\$HADOOP_HOME/etc/hadoop" >> $ENV_FILE
echo -e "export HADOOP_LOG_DIR=\$HADOOP_HOME/logs" >> $ENV_FILE

echo -e "export PATH=\$PATH:\$HADOOP_BIN_PATH" >> $ENV_FILE
echo -e "export PATH=\$PATH:\$HADOOP_SBIN_PATH" >> $ENV_FILE
echo -e "export PATH=\$PATH:\$JAVA_HOME/bin" >> $ENV_FILE

echo -e "export \"HADOOP_OPTS=\$HADOOP_OPTS -Dhadoop.home='\$HADOOP_HOME'\""  >> $ENV_FILE

export LINE="export \"HADOOP_OPTS=\$HADOOP_OPTS -Dhadoop.home='\$HADOOP_HOME'\""
grep -qF "$LINE" $HADOOP_HOME/etc/hadoop/hadoop-env.sh || echo -e $LINE >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

export LINE="export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64"
grep -qF "$LINE" $HADOOP_HOME/etc/hadoop/hadoop-env.sh || echo -e $LINE >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc

source $ENV_FILE
```

## Configure the core-site.xml file

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

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

## Initialize HDFS

Before starting using the HDFS, you will need to format it.

In your **Ubuntu** terminal, execute the following command to format the Name Node:

```sh
hdfs namenode -format -force
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
