# Setup additional HDFS data node in your cluster

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **C:\hadoop**.

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop

git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git %HADOOP_HOME%
```

Now checkout the current step branch:

```
cd %HADOOP_HOME%

git fetch --all
git reset --hard origin/step-02
git clean -dfq
```

## Set your Hadoop & Java Home

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_221
```

## Create the NameNode config file as Master

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop-master-nn

copy /Y %HADOOP_HOME%\etc\hadoop\core-site.xml %HADOOP_HOME%\etc\hadoop-master-nn\core-site.xml

notepad %HADOOP_HOME%\etc\hadoop-master-nn\core-site.xml
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
        <value>/${hadoop.home}/tmp/master-nn</value>
    </property>
</configuration>
```

Here, we have set a separate directory to store the name node files.

For more details about the configuration file, you can check the following link:

 - https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml

## Create the DataNodes config files as Slaves

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop-slave-1-dn

copy /Y %HADOOP_HOME%\etc\hadoop\hdfs-site.xml %HADOOP_HOME%\etc\hadoop-slave-1-dn\hdfs-site.xml

notepad %HADOOP_HOME%\etc\hadoop-slave-1-dn\hdfs-site.xml
```

Replace the file content with:

```
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

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop-slave-2-dn
mkdir %HADOOP_HOME%\etc\hadoop-slave-3-dn

copy /Y %HADOOP_HOME%\etc\hadoop-slave-1-dn\hdfs-site.xml %HADOOP_HOME%\etc\hadoop-slave-2-dn\hdfs-site.xml
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1-dn\hdfs-site.xml %HADOOP_HOME%\etc\hadoop-slave-3-dn\hdfs-site.xml

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-2-dn\hdfs-site.xml

powershell -Command "(gc %xml_file%) -replace 'slave-1', 'slave-2' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19866', '29866' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19864', '29864' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19867', '29867' | Out-File -encoding ASCII %xml_file%"

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-3-dn\hdfs-site.xml

powershell -Command "(gc %xml_file%) -replace 'slave-1', 'slave-3' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19866', '39866' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19864', '39864' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '19867', '39867' | Out-File -encoding ASCII %xml_file%"
```

You can now open the generate/modified xml file.

```
notepad %HADOOP_HOME%\etc\hadoop-slave-1-dn\hdfs-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-2-dn\hdfs-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-3-dn\hdfs-site.xml
```

Here you can notice that we have assigned a set of unique port number and folder name for each slave.

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

## Create a client config files

Make a copy of `%HADOOP_HOME%\etc\hadoop\hdfs-site.xml` into `%HADOOP_HOME%\etc\hadoop-client\hdfs-site.xml`.

Execute the following commands:

```
mkdir %HADOOP_HOME%\etc\hadoop-client

copy %HADOOP_HOME%\etc\hadoop\hdfs-site.xml %HADOOP_HOME%\etc\hadoop-client\hdfs-site.xml

notepad %HADOOP_HOME%\etc\hadoop-client\hdfs-site.xml
```

Replace the file content with:

```
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

Execute the following commands:

```
copy /Y %HADOOP_HOME%\etc\hadoop\core-site.xml %HADOOP_HOME%\etc\hadoop-client\core-site.xml

notepad %HADOOP_HOME%\etc\hadoop-client\core-site.xml
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
        <value>/${hadoop.home}/tmp/client</value>
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

Then, execute the following command:

```
hdfs --config %HADOOP_HOME%\etc\hadoop-master-nn namenode -format -force
```


## Start HDFS NameNode

In a command prompt, execute the following commands:

```
start "hdfs - master namenode" hdfs --config %HADOOP_HOME%\etc\hadoop-master-nn namenode
```

## Start HDFS DateNode

In a command prompt, execute the following commands:

```
start "hdfs - slave-1 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-1-dn datanode
start "hdfs - slave-2 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-2-dn datanode
start "hdfs - slave-3 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-3-dn datanode
```
