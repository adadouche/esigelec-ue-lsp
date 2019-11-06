# Setup additional HDFS data node in your cluster

## Prerequisites

You will need to have a Hadoop tar ball extracted in:

```sh
C:\MyWork\hadoop
```

## Create the NameNode config file

Make a copy of the following file:

```
C:\MyWork\hadoop\etc\hadoop\core-site.xml
```

and name it:

```
C:\MyWork\hadoop\etc\hadoop\core-site.namenode.xml
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
C:\MyWork\hadoop\etc\hadoop\hdfs-site.xml
```

and name it:

```
C:\MyWork\hadoop\etc\hadoop\hdfs-site.datanode-1.xml
```

Open the hdfs-site.datanode-1.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
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
C:\MyWork\hadoop\etc\hadoop\hdfs-site.xml
```

and name it:

```
C:\MyWork\hadoop\etc\hadoop\hdfs-site.datanode-2.xml
```

Open the hdfs-site.datanode-2.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
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
C:\MyWork\hadoop\etc\hadoop\hdfs-site.xml
```

and name it:

```
C:\MyWork\hadoop\etc\hadoop\hdfs-site.datanode-3.xml
```

Open the hdfs-site.datanode-3.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
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
C:\MyWork\hadoop\etc\hadoop\hdfs-site.xml
```

and name it:

```
C:\MyWork\hadoop\etc\hadoop\hdfs-site.client.xml
```

Open the hdfs-site.client.xml file and replace the content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
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

## Initialize HDFS

Before starting using the HDFS, you will need to format it.

In a command prompt, execute the following commands to set the environment variables:

```
cd C:\MyWork\hadoop

.\etc\hadoop\hadoop-env.cmd
```

Then, execute the following commands:

```
.\bin\hdfs namenode -conf .\etc\hadoop\core-site.namenode.xml -format -force
```

## Start HDFS NameNode

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - namenode" hdfs namenode -conf .\etc\hadoop\core-site.namenode.xml
```

## Start HDFS DateNode

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - datanode_1" hdfs datanode -conf .\etc\hadoop\hdfs-site.datanode-1.xml
start "Apache Hadoop Distribution - datanode_2" hdfs datanode -conf .\etc\hadoop\hdfs-site.datanode-2.xml  
start "Apache Hadoop Distribution - datanode_3" hdfs datanode -conf .\etc\hadoop\hdfs-site.datanode-3.xml
```
