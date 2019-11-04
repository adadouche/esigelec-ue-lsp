# Setup additional DataNode

## Prerequisites

You will need to have a running single node Hadoop cluster.

## Create the NameNode config file

Make a copy of the following file:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\core-site.xml
```

and name it:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\core-site.namenode.xml
```

Open the core-site.namenode.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/${hadoop.home}/tmp/namenode</value>
    </property>
</configuration>
```

For more details about the configuration file, you can check the following link:

 - https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml

## Create the DataNode-1 config files

Make a copy of the following file:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hdfs-site.xml
```

and name it:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hdfs-site.datanode-1.xml
```

Open the hdfs-site.datanode-1.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>4</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///${hadoop.home}/tmp/datanode-1/dfs/data</value>
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

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

## Create the DataNode-2 config files

Make a copy of the following file:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hdfs-site.xml
```

and name it:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hdfs-site.datanode-2.xml
```

Open the hdfs-site.datanode-2.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>4</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///${hadoop.home}/tmp/datanode-2/dfs/data</value>
  </property>
  <property>
    <name>datanode.https.port</name>
    <value>52475</value>
  </property>
  <property>
    <name>dfs.datanode.address</name>
    <value>0.0.0.0:29866</value>
  </property>
  <property>
    <name>dfs.datanode.http.address</name>
    <value>0.0.0.0:29864</value>
  </property>
  <property>
    <name>dfs.datanode.ipc.address</name>
    <value>0.0.0.0:29867</value>
  </property>   
</configuration>
```

## Create the DataNode-3 config files

Make a copy of the following file:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hdfs-site.xml
```

and name it:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hdfs-site.datanode-3.xml
```

Open the hdfs-site.datanode-3.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>4</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///${hadoop.home}/tmp/datanode-3/dfs/data</value>
  </property>
  <property>
    <name>datanode.https.port</name>
    <value>53475</value>
  </property>
  <property>
    <name>dfs.datanode.address</name>
    <value>0.0.0.0:39866</value>
  </property>
  <property>
    <name>dfs.datanode.http.address</name>
    <value>0.0.0.0:39864</value>
  </property>
  <property>
    <name>dfs.datanode.ipc.address</name>
    <value>0.0.0.0:39867</value>
  </property>   
</configuration>
```

## Create the client config files

Make a copy of the following file:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hdfs-site.xml
```

and name it:

```
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hdfs-site.client.xml
```

Open the hdfs-site.datanode-3.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.namenode.replication.min</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.block.size</name>
    <value>2m</value>
  </property>  
</configuration>
```

## Initialize HDFS

Before starting using the HDFS, you will need to format it.

In a command prompt, execute the following commands to set the environment variables:

```
cd C:\MyWork\hadoop-3.0.0-SNAPSHOT

.\etc\hadoop\hadoop-env.cmd
```

Then, execute the following commands:

```
.\bin\hdfs namenode -conf .\etc\hadoop\core-site.namenode.xml -format -force
```

## Start HDFS NameNode

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - namenode  " hdfs namenode -conf .\etc\hadoop\core-site.namenode.xml
```

## Start HDFS DateNode

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - datanode_1" hdfs datanode -conf .\etc\hadoop\hdfs-site.datanode-1.xml
start "Apache Hadoop Distribution - datanode_2" hdfs datanode -conf .\etc\hadoop\hdfs-site.datanode-2.xml  
start "Apache Hadoop Distribution - datanode_3" hdfs datanode -conf .\etc\hadoop\hdfs-site.datanode-3.xml
```


## Interact with the File System

Download a local copy of the following link:

- http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

Unzip the file into your HADOOP HOME directory.

Now, let's import:

```sh
.\bin\hdfs dfs -put "%HADOOP_HOME%\1500000 Sales Records.csv" "/1500000 Sales Records.1.csv"
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

You can also explore the "data" stored in the windows explorer at: C:\MyWork\hadoop-3.0.0-SNAPSHOT\tmp
