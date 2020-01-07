# Setup additional HDFS data node in your cluster

## Goal

In this tutorial, you will ***simulate*** a multi node Hadoop cluster.

To do so, you will create a series of subfolders that will be used by each ***fake*** node processes as their ***etc/hadoop*** configuration directory.

In the end, you folder structure will look like this:

```
$HADOOP_HOME
|-- etc
    |-- hadoop-master-nn
    |   |-- core-site.xml
    |   |-- log4j.properties    
    |-- hadoop-slave-1-dn
    |   |-- hdfs-site.xml
    |   |-- log4j.properties    
    |-- hadoop-slave-2-dn
    |   |-- hdfs-site.xml
    |   |-- log4j.properties    
    |-- hadoop-slave-3-dn
    |   |-- hdfs-site.xml
    |   |-- log4j.properties    
    |-- hadoop-client
        |-- core-site.xml
        |-- hdfs-site.xml
```

As you can notice, even the client process will hold it's own configuration folder to ***fake*** a remote connection.

Once the directory structure is created, you will format the ***master Name Node*** and then start it in a separate shell window along with the 3 ***slave Data Node*** processes.

The reason, you will use distinct shell windows is to allow you to monitor the logs.

In the *"real life"*, you would browse the logs instead using a *mode* command.

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
git reset --hard origin/step-02
git clean -dfq

./.setup.sh
```

## Stop your current HDFS processes

From the last tutorial, you started a Name Node and a Data Node process which you will need to stop before continuing.

In your **Ubuntu** terminal, execute:

```sh
stop-dfs.sh
```

You can check that the processes are stopped using the **`jps`** command.

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill -9 $(jps -mlV | awk '{ print $1 }')
```

## Create the Master Name Node config file - core-site

In your **Ubuntu** terminal, execute the following commands:

```sh
mkdir -p $HADOOP_HOME/etc/hadoop-master-nn

cp $HADOOP_HOME/etc/hadoop/core-site.xml $HADOOP_HOME/etc/hadoop-master-nn/core-site.xml

nano $HADOOP_HOME/etc/hadoop-master-nn/core-site.xml
```

Replace the file content with:

```sh
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>${hadoop.home}/tmp/hadoop-master-nn</value>
    </property>
</configuration>
```

Here, we have set a separate directory to store the name node temporary files.

For more details about the configuration file, you can check the following link:

 - https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml

## Create the Slaves Data Nodes config files - hdfs-site

In your **Ubuntu** terminal, execute the following commands:

```sh
mkdir -p $HADOOP_HOME/etc/hadoop-slave-1-dn

cp $HADOOP_HOME/etc/hadoop/hdfs-site.xml $HADOOP_HOME/etc/hadoop-slave-1-dn/hdfs-site.xml

nano $HADOOP_HOME/etc/hadoop-slave-1-dn/hdfs-site.xml
```

Replace the file content with:

```xml
<configuration>
  <property>
    <name>dfs.namenode.servicerpc-address</name>
    <value>localhost:9000</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///${hadoop.home}/data/slave-1-dn</value>
  </property>
  <property>
    <name>datanode.https.port</name>
    <value>51475</value>
  </property>
  <property>
    <name>dfs.datanode.address</name>
    <value>0.0.0.0:19866</value>
  </property>
  <property>
    <name>dfs.datanode.http.address</name>
    <value>0.0.0.0:19864</value>
  </property>
  <property>
    <name>dfs.datanode.ipc.address</name>
    <value>0.0.0.0:19867</value>
  </property>   
</configuration>
```

In your **Ubuntu** terminal, execute the following commands:

```sh
mkdir -p $HADOOP_HOME/etc/hadoop-slave-2-dn
mkdir -p $HADOOP_HOME/etc/hadoop-slave-3-dn

cp $HADOOP_HOME/etc/hadoop-slave-1-dn/hdfs-site.xml    $HADOOP_HOME/etc/hadoop-slave-2-dn/hdfs-site.xml
cp $HADOOP_HOME/etc/hadoop-slave-1-dn/hdfs-site.xml    $HADOOP_HOME/etc/hadoop-slave-3-dn/hdfs-site.xml

export xml_file=$HADOOP_HOME/etc/hadoop-slave-2-dn/hdfs-site.xml

sed -i 's/slave-1/slave-2/g' $xml_file
sed -i 's/19866/29866/g' $xml_file
sed -i 's/19864/29864/g' $xml_file
sed -i 's/19867/29867/g' $xml_file

export xml_file=$HADOOP_HOME/etc/hadoop-slave-3-dn/hdfs-site.xml

sed -i 's/slave-1/slave-3/g' $xml_file
sed -i 's/19866/39866/g' $xml_file
sed -i 's/19864/39864/g' $xml_file
sed -i 's/19867/39867/g' $xml_file
```

You can now open the generate/modified xml file.

```sh
more $HADOOP_HOME/etc/hadoop-slave-1-dn/hdfs-site.xml
more $HADOOP_HOME/etc/hadoop-slave-2-dn/hdfs-site.xml
more $HADOOP_HOME/etc/hadoop-slave-3-dn/hdfs-site.xml
```

Here you can notice that we have assigned a set of unique port number and folder name for each slave.

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

## Create a client config files

Make a copy of `$HADOOP_HOME/etc/hadoop/hdfs-site.xml` into `$HADOOP_HOME/etc/hadoop-client/hdfs-site.xml`.

In your **Ubuntu** terminal, execute the following commands:

```sh
mkdir -p $HADOOP_HOME/etc/hadoop-client

cp $HADOOP_HOME/etc/hadoop/hdfs-site.xml    $HADOOP_HOME/etc/hadoop-client/hdfs-site.xml

nano $HADOOP_HOME/etc/hadoop-client/hdfs-site.xml
```

Replace the file content with:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.block.size</name>
    <value>2m</value>
  </property>  
</configuration>
```

In your **Ubuntu** terminal, execute the following commands:

```sh
cp $HADOOP_HOME/etc/hadoop/core-site.xml $HADOOP_HOME/etc/hadoop-client/core-site.xml

nano $HADOOP_HOME/etc/hadoop-client/core-site.xml
```

Replace the file content with:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>${hadoop.home}/tmp/hadoop-client</value>
    </property>
</configuration>
```

This configuration file will be used to upload the file.

## Create the Log4j config files for all processes

In your **Ubuntu** terminal, execute the following commands:

```sh
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-client/log4j.properties
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-master-nn/log4j.properties
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-slave-1-dn/log4j.properties
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-slave-2-dn/log4j.properties
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-slave-3-dn/log4j.properties
```

## Initialize HDFS

> **Make sure your previous NameNode & DataNode processes are shutdown before continuing!!!**

Before starting using your master/slave HDFS configuration, you will need to format it.

In your **Ubuntu** terminal, execute the following commands :

```sh
rm -rf $HADOOP_HOME/tmp/* $HADOOP_HOME/data/* $HADOOP_HOME/logs/* $HADOOP_HOME/pid/*
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode -format -force -clusterID local
```

## Start HDFS Name Node

Open a new **Ubuntu** terminal, execute the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode &
```

## Start HDFS Date Nodes

Open a new **Ubuntu** terminal, execute the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn datanode &
```

Open a new **Ubuntu** terminal, execute the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn datanode &
```

Open a new **Ubuntu** terminal, execute the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn datanode &
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
```

It should return four processes like this:

```
21428 DataNode
21531 DataNode
21659 DataNode
21324 NameNode
```

The first column is the PID (process ID) that you can use if you want to kill a process.

You can also get details about HDFS processes using the following URL:

 - http://localhost:9870/

With the Data Nodes:

 - http://localhost:9870/dfshealth.html#tab-datanode

## Stop HDFS daemons

You can now close the Ubuntu terminals for the Master Name Node & the Slave Data Nodes.

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill -9 $(jps -mlV | awk '{ print $1 }')
```
