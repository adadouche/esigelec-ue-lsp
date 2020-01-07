# Spark on YARN

## Goal

In this tutorial, you will start building your first Spark programs and tryout different options using Spark on YARN mode:

- consume a text file store locally then in HDFS (WordCount example)
- consume a CSV file store locally then in HDFS (1500000_Sales_Records example)
- try Spark "client" & "cluster" mode on YARN

The main steps are:

  - Start HDFS/YARN processes
  - Configure Spark for YARN (JAR caching for optimization)
  - Use PySpark or Spark-Submit to execute your scripts

In this mode, the process is handled by YARN as a Resource Manager, therefore you don't need to start the Spark cluster.

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

git pull
git reset --hard origin/step-10
git clean -dfq

./.setup.sh
```

## Start HDFS & YARN processes

In order to reduce the resource footprint, you will start using a ***smaller** Hadoop cluster:

- 1 Name Node
- 1 Data Node
- 1 Resource Manager
- 1 Node Manager

You can kill your HDFS & YARN processes using the following command:

```sh
kill -9 $(jps -mlV | grep -E 'Node|Manager' | awk '{ print $1 }')
```

Then start the HDFS & YARN processes using the following commands:

```sh
# First we reformat the Name Node to avoid inconsistency issues
rm -rf $HADOOP_HOME/tmp/* $HADOOP_HOME/data/* $HADOOP_HOME/logs/* $HADOOP_HOME/pid/*
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode -format -force -clusterID local

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn  
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-nn  
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon start namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon start datanode

hdfs dfsadmin -safemode leave

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon start resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
export HADOOP_LOG_DIR=$HADOOP_HOME/logs/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon start nodemanager
```

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

You can also get details about HDFS processes using the following URL:

 - Name Node : http://localhost:9870/
 - Resource Manager	: http://localhost:8088/

## Upload data files in HDFS

In our example, you will be using the 1500000_Sales_Records and the WordCount java program as input files.


First, download the 1500000_Sales_Records file locally and extract it:

```sh
cd $LSP_HOME
rm  $LSP_HOME/'1500000_Sales_Records.7z'

wget http://eforexcel.com/wp/wp-content/uploads/2017/07/1500000%20Sales%20Records.7z

mv 1500000\ Sales\ Records.7z 1500000_Sales_Records.7z

7z e 1500000_Sales_Records.7z

mv 1500000\ Sales\ Records.csv 1500000_Sales_Records.csv
```

And finally, let's import the data files:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /wordcount/input

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put $HADOOP_HOME/mr/wordcount/src/WordCount.java /wordcount/input

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -Ddfs.block.size=10m -put $(echo ~)/esigelec-ue-lsp-hdp/1500000_Sales_Records.csv /1500000_Sales_Records.csv
```

## Configure Spark - Jar Cache

Create a new spark-defaults.conf file in the Hive configuration folder using the following command:

```sh
cp $SPARK_HOME/conf/spark-defaults.conf.template $SPARK_HOME/conf/spark-defaults.conf

nano $SPARK_HOME/conf/spark-defaults.conf
```

Then paste the following content

```sh
spark.yarn.jars                   hdfs://localhost:9000/spark-jars/*
```

## Configure Spark - Cache Spark Jars in HDFS

In order to work efficiently, you should cache all Spark Jars in HDFS and provide the cache location to Spark for YARN to use it.

To do execute the following to upload all Spark Jars in HDFS:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /spark-jars

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put $SPARK_HOME/jars/* /spark-jars/
```
## Spark Jobs - Use YARN client mode (PySpark shell)

One of the possible Spark deployment is to leverage the YARN cluster either in client or cluster mode.

To start the PySpark shell utility in YARN client mode, you can execute the following commands:

```sh
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop-client

pyspark --master yarn --deploy-mode client
```

Once started you can check the YARN Resource Manager for a running application ***PySparkShell***:

  - YARN Resource Manage: http://localhost:8088/cluster/apps

Every command submitted in pyspark will now be executed within YARN using the client mode (so if you close you pyspark session, the job will be killed).

Try again your code:

```python
import datetime

def isfloat(word):
  try:
    float(word)
    return True
  except ValueError:
    return False


def isdate(word):
  try:
    datetime.datetime.strptime(word, '%m/%d/%Y')
    return True
  except ValueError:
    return False


input_file = sc.textFile("hdfs://localhost:9000/1500000_Sales_Records.csv")

# add row id to each lines
input_file = input_file.zipWithIndex()
# skip the first line
input_file = input_file.filter(lambda tuple: tuple[1] > 0)

counts = input_file.flatMap(lambda tuple: tuple[0].split(",")) \
  .map(lambda word: (word, 1)) \
  .reduceByKey(lambda a, b: a + b) \
  .filter(lambda tuple: not isfloat(tuple[0])) \
  .filter(lambda tuple: not isdate(tuple[0])) \
  .toDF()

counts.show()
```

if you exit you pyspark (using exit()), you will see that the PySparkShell application will be terminated (in the YARN Resource manager http://localhost:8088).

The PySpark shell utility doesn't support the YARN cluster mode.

So, to try out the YARN cluster mode, you can use the Spark-submit utility.

## Spark Jobs - Use YARN cluster mode (with Spark-Submit)

To start the Spark-Submit shell utility in YARN cluster mode, you will first need to create your python program file.

Execute the following commands to create the 1500000_Sales_Records.py file:

```sh
mkdir -p $SPARK_HOME/examples/py/

nano $SPARK_HOME/examples/py/1500000_Sales_Records.py
```

Then you can cpaste the following code:

```python
from pyspark import SparkContext, SparkConf
from pyspark.sql import SQLContext
from pyspark import sql

conf = SparkConf().setAppName("1500000_Sales_Records")
sc = SparkContext(conf=conf)
sqlContext = sql.SQLContext(sc)

import datetime

def isfloat(word):
  try:
    float(word)
    return True
  except ValueError:
    return False


def isdate(word):
  try:
    datetime.datetime.strptime(word, '%m/%d/%Y')
    return True
  except ValueError:
    return False


input_file = sc.textFile("hdfs://localhost:9000/1500000_Sales_Records.csv")

# add row id to each lines
input_file = input_file.zipWithIndex()
# skip the first line
input_file = input_file.filter(lambda tuple: tuple[1] > 0)

input_file.flatMap(lambda tuple: tuple[0].split(",")) \
  .map(lambda word: (word, 1)) \
  .reduceByKey(lambda a, b: a + b) \
  .filter(lambda tuple: not isfloat(tuple[0])) \
  .filter(lambda tuple: not isdate(tuple[0])) \
  .saveAsTextFile("hdfs://localhost:9000/1500000_Sales_Records_result")
```

As you can notice the program is slightly different from the one used in PySpark as it required to explicitly initialize the Spark Context and SQL Context.
Then, you can execute the following commands:

```sh
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop-client

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -rm -R -f /1500000_Sales_Records_result

spark-submit \
  --master yarn \
  --deploy-mode cluster \
  $SPARK_HOME/examples/py/1500000_Sales_Records.py

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -ls  /1500000_Sales_Records_result
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -cat /1500000_Sales_Records_result/part-00000
```

You can also get details about the process using the following URL:

 - YARN Resource Manager : http://localhost:8088/

## Spark Jobs - Use YARN client mode (with Spark-Submit)

You can also run it in ***client mode***:

```sh
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop-client

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -rm -R -f /1500000_Sales_Records_result

spark-submit \
  --master yarn \
  --deploy-mode client \
  $SPARK_HOME/examples/py/1500000_Sales_Records.py

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -cat /1500000_Sales_Records_result/part-00000
```

You can also get details about the process using the following URL:

 - YARN Resource Manager : http://localhost:8088/

## Spark Jobs - Use local mode (with Spark-Submit)

You can also run it in ***Standalone mode***:

```sh
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -rm -R -f /1500000_Sales_Records_result

spark-submit $SPARK_HOME/examples/py/1500000_Sales_Records.py

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -ls  /1500000_Sales_Records_result
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -cat /1500000_Sales_Records_result/part-00000
```

You can also get details about the process using the following URL:

 - Spark Web UI : http://localhost:4040/

## Stop HDFS & YARN processes

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node|Manager'$
```

If the command returns any entries, then you need to stop the HDFS & YARN processes using the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon stop namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon stop datanode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon stop resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon stop nodemanager
```

You can check that your HDFS & YARN processes are stopped using the following command which should return no results:

```sh
jps | grep -E 'Node|Manager'$
```

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill -9 $(jps -mlV | grep -E 'Node|Manager|NodeManager' | awk '{ print $1 }')
```
