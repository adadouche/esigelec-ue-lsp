# Setup your cluster for MapReduce

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

git reset --hard origin/new-step-04
git clean -dfq
```

## Set your Hadoop environment variables

In your **Ubuntu** terminal, execute:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh
```

## Extend the YARN environment variables

In your **Ubuntu** terminal, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

echo -e "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> $HADOOP_HOME/etc/hadoop/yarn-env.sh

echo -e "export /"YARN_OPTS=/$YARN_OPTS -Dyarn.home='/$HADOOP_HOME'/"" >> $HADOOP_HOME/etc/hadoop/yarn-env.sh
echo -e "export /"YARN_OPTS=/$YARN_OPTS -Dhadoop.home='/$HADOOP_HOME'/"" >> $HADOOP_HOME/etc/hadoop/yarn-env.sh
```

## Create the Master Resource Manager config file - core-site

In your **Ubuntu** terminal, execute the following commands:

```sh
mkdir -p "$HADOOP_HOME/etc/hadoop-master-rm"

cp "$HADOOP_HOME/etc/hadoop/core-site.xml" "$HADOOP_HOME/etc/hadoop-master-rm/core-site.xml"

nano "$HADOOP_HOME/etc/hadoop-master-rm/core-site.xml"
```

Replace the file content with:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>${yarn.home}/tmp/hadoop-master-rm</value>
    </property>
</configuration>
```

## Create the Master Resource Manager config file - yarn-site

In your **Ubuntu** terminal, execute the following commands:

```sh
cp "$HADOOP_HOME/etc/hadoop/yarn-site.xml" "$HADOOP_HOME/etc/hadoop-master-rm/yarn-site.xml"

nano "$HADOOP_HOME/etc/hadoop-master-rm/yarn-site.xml"
```

Replace the file content with:

```xml
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
        <name>yarn.node-attribute.fs-store.root-dir</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
    </property>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>    
</configuration>
```

## Create the Master Resource Manager config file - capacity-scheduler

Now, create the capacity scheduler configuration.

In your **Ubuntu** terminal, execute the following commands:

```sh
nano "$HADOOP_HOME/etc/hadoop-master-rm/capacity-scheduler.xml"
```

Replace the file content with:

```xml
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

## Create the Slaves Node Manager config file - yarn-site

In your **Ubuntu** terminal, execute the following commands:

```sh
mkdir -p "$HADOOP_HOME/etc/hadoop-slave-1-nm"

cp "$HADOOP_HOME/etc/hadoop/yarn-site.xml" "$HADOOP_HOME/etc/hadoop-slave-1-nm/yarn-site.xml"

nano "$HADOOP_HOME/etc/hadoop-slave-1-nm/yarn-site.xml"
```

Replace the file content with:

```xml
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
        <name>yarn.nodemanager.hostname</name>
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

In your **Ubuntu** terminal, execute the following commands:

```sh
mkdir -p "$HADOOP_HOME/etc/hadoop-slave-2-nm"
mkdir -p "$HADOOP_HOME/etc/hadoop-slave-3-nm"

cp $HADOOP_HOME/etc/hadoop-slave-1-nm/yarn-site.xml $HADOOP_HOME/etc/hadoop-slave-2-nm/yarn-site.xml
cp $HADOOP_HOME/etc/hadoop-slave-1-nm/yarn-site.xml $HADOOP_HOME/etc/hadoop-slave-3-nm/yarn-site.xml

export xml_file=$HADOOP_HOME/etc/hadoop-slave-2-nm/yarn-site.xml

sed -i 's/8140/8240/g' $xml_file
sed -i 's/8142/8242/g' $xml_file

export xml_file=$HADOOP_HOME/etc/hadoop-slave-3-nm/yarn-site.xml

sed -i 's/8140/8340/g' $xml_file
sed -i 's/8142/8342/g' $xml_file
```

You can now open the generate/modified xml file.

```sh
more $HADOOP_HOME/etc/hadoop-slave-1-nm/yarn-site.xml
more $HADOOP_HOME/etc/hadoop-slave-2-nm/yarn-site.xml
more $HADOOP_HOME/etc/hadoop-slave-3-nm/yarn-site.xml
```

## Create the Slaves Node Manager config file - mapred-site

In your **Ubuntu** terminal, execute the following commands:

```sh
cp "$HADOOP_HOME/etc/hadoop/mapred-site.xml" "$HADOOP_HOME/etc/hadoop-slave-1-nm/mapred-site.xml"

nano "$HADOOP_HOME/etc/hadoop-slave-1-nm/mapred-site.xml"
```

Replace the file content with:

```xml
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

In your **Ubuntu** terminal, execute the following commands:

```sh
cp $HADOOP_HOME/etc/hadoop-slave-1-nm/mapred-site.xml $HADOOP_HOME/etc/hadoop-slave-2-nm/mapred-site.xml
cp $HADOOP_HOME/etc/hadoop-slave-1-nm/mapred-site.xml $HADOOP_HOME/etc/hadoop-slave-3-nm/mapred-site.xml

export xml_file=$HADOOP_HOME/etc/hadoop-slave-2-nm/mapred-site.xml

sed -i 's/13562/23562/g' $xml_file

export xml_file=$HADOOP_HOME/etc/hadoop-slave-3-nm/mapred-site.xml

sed -i 's/13562/33562/g' $xml_file
```

You can now open the generate/modified xml file.

```sh
more $HADOOP_HOME/etc/hadoop-slave-1-nm/mapred-site.xml
more $HADOOP_HOME/etc/hadoop-slave-2-nm/mapred-site.xml
more $HADOOP_HOME/etc/hadoop-slave-3-nm/mapred-site.xml
```

## Create the Slaves Node Manager config file - core-site

In your **Ubuntu** terminal, execute the following commands:

```sh
cp "$HADOOP_HOME/etc/hadoop/core-site.xml" "$HADOOP_HOME/etc/hadoop-slave-1-nm/core-site.xml"

nano "$HADOOP_HOME/etc/hadoop-slave-1-nm/core-site.xml"
```

Replace the file content with:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/${hadoop.home}/tmp/hadoop-slave-1-nm</value>
    </property>
</configuration>
```

In your **Ubuntu** terminal, execute the following commands:

```sh
cp $HADOOP_HOME/etc/hadoop-slave-1-nm/core-site.xml $HADOOP_HOME/etc/hadoop-slave-2-nm/core-site.xml
cp $HADOOP_HOME/etc/hadoop-slave-1-nm/core-site.xml $HADOOP_HOME/etc/hadoop-slave-3-nm/core-site.xml

export xml_file=$HADOOP_HOME/etc/hadoop-slave-2-nm/core-site.xml

sed -i 's/slave-1/slave-2/g' $xml_file

export xml_file=$HADOOP_HOME/etc/hadoop-slave-3-nm/core-site.xml

sed -i 's/slave-1/slave-3/g' $xml_file
```

You can now open the generate/modified xml file.

```sh
more $HADOOP_HOME/etc/hadoop-slave-1-nm/core-site.xml
more $HADOOP_HOME/etc/hadoop-slave-2-nm/core-site.xml
more $HADOOP_HOME/etc/hadoop-slave-3-nm/core-site.xml
```

## Create the Log4j config files for all processes

In your **Ubuntu** terminal, execute the following commands:

```sh
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-master-rm/log4j.properties
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-slave-1-nm/log4j.properties
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-slave-2-nm/log4j.properties
cp $HADOOP_HOME/etc/hadoop/log4j.properties $HADOOP_HOME/etc/hadoop-slave-3-nm/log4j.properties
```

## Extend the client config files - mapred-site

In your **Ubuntu** terminal, execute the following commands:

```sh
cp "$HADOOP_HOME/etc/hadoop/mapred-site.xml" "$HADOOP_HOME/etc/hadoop-client/mapred-site.xml"

nano "$HADOOP_HOME/etc/hadoop-client/mapred-site.xml"
```

Replace the file content with:

```xml
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

## Extend the client config files - yarn-site

In your **Ubuntu** terminal, execute the following commands:

```sh
cp "$HADOOP_HOME/etc/hadoop/mapred-site.xml" "$HADOOP_HOME/etc/hadoop-client/yarn-site.xml"

nano "$HADOOP_HOME/etc/hadoop-client/yarn-site.xml"
```

Replace the file content with:

```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
</configuration>
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

## Start the Master Resource Manager

Open a new **Ubuntu** terminal, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm resourcemanager
```


## Start the Slaves Node Manager

Open a new **Ubuntu** terminal, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm nodemanager
```

Open a new **Ubuntu** terminal, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm nodemanager
```

Open a new **Ubuntu** terminal, execute the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-3-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm nodemanager
```

You can check the status of your cluster using the following link:

 - Resource Manager	: http://localhost:8088/cluster/nodes

You can now close each of the Ubuntu terminal for the Master Resource Manager & Slaves Node Manager.

## Stop HDFS processes

You can check that your HDFS processes are started using the following command:

```sh
jps | grep Node$
```

If the command returns any entries, then you need to stop the HDFS processes using the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon stop namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-2-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-3-dn --daemon stop datanode
```

You can check that your HDFS processes are stopped using the following command which should return no results:

```sh
jps | grep Node$
```
