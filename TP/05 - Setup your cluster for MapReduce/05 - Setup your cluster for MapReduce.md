# Setup your cluster for MapReduce

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **C:\hadoop**.

Open a DOS command prompt and execute:

```sh
git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git C:\hadoop

set HADOOP_HOME=C:\hadoop

cd %HADOOP_HOME%
git checkout --track step-04
```

## Set your Hadoop Home

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop
```

## Create the Resource Manager config file as master

Execute the following commands:

```
copy /Y "%HADOOP_HOME%\etc\hadoop\yarn-site.xml" "%HADOOP_HOME%\etc\hadoop-master\yarn-site.xml"

notepad "%HADOOP_HOME%\etc\hadoop-master\yarn-site.xml"
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

Now, create the capacity scheduler configuration.

Execute the following commands:

```
notepad "%HADOOP_HOME%\etc\hadoop-master\capacity-scheduler.xml
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
copy "%HADOOP_HOME%\etc\hadoop\yarn-site.xml" "%HADOOP_HOME%\etc\hadoop-slave-1\yarn-site.xml"

notepad "%HADOOP_HOME%\etc\hadoop-slave-1\yarn-site.xml"
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
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1\yarn-site.xml %HADOOP_HOME%\etc\hadoop-slave-2\yarn-site.xml
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1\yarn-site.xml %HADOOP_HOME%\etc\hadoop-slave-3\yarn-site.xml

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-2\yarn-site.xml

powershell -Command "(gc %xml_file%) -replace '8140', '8240' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '8142', '8242' | Out-File -encoding ASCII %xml_file%"

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-3\yarn-site.xml

powershell -Command "(gc %xml_file%) -replace '8140', '8340' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '8142', '8342' | Out-File -encoding ASCII %xml_file%"
```

You can now open the generate/modified xml file.

```
notepad %HADOOP_HOME%\etc\hadoop-slave-1\yarn-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-2\yarn-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-3\yarn-site.xml
```

```
copy /Y "%HADOOP_HOME%\etc\hadoop\mapred-site.xml.template" "%HADOOP_HOME%\etc\hadoop-slave-1\mapred-site.xml"

notepad "%HADOOP_HOME%\etc\hadoop-slave-1\mapred-site.xml"
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
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1\mapred-site.xml %HADOOP_HOME%\etc\hadoop-slave-2\mapred-site.xml
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1\mapred-site.xml %HADOOP_HOME%\etc\hadoop-slave-3\mapred-site.xml

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-2\mapred-site.xml

powershell -Command "(gc %xml_file%) -replace '13562', '23562' | Out-File -encoding ASCII %xml_file%"

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-3\mapred-site.xml

powershell -Command "(gc %xml_file%) -replace '13562', '33562' | Out-File -encoding ASCII %xml_file%"
```

You can now open the generate/modified xml file.

```
notepad %HADOOP_HOME%\etc\hadoop-slave-1\mapred-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-2\mapred-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-3\mapred-site.xml
```

Execute the following commands:

```
copy /Y "%HADOOP_HOME%\etc\hadoop\core-site.xml" "%HADOOP_HOME%\etc\hadoop-slave-1\core-site.xml"

notepad "%HADOOP_HOME%\etc\hadoop-slave-1\core-site.xml"
```

Replace the file content with:

```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/${hadoop.home}/tmp/slave-1</value>
    </property>
</configuration>
```

Execute the following commands:

```
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1\core-site.xml %HADOOP_HOME%\etc\hadoop-slave-2\core-site.xml
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1\core-site.xml %HADOOP_HOME%\etc\hadoop-slave-3\core-site.xml

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-2\core-site.xml

powershell -Command "(gc %xml_file%) -replace 'slave-1', 'slave-2' | Out-File -encoding ASCII %xml_file%"

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-3\core-site.xml

powershell -Command "(gc %xml_file%) -replace 'slave-1', 'slave-3' | Out-File -encoding ASCII %xml_file%"
```

You can now open the generate/modified xml file.

```
notepad %HADOOP_HOME%\etc\hadoop-slave-1\core-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-2\core-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-3\core-site.xml
```


## Start HDFS NameNode & DataNode process

If the HDFS processes are not started yet, you will need to start.

Open a DOS command prompt and execute:

```sh

%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd

start "Apache Hadoop Distribution - namenode" hdfs --config %HADOOP_HOME%\etc\hadoop-master namenode

start "Apache Hadoop Distribution - slave-1" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-1 datanode
start "Apache Hadoop Distribution - slave-2" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-2 datanode
start "Apache Hadoop Distribution - slave-3" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-3 datanode
```

## Start YARN Resource Manager daemon

In a command prompt, execute the following commands:

```
%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd

start "Apache Hadoop Distribution - YARN Resource Manager" yarn --config "%HADOOP_HOME%\etc\hadoop-master" resourcemanager

start "Apache Hadoop Distribution - YARN Node Manager 1" yarn --config "%HADOOP_HOME%\etc\hadoop-slave-1" nodemanager
start "Apache Hadoop Distribution - YARN Node Manager 2" yarn --config "%HADOOP_HOME%\etc\hadoop-slave-2" nodemanager
start "Apache Hadoop Distribution - YARN Node Manager 3" yarn --config "%HADOOP_HOME%\etc\hadoop-slave-3" nodemanager
```

You can check the status of your cluster using the following links:

 - Name Node : http://localhost:50070/
 - Resource Manager	: http://localhost:8088/
