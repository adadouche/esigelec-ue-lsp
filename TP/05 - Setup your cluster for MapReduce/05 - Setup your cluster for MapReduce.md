# Setup your cluster for MapReduce

## Prerequisites

You will need to have a Hadoop tar ball extracted and properly configured in:

```sh
C:\MyWork\hadoop
```

## Configure the default mapred-site.xml file

Open the following file:

```sh
copy "%HADOOP_HOME%\etc\hadoop\mapred-site.xml.template" "%HADOOP_HOME%\etc\hadoop\mapred-site.xml"
notepad %HADOOP_HOME%\etc\hadoop\mapred-site.xml
```

Paste in the following configuration:

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
</configuration>
```

For more details about the configuration file, you can check the following link:

 - https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml

## Configure the default yarn-site.xml file

Open the following file:

```sh
notepad %HADOOP_HOME%\etc\hadoop\yarn-site.xml
```

Replace the file content with:

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
</configuration>
```

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-common/yarn-default.xml

## Create the Resource Manager config file as master

Execute the following commands:

```
copy "%HADOOP_HOME%\etc\hadoop\yarn-site.xml" "%HADOOP_HOME%\etc\hadoop\master\yarn-site.xml"
notepad "%HADOOP_HOME%\etc\hadoop\master\yarn-site.xml"
```

Replace the file content with:

```
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
</configuration>
```

Now create the capacity scheduler configuration.

Execute the following commands:

```
notepad "%HADOOP_HOME%\etc\hadoop\master\capacity-scheduler.xml
```
Replace the file content with:

```
<?xml version="1.0"?>
<configuration>
	<property>
		<name>yarn.scheduler.capacity.root.queues</name>
		<value>default</value>
	</property>
	<property>
		<name>yarn.scheduler.capacity.root.default.capacity</name>
		<value>100</value>
	</property>
</configuration>
```

## Create the Node Manager config file as slaves

Execute the following commands:

```
copy "%HADOOP_HOME%\etc\hadoop\yarn-site.xml" "%HADOOP_HOME%\etc\hadoop\slave-1\yarn-site.xml"
notepad "%HADOOP_HOME%\etc\hadoop\slave-1\yarn-site.xml"
```

Replace the file content with:

```
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
    <property>
        <name>yarn.nodemanager.localizer.address</name>
        <value>${yarn.nodemanager.hostname}:8140</value>
    </property>
	<property>
        <name>yarn.nodemanager.webapp.address</name>
        <value>${yarn.nodemanager.hostname}:8142</value>
    </property>
</configuration>
```

Execute the following commands:

```
copy "%HADOOP_HOME%\etc\hadoop\mapred-site.xml" "%HADOOP_HOME%\etc\hadoop\slave-1\mapred-site.xml"
notepad "%HADOOP_HOME%\etc\hadoop\slave-1\mapred-site.xml"
```

Replace the file content with:

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.application.classpath</name>
        <value>$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*:$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*</value>
    </property>
    <property>
        <name>mapreduce.shuffle.port</name>
        <value>13562</value>
    </property>
</configuration>
```

Execute the following commands:

```
copy "%HADOOP_HOME%\etc\hadoop\slave-1\yarn-site.xml" "%HADOOP_HOME%\etc\hadoop\slave-2\yarn-site.xml"
notepad "%HADOOP_HOME%\etc\hadoop\slave-2\yarn-site.xml"
```

Increase the yarn.nodemanager.localizer.address & yarn.nodemanager.webapp.address port numbers (8140 & 8142) by a hundred (=> 8240 & 8242) to make them unique.

Execute the following commands:

```
copy "%HADOOP_HOME%\etc\hadoop\slave-1\yarn-site.xml" "%HADOOP_HOME%\etc\hadoop\slave-3\yarn-site.xml"
notepad "%HADOOP_HOME%\etc\hadoop\slave-3\yarn-site.xml"
```

Increase the yarn.nodemanager.localizer.address & yarn.nodemanager.webapp.address port numbers (8140 & 8142) by 2 hundred (=> 8340 & 8342) to make them unique.

Execute the following commands:

```
copy "%HADOOP_HOME%\etc\hadoop\slave-1\mapred-site.xml" "%HADOOP_HOME%\etc\hadoop\slave-2\mapred-site.xml"
notepad "%HADOOP_HOME%\etc\hadoop\slave-2\mapred-site.xml"
```

Increase the mapreduce.shuffle.port port numbers (13562) by a 10 thousand (=> 23562) to make them unique.

Execute the following commands:

```
copy "%HADOOP_HOME%\etc\hadoop\slave-1\mapred-site.xml" "%HADOOP_HOME%\etc\hadoop\slave-3\mapred-site.xml"
notepad "%HADOOP_HOME%\etc\hadoop\slave-3\mapred-site.xml"
```

Increase the mapreduce.shuffle.port port numbers (13562) by a 20 thousand (=> 33562) to make them unique.

## Reinitialize HDFS & Start HDFS daemons

Before starting using the HDFS, you will need to re-format it.

Stop all NameNode & DataNode process.

In a command prompt, execute the following commands to set the environment variables:

```
%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd

cd %HADOOP_HOME%
```

Then, execute the following commands:

```
.\bin\hdfs --config %HADOOP_HOME%\etc\hadoop\master namenode -format -force
```

##

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - namenode" hdfs --config %HADOOP_HOME%\etc\hadoop\master namenode

start "Apache Hadoop Distribution - slave-1" hdfs --config %HADOOP_HOME%\etc\hadoop\slave-1 datanode
start "Apache Hadoop Distribution - slave-2" hdfs --config %HADOOP_HOME%\etc\hadoop\slave-2 datanode
start "Apache Hadoop Distribution - slave-3" hdfs --config %HADOOP_HOME%\etc\hadoop\slave-3 datanode
```

## Start YARN Resource Manager daemon

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - YARN Resource Manager" yarn --config "%HADOOP_HOME%\etc\hadoop\master" resourcemanager

start "Apache Hadoop Distribution - YARN Node Manager 1" yarn --config "%HADOOP_HOME%\etc\hadoop\slave-1" nodemanager
start "Apache Hadoop Distribution - YARN Node Manager 2" yarn --config "%HADOOP_HOME%\etc\hadoop\slave-2" nodemanager
start "Apache Hadoop Distribution - YARN Node Manager 3" yarn --config "%HADOOP_HOME%\etc\hadoop\slave-3" nodemanager
```

You can check the status of your cluster using the following links:

 - Name Node : http://localhost:50070/
 - Resource Manager	: http://localhost:8088/
