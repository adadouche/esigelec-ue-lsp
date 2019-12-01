# Setup Hive

## Goal

In this tutorial, using your ***simulated*** a multi node Hadoop cluster, you will setup Hive to consume the underlying HDFS data.

The main steps are:

  - Download and un-compress the Hive Tar ball
  - Start your HDFS/YARN/MapReduce processes
  - Create the Hive Meta store and Temp directory in HDFS
  - Configure the hive-site file
  - Start the Hive process

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **esigelec-ue-lsp-hdp** in your Ubuntu home directory (**~**).

If you didn't clone the repository yet, you can do so using the following command:

```sh
cd ~
git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git
```

Now checkout the current step branch:

```sh
cd ~/esigelec-ue-lsp-hdp

git reset --hard origin/new-step-06
git clean -dfq
```

## Download the Hive 3.1.2 distribution

In your **Ubuntu** terminal, execute:

```sh
cd ~
wget https://apache.mirrors.benatherton.com/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
```

## Extract the Hive 3.1.2 distribution into the Git Folder

In your **Ubuntu** terminal, execute:

```sh
cd ~
tar xvf apache-hive-3.1.2-bin.tar.gz -C ~/esigelec-ue-lsp-hdp

mv ~/esigelec-ue-lsp-hdp/apache-hive-3.1.2-bin ~/esigelec-ue-lsp-hdp/hive-3.1.2
```

## Configure the your environment with Hive environment variables

In your **Ubuntu** terminal, execute:

```sh
export ENV_FILE=~/esigelec-ue-lsp-hdp/.set_hive_env.sh
export HIVE_HOME=$(echo ~)/esigelec-ue-lsp-hdp/hive-3.1.2

rm -f $ENV_FILE

echo -e "umask 022" > $ENV_FILE

echo -e "export LSP_HOME=$(echo ~)/esigelec-ue-lsp-hdp" >> $ENV_FILE

echo -e "export HIVE_HOME=\$LSP_HOME/hive-3.1.2" >> $ENV_FILE
echo -e "export HIVE_CONF_DIR=\$HIVE_HOME/conf" >> $ENV_FILE

echo -e "export PATH=\$PATH:\$HIVE_HOME/bin" >> $ENV_FILE

echo -e "export \"HADOOP_OPTS=\$HADOOP_OPTS -Dhive.home='\$HIVE_HOME' \"" >> $ENV_FILE

cp $HIVE_HOME/conf/hive-env.sh.template $HIVE_HOME/conf/hive-env.sh

export LINE="export HADOOP_HOME=$LSP_HOME/hadoop-3.2.1"
grep -qF "$LINE" $HIVE_HOME/conf/hive-env.sh || echo -e $LINE >> $HIVE_HOME/conf/hive-env.sh

export LINE="export HIVE_CONF_DIR=\$HIVE_HOME/conf"
grep -qF "$LINE" $HIVE_HOME/conf/hive-env.sh || echo -e $LINE >> $HIVE_HOME/conf/hive-env.sh

export LINE="export HADOOP_CONF_DIR=\$HADOOP_HOME/etc/hadoop-client"
grep -qF "$LINE" $HIVE_HOME/conf/hive-env.sh || echo -e $LINE >> $HIVE_HOME/conf/hive-env.sh

export LINE="export \"HADOOP_OPTS=\$HADOOP_OPTS -Dhive.home='\$HIVE_HOME' \""
grep -qF "$LINE" $HIVE_HOME/conf/hive-env.sh || echo -e $LINE >> $HIVE_HOME/conf/hive-env.sh

export LINE="export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64"
grep -qF "$LINE" $HIVE_HOME/conf/hive-env.sh || echo -e $LINE >> $HIVE_HOME/conf/hive-env.sh

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc

source $ENV_FILE
```

## Copy GUAVA version from Hadoop to the Hive 3.1.2 distribution

The GUAVA version used across Hadoop and Hive are different which is causing issue, this is why you need to copy over the one from Hadoop to Hive.

In your **Ubuntu** terminal, execute:

```sh
mv $HIVE_HOME/lib/guava-19.0.jar $HIVE_HOME/lib/guava-19.0.jar.old
cp $HADOOP_HOME/share/hadoop/common/lib/guava-27.0-jre.jar $HIVE_HOME/lib/guava-27.0-jre.jar
```

## Start HDFS & Yarn processes

You can check that your HDFS & Yarn processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

If the command returns the following, then you don't need to start the HDFS & Yarn processes again:
 - 1 Name Node
 - 3 Data Node
 - 1 Resource Manager
 - 3 Node Manager

If you need to start the HDFS & Yarn processes, execute the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn  
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-nn  
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon start namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon start datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-2-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn --daemon start datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-3-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon start datanode

sleep 30

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon start resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon start nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm --daemon start nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-3-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm --daemon start nodemanager
```

You can check that your HDFS & Yarn processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

You can also get details about HDFS processes using the following URL:

 - Name Node : http://localhost:9870/
 - Resource Manager	: http://localhost:8088/

## Create Hive Meta store and temporary directory

In your Ubuntu terminal, execute the following commands:

```sh
hadoop fs -mkdir -p /tmp
hadoop fs -mkdir -p /user/hive/warehouse
hadoop fs -chmod g+w /tmp
hadoop fs -chmod g+w /user/hive/warehouse
```

## Configure Hive - hive-site

In your **Ubuntu** terminal, execute the following commands:

```sh
nano "$HIVE_HOME/conf/hive-site.xml"
```

Replace the file content with:

```xml
<configuration>
	<property>
		<name>hive.server2.enable.doAs</name>
		<value>false</value>
	</property>
	<property>
		<name>hive.exec.mode.local.auto</name>
		<value>false</value>
	</property>
	<property>
		<name>javax.jdo.option.ConnectionURL</name>  
		<value>jdbc:derby:;databaseName=${hive.home}/metastore_db;create=true</value>
	</property>
</configuration>
```

## Configure Hive - Initialize the Hive Meta store

You can cleanup previous experiments using the following commands:

```sh
rm -rf $HIVE_HOME/metastore_db
```

The Schema Tool is a offline command line tool to manage the Hive Meta Store.

Then, make sure to start the next commands in the $HIVE_HOME directory as described below.

In your Ubuntu terminal, execute the following commands:

```sh
schematool -dbType derby -initSchema
```

It should return the following details :

```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/esigelec-ue-lsp-hdp/hive-3.1.2/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:derby:;databaseName=/home/hadoop/esigelec-ue-lsp-hdp/hive-3.1.2/metastore_db;create=true
Metastore Connection Driver :    org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:       APP
Hive distribution version:       3.1.0
Metastore schema version:        3.1.0
````

**Maker sur the `Metastore connection URL` is located in the Hive directory (/home/hadoop/esigelec-ue-lsp-hdp/hive-3.1.2)**.

## Start Hive

Open a separate new **Ubuntu** terminal, execute the following commands:

```sh
hiveserver2 &
```

You can check the status of your Hive instance using the following links:

 - Hive : http://localhost:10002/

> ### **Note:**
> The above URL might take a while to become available, so be patient.

You can also check that the process is running using the following command in your other Ubuntu terminal:

```sh
jps -mlV | grep HiveServer2
```

You will have to wait for 4 **Hive Session ID** entries to be displayed in order to consider that Hive is properly started:

```
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/home/hadoop/esigelec-ue-lsp-hdp/hive-3.1.2/lib/log4j-slf4j-impl-2.10.0.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/share/hadoop/common/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.apache.logging.slf4j.Log4jLoggerFactory]
Metastore connection URL:        jdbc:derby:;databaseName=/home/hadoop/esigelec-ue-lsp-hdp/hive-3.1.2/metastore_db;create=true
Metastore Connection Driver :    org.apache.derby.jdbc.EmbeddedDriver
Metastore connection User:       APP
Hive distribution version:       3.1.0
Metastore schema version:        3.1.0
````

## Stop Hive processes

You can check that your Hive processes are started using the following command:

```sh
jps -mlV | grep HiveServer2$
```

If the command returns any entries, then you need to stop the Hive processes.

Unfortunately, there is no graceful way of doing it so you will need to kill the process using the following commands:

```sh
kill -9 $(jps -mlV | grep HiveServer2$ | awk '{ print $1 }')
```

You can check that your HDFS processes are stopped using the following command which should return no results:

```sh
jps | grep HiveServer2$
```

## Stop HDFS & Yarn processes

You can check that your HDFS & Yarn processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

If the command returns any entries, then you need to stop the HDFS & Yarn processes using the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon stop namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon stop resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon stop nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm --daemon stop nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm --daemon stop nodemanager
```

You can check that your HDFS & Yarn processes are stopped using the following command which should return no results:

```sh
jps | grep -E 'Node|Manager'$
```

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill $(jps -mlV | grep -E 'Node|Manager|NodeManager' | awk '{ print $1 }')
```
