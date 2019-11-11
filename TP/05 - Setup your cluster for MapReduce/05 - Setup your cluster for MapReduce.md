# Setup your cluster for MapReduce

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
git reset --hard origin/step-04
git clean -dfq
```

## Set your Hadoop & Java Home

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_221
```

## Extend the YARN environment variables

In your DOS command prompt, execute:

```sh
notepad %HADOOP_HOME%\etc\hadoop\yarn-env.cmd
```

Locate the following line:

```
@rem limitations under the License.
```

and add the following content right after:

```
set HADOOP_YARN_HOME=%HADOOP_HOME%


for %%I in ("%JAVA_HOME%") do (
  set "JAVA_HOME=%%~sI"
)
set "JAVA_HOME=%JAVA_HOME:\=/%"

set HADOOP_BIN_PATH=%HADOOP_HOME%\bin
set HADOOP_SBIN_PATH=%HADOOP_HOME%\sbin
set HADOOP_CONF_DIR=%HADOOP_HOME%\etc\hadoop
set HADOOP_LOG_DIR=%HADOOP_HOME%\logs

set PATH=%PATH%;%HADOOP_BIN_PATH%
set PATH=%PATH%;%HADOOP_SBIN_PATH%

set "HADOOP_HOME_OPTS=%HADOOP_HOME:\=/%"

set YARN_OPTS=%YARN_OPTS% -Dyarn.home=%HADOOP_HOME_OPTS%
set YARN_OPTS=%YARN_OPTS% -Dhadoop.home=%HADOOP_HOME_OPTS%
```

## Create the Resource Manager config file as master

Execute the following commands:

```
mkdir "%HADOOP_HOME%\etc\hadoop-master-rm"

copy /Y "%HADOOP_HOME%\etc\hadoop\yarn-site.xml" "%HADOOP_HOME%\etc\hadoop-master-rm\yarn-site.xml"

notepad "%HADOOP_HOME%\etc\hadoop-master-rm\yarn-site.xml"
```

Replace the file content with:

```
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
notepad "%HADOOP_HOME%\etc\hadoop-master-rm\capacity-scheduler.xml
```

Replace the file content with:

```
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
mkdir "%HADOOP_HOME%\etc\hadoop-slave-1-nm"

copy "%HADOOP_HOME%\etc\hadoop\yarn-site.xml" "%HADOOP_HOME%\etc\hadoop-slave-1-nm\yarn-site.xml"

notepad "%HADOOP_HOME%\etc\hadoop-slave-1-nm\yarn-site.xml"
```

Replace the file content with:

```
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
mkdir "%HADOOP_HOME%\etc\hadoop-slave-2-nm"
mkdir "%HADOOP_HOME%\etc\hadoop-slave-3-nm"

copy /Y %HADOOP_HOME%\etc\hadoop-slave-1-nm\yarn-site.xml %HADOOP_HOME%\etc\hadoop-slave-2-nm\yarn-site.xml
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1-nm\yarn-site.xml %HADOOP_HOME%\etc\hadoop-slave-3-nm\yarn-site.xml

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-2-nm\yarn-site.xml

powershell -Command "(gc %xml_file%) -replace '8140', '8240' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '8142', '8242' | Out-File -encoding ASCII %xml_file%"

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-3-nm\yarn-site.xml

powershell -Command "(gc %xml_file%) -replace '8140', '8340' | Out-File -encoding ASCII %xml_file%"
powershell -Command "(gc %xml_file%) -replace '8142', '8342' | Out-File -encoding ASCII %xml_file%"
```

You can now open the generate/modified xml file.

```
notepad %HADOOP_HOME%\etc\hadoop-slave-1-nm\yarn-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-2-nm\yarn-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-3-nm\yarn-site.xml
```

Execute the following commands:

```
copy /Y "%HADOOP_HOME%\etc\hadoop\mapred-site.xml.template" "%HADOOP_HOME%\etc\hadoop-slave-1-nm\mapred-site.xml"

notepad "%HADOOP_HOME%\etc\hadoop-slave-1-nm\mapred-site.xml"
```

Replace the file content with:

```
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
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1-nm\mapred-site.xml %HADOOP_HOME%\etc\hadoop-slave-2-nm\mapred-site.xml
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1-nm\mapred-site.xml %HADOOP_HOME%\etc\hadoop-slave-3-nm\mapred-site.xml

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-2-nm\mapred-site.xml

powershell -Command "(gc %xml_file%) -replace '13562', '23562' | Out-File -encoding ASCII %xml_file%"

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-3-nm\mapred-site.xml

powershell -Command "(gc %xml_file%) -replace '13562', '33562' | Out-File -encoding ASCII %xml_file%"
```

You can now open the generate/modified xml file.

```
notepad %HADOOP_HOME%\etc\hadoop-slave-1-nm\mapred-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-2-nm\mapred-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-3-nm\mapred-site.xml
```

Execute the following commands:

```
copy /Y "%HADOOP_HOME%\etc\hadoop\core-site.xml" "%HADOOP_HOME%\etc\hadoop-slave-1-nm\core-site.xml"

notepad "%HADOOP_HOME%\etc\hadoop-slave-1-nm\core-site.xml"
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
        <value>/${hadoop.home}/tmp/slave-1-nm</value>
    </property>
</configuration>
```

Execute the following commands:

```
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1-nm\core-site.xml %HADOOP_HOME%\etc\hadoop-slave-2-nm\core-site.xml
copy /Y %HADOOP_HOME%\etc\hadoop-slave-1-nm\core-site.xml %HADOOP_HOME%\etc\hadoop-slave-3-nm\core-site.xml

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-2-nm\core-site.xml

powershell -Command "(gc %xml_file%) -replace 'slave-1', 'slave-2' | Out-File -encoding ASCII %xml_file%"

set xml_file=%HADOOP_HOME%\etc\hadoop-slave-3-nm\core-site.xml

powershell -Command "(gc %xml_file%) -replace 'slave-1', 'slave-3' | Out-File -encoding ASCII %xml_file%"
```

You can now open the generate/modified xml file.

```
notepad %HADOOP_HOME%\etc\hadoop-slave-1-nm\core-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-2-nm\core-site.xml
notepad %HADOOP_HOME%\etc\hadoop-slave-3-nm\core-site.xml
```


## Start HDFS NameNode & DataNode process

If the HDFS processes are not started yet, you will need to start.

In your DOS command prompt, execute the following commands to set the environment variables:

```
%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd
```

Then, execute the following command:

```sh
rd /S /Q %HADOOP_HOME%\tmp
rd /S /Q %HADOOP_HOME%\data

hdfs --config %HADOOP_HOME%\etc\hadoop-master-nn namenode -format -force

start "hdfs - master namenode" hdfs --config %HADOOP_HOME%\etc\hadoop-master-nn namenode

start "hdfs - slave-1 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-1-dn datanode
start "hdfs - slave-2 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-2-dn datanode
start "hdfs - slave-3 datanode" hdfs --config %HADOOP_HOME%\etc\hadoop-slave-3-dn datanode
```

## Start YARN Resource Manager daemon

In your DOS command prompt, execute the following commands to set the environment variables:

```
%HADOOP_HOME%\etc\hadoop\yarn-env.cmd
```

Then, execute the following command:

```
start "yarn - master resourcemanager" yarn --config "%HADOOP_HOME%\etc\hadoop-master-rm" resourcemanager

start "yarn - slave-1 nodemanager" yarn --config "%HADOOP_HOME%\etc\hadoop-slave-1-nm" nodemanager
start "yarn - slave-2 nodemanager" yarn --config "%HADOOP_HOME%\etc\hadoop-slave-2-nm" nodemanager
start "yarn - slave-3 nodemanager" yarn --config "%HADOOP_HOME%\etc\hadoop-slave-3-nm" nodemanager
```

You can check the status of your cluster using the following links:

 - Name Node : http://localhost:50070/
 - Resource Manager	: http://localhost:8088/
