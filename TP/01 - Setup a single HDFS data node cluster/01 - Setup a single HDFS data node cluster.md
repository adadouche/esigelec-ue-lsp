# 01 - Setup a single HDFS data node cluster

## Prerequisites

You will need to have a Hadoop tar ball extracted in:

```sh
C:\MyWork\hadoop
```

## Configure the Hadoop environment

##### Extend the environment variables

Open the following file:

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

##### Configure the core-site.xml file

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

##### Configure the hdfs-site.xml file

Open the following file:

```sh
C:\MyWork\hadoop\etc\hadoop\hdfs-site.xml
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

## Initialize HDFS

Before starting using the HDFS, you will need to format it.

In a command prompt, execute the following commands to set the environment variables:

```
C:\MyWork\hadoop\etc\hadoop\hadoop-env.cmd
```

Then, execute the following commands:

```
cd %HADOOP_HOME%

.\bin\hdfs namenode -format
```

This will format the name node `dfs`.

## Start HDFS daemons

In a command prompt, execute the following commands:

```
.\sbin\start-dfs.cmd
```
