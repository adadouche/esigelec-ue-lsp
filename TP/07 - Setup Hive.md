# Get Hadoop & Hive from GitHub on WSL

## Prerequisites

If you didn't manage to finish the previous step, you can start from fresh using the last step branch from Git.

It assumes that you don't have an existing directory **esigelec-ue-lsp-hdp** in your Ubuntu home directory (**~**).

Open an **Ubuntu** terminal and execute:

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
rm $ENV_FILE

echo -e "umask 022" >> $ENV_FILE

echo -e "export LSP_HOME=$(echo ~)/esigelec-ue-lsp-hdp" >> $ENV_FILE

echo -e "export HIVE_HOME=\$LSP_HOME/hive-3.1.2" >> $ENV_FILE
echo -e "export HADOOP_HOME=\$LSP_HOME/hadoop-3.2.1" >> $ENV_FILE

echo -e "export PATH=\$PATH:\$HADOOP/bin" >> $ENV_FILE
echo -e "export PATH=\$PATH:\$HADOOP/sbin" >> $ENV_FILE
echo -e "export PATH=\$PATH:\$HIVE_HOME/bin" >> $ENV_FILE
echo -e "export PATH=\$PATH:\$JAVA_HOME/bin" >> $ENV_FILE

echo -e "source $ENV_FILE" >> ~/.bashrc

source $ENV_FILE
```

## Copy GUAVA version from Hadoop to the Hive 3.1.2 distribution

The GUAVA version used across Hadoop and Hive are different which is causing issue, this is why you need to copy over the one from Hadoop to Hive.

In your **Ubuntu** terminal, execute:

```sh
mv $HIVE_HOME/lib/guava-19.0.jar $HIVE_HOME/lib/guava-19.0.jar.old
cp $HADOOP_HOME/share/hadoop/common/lib/guava-27.0-jre.jar $HIVE_HOME/lib/guava-27.0-jre.jar
```

## Start HDFS processes

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
```

If the command returns 1 NameNode and 3 DataNode entries, then you don't need to start the HDFS processes again.

If you need to start the HDFS processes, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

rm -rf $HADOOP_HOME/tmp
rm -rf $HADOOP_HOME/data
rm -rf $HADOOP_HOME/pid
rm -rf $HADOOP_HOME/logs

hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode -format -force

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
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
```

You can also get details about HDFS processes using the following URL:

 - http://localhost:9870/

## Start YARN processes

You can check that your YARN processes are started using the following command:

```sh
jps | grep Manager$
```

If the command returns 1 ResouceManager and 3 NodeManager entries, then you don't need to start the YARN processes again.

If you need to start the YARN processes, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

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

You can check that the processes are started by using the following command:

```sh
jps | grep Manager$
```

You can also get details about HDFS processes using the following URL:

 - Resource Manager	: http://localhost:8088/

## Create Hive Meta store and tmp directory

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
cd $HIVE_HOME

schematool -dbType derby -initSchema
```

You can then verify that the Meta store version:

```sh
schematool -dbType derby -info
```

## Start Hive

Open a new **Ubuntu** terminal, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hive_env.sh

cd $HIVE_HOME

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
