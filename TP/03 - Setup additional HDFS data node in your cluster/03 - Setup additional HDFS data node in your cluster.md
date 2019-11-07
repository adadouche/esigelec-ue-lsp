# Setup additional HDFS data node in your cluster

## Prerequisites

You will need to have a Hadoop tar ball extracted in:

```sh
C:\MyWork\hadoop
```
## Extend the environment variables

If not done yet, you will update the following file:

```sh
C:\MyWork\hadoop\etc\hadoop\hadoop-env.cmd
```

Locate the following line:

```
@rem Set Hadoop-specific environment variables here.
```

and add the following content right after:

```sh
set HADOOP_HOME=C:\MyWork\hadoop

set HADOOP_BIN_PATH=%HADOOP_HOME%\bin
set HADOOP_CONF_DIR=%HADOOP_HOME%\etc\hadoop

set JAVA_HOME=C:/Progra~1/Java/jdk1.8.0_221
set PATH=%PATH%;%HADOOP_BIN_PATH%

set "HADOOP_HOME_OPTS=%HADOOP_HOME:\=/%"

set HADOOP_OPTS=-Dhadoop.home=%HADOOP_HOME_OPTS%
```

## Configure the default core-site.xml file

Open the following file:

```sh
C:\MyWork\hadoop\etc\hadoop\core-site.xml
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
        <value>/${hadoop.home}/tmp/hadoop-${user.name}</value>
    </property>
</configuration>
```

For more details about the configuration file, you can check the following link:

 - https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml

## Configure the default hdfs-site.xml file

Open the following file:

```sh
notepad %HADOOP_HOME%\etc\hadoop\hdfs-site.xml
```

Paste in the following configuration:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

## Create the NameNode config file as Master

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop\master
copy %HADOOP_HOME%\etc\hadoop\core-site.xml %HADOOP_HOME%\etc\hadoop\master\core-site.xml
notepad %HADOOP_HOME%\etc\hadoop\master\core-site.xml
```

Replace the file content with:

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

Here, we have set a separate directory to store the name node files.

For more details about the configuration file, you can check the following link:

 - https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml

## Create the DataNodes config files as Slaves

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop\slave-1
copy %HADOOP_HOME%\etc\hadoop\hdfs-site.xml %HADOOP_HOME%\etc\hadoop\slave-1\hdfs-site.xml
notepad %HADOOP_HOME%\etc\hadoop\slave-1\hdfs-site.xml
```

Replace the file content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.namenode.servicerpc-address</name>
    <value>localhost:9000</value>
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

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop\slave-2
copy %HADOOP_HOME%\etc\hadoop\hdfs-site.xml %HADOOP_HOME%\etc\hadoop\slave-2\hdfs-site.xml
notepad %HADOOP_HOME%\etc\hadoop\slave-2\hdfs-site.xml
```

Replace the file content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.namenode.servicerpc-address</name>
    <value>localhost:9000</value>
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

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop\slave-3
copy %HADOOP_HOME%\etc\hadoop\hdfs-site.xml %HADOOP_HOME%\etc\hadoop\slave-3\hdfs-site.xml
notepad %HADOOP_HOME%\etc\hadoop\slave-3\hdfs-site.xml
```

Replace the file content with:

```
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
    <name>dfs.namenode.servicerpc-address</name>
    <value>localhost:9000</value>
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

Here, we have assigned a set of unique port number for the instance of the data node to start along with a specific directory to store the data set blocks.

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

## Create a client config files

Make a copy of `C:\MyWork\hadoop\etc\hadoop\hdfs-site.xml` into `C:\MyWork\hadoop\etc\hadoop\hdfs-site.client.xml`.

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop\client-1
copy %HADOOP_HOME%\etc\hadoop\hdfs-site.xml %HADOOP_HOME%\etc\hadoop\client-1\hdfs-site.xml
notepad %HADOOP_HOME%\etc\hadoop\client-1\hdfs-site.xml
```

Replace the file content with:

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

This configuration file will be used to upload the file.

## Initialize HDFS

Before starting using the HDFS, you will need to format it.

In a command prompt, execute the following commands to set the environment variables:

```
C:\MyWork\hadoop\etc\hadoop\hadoop-env.cmd

cd %HADOOP_HOME%
```

Then, execute the following commands:

```
.\bin\hdfs --config %HADOOP_HOME%\etc\hadoop\master namenode -format -force
```

## Start HDFS NameNode

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - namenode" hdfs --config %HADOOP_HOME%\etc\hadoop\master namenode
```

## Start HDFS DateNode

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - slave-1" hdfs --config %HADOOP_HOME%\etc\hadoop\slave-1 datanode
start "Apache Hadoop Distribution - slave-2" hdfs --config %HADOOP_HOME%\etc\hadoop\slave-2 datanode
start "Apache Hadoop Distribution - slave-3" hdfs --config %HADOOP_HOME%\etc\hadoop\slave-3 datanode
```
