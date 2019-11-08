# Setup additional HDFS data node in your cluster

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **C:\hadoop**.

Open a DOS command prompt and execute:

```sh
git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git C:\hadoop

set HADOOP_HOME=C:\hadoop

cd %HADOOP_HOME%
git checkout --track step-02
```

## Set your Hadoop Home

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop
```

## Create the NameNode config file as Master

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop-master

copy /Y %HADOOP_HOME%\etc\hadoop\core-site.xml %HADOOP_HOME%\etc\hadoop-master\core-site.xml

notepad %HADOOP_HOME%\etc\hadoop-master\core-site.xml
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
mkdir %HADOOP_HOME%\etc\hadoop-slave-1

copy /Y %HADOOP_HOME%\etc\hadoop\hdfs-site.xml %HADOOP_HOME%\etc\hadoop-slave-1\hdfs-site.xml

notepad %HADOOP_HOME%\etc\hadoop-slave-1\hdfs-site.xml
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
    <value>file:///${hadoop.home}/data/slave-1</value>
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
mkdir %HADOOP_HOME%\etc\hadoop-slave-2
mkdir %HADOOP_HOME%\etc\hadoop-slave-3

copy /Y %HADOOP_HOME%\etc\hadoop-slave-1\hdfs-site.xml %HADOOP_HOME%\etc\hadoop-slave-2\hdfs-site.xml
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1\hdfs-site.xml %HADOOP_HOME%\etc\hadoop-slave-3\hdfs-site.xml

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-2\hdfs-site.xml

powershell -Command "(gc %xml_file%) -replace 'slave-1', 'slave-2' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19866', '29866' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19864', '29864' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19867', '39867' | Out-File -encoding ASCII %xml_file%"

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-3\hdfs-site.xml

powershell -Command "(gc %xml_file%) -replace 'slave-1', 'slave-3' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19866', '39866' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19864', '39864' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19867', '39867' | Out-File -encoding ASCII %xml_file%"
```

You can now open the generate/modified xml file.

```
notepad %HADOOP_HOME%\etc\hadoop-slave-1\hdfs-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-2\hdfs-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-3\hdfs-site.xml
```

Here you can notice that we have assigned a set of unique port number and folder name for each slave.

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

## Create a client config files

Make a copy of `%HADOOP_HOME%\etc\hadoop\hdfs-site.xml` into `%HADOOP_HOME%\etc\hadoop\hdfs-site.client.xml`.

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop-client

copy %HADOOP_HOME%\etc\hadoop\hdfs-site.xml %HADOOP_HOME%\etc\hadoop-client\hdfs-site.xml

notepad %HADOOP_HOME%\etc\hadoop-client\hdfs-site.xml
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

> **Make sure your previous NameNode & DataNode processes are shutdown before continuing!!!**

Before starting using your master/slave HDFS configuration, you will need to format it.

In your DOS command prompt, execute the following commands to set the environment variables:

```
%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd
```

Then, execute the following commands:

```
hdfs --config %HADOOP_HOME%\etc\hadoop-master namenode -format -force
```


## Start HDFS NameNode

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - namenode" hdfs --config %HADOOP_HOME%\etc\hadoop-master namenode
```

## Start HDFS DateNode

In a command prompt, execute the following commands:

```
start "Apache Hadoop Distribution - slave-1" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-1 datanode
start "Apache Hadoop Distribution - slave-2" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-2 datanode
start "Apache Hadoop Distribution - slave-3" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-3 datanode
```
