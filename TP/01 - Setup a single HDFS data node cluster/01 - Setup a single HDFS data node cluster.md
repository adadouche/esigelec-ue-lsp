# 01 - Setup a single HDFS data node cluster

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
git reset --hard origin/step-00
git clean -dfq
```

## Configure the Hadoop environment

##### Extend the Hadoop environment variables

In your DOS command prompt, execute:

```sh
notepad %HADOOP_HOME%\etc\hadoop\hadoop-env.cmd
```

Locate the following line:

```
@rem Set Hadoop-specific environment variables here.
```

and add the following content right after:

```sh
for %%I in ("%JAVA_HOME%") do (
  set "JAVA_HOME=%%~sI"
)
set "JAVA_HOME=%JAVA_HOME:\=/%"

set "USERNAME=%USERNAME: =_%"

set HADOOP_BIN_PATH=%HADOOP_HOME%\bin
set HADOOP_SBIN_PATH=%HADOOP_HOME%\sbin
set HADOOP_CONF_DIR=%HADOOP_HOME%\etc\hadoop

set PATH=%PATH%;%HADOOP_BIN_PATH%
set PATH=%PATH%;%HADOOP_SBIN_PATH%

set "HADOOP_HOME_OPTS=%HADOOP_HOME:\=/%"

set HADOOP_OPTS=-Dhadoop.home=%HADOOP_HOME_OPTS%
```

##### Configure the core-site.xml file

In your DOS command prompt, execute:

```sh
notepad %HADOOP_HOME%\etc\hadoop\core-site.xml
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

In your DOS command prompt, execute:

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

## Initialize HDFS

Before starting using the HDFS, you will need to format it.

In your DOS command prompt, execute the following commands to set the environment variables:

```
set HADOOP_HOME=C:\hadoop
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_221

cd %HADOOP_HOME%

%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd
```

Then, execute the following commands:

```
hdfs namenode -format -force
```

This will format the name node `dfs`.

## Start HDFS daemons

In a command prompt, execute the following commands:

```
start-dfs
```
