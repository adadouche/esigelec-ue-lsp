# Spark Standalone with Local & HDFS files

## Goal

In this tutorial, you will start building your first Spark programs and tryout different options using Spark Standalone mode:

- consume a text file store locally then in HDFS (WordCount example)
- consume a CSV file store locally then in HDFS (1500000_Sales_Records example)

The main steps are:

  - Start HDFS processes
  - Start Spark processes
  - Use PySpark to execute your scripts

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
git reset --hard origin/step-09
git clean -dfq

./.setup.sh
```

## Start HDFS processes

In order to reduce the resource footprint, you will start using a ***smaller** Hadoop cluster:

- 1 Name Node
- 1 Data Node

You can kill your HDFS & YARN processes using the following command:

```sh
kill -9 $(jps -mlV | grep -E 'Node' | awk '{ print $1 }')
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
```

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node'$
```

You can also get details about HDFS processes using the following URL:

 - Name Node : http://localhost:9870/

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

## Start Spark Master & Slave processes

You can check that your Spark Master & Slave processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

If the command returns the following, then you don't need to start the Spark Master & Slave processes again:
 - 1 Master
 - 1 Worker

If you need to start the Spark Master & Slave processes, execute the following commands:

```sh
start-master.sh
start-slaves.sh
```

> ### **Note:**
> if you receive the following error message:
```
localhost: ssh: connect to host localhost port 22: Connection refused
```
> then execute the following commands to restart ssh:
```sh
sudo service ssh restart
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

You can also get details about HDFS processes using the following URL:

 - Spark Web UI : http://localhost:8080/

## Spark Jobs - Local/HDFS Text file

In your **Ubuntu** terminal, execute:

```sh
pyspark
```

Then you can paste the following code:

```python
input_file = sc.textFile("file:///home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/mr/wordcount/src/WordCount.java")
counts = input_file.flatMap(lambda line: line.split(" ")) \
             .map(lambda word: (word, 1)) \
             .reduceByKey(lambda a, b: a + b) \
             .toDF()
counts.show()
```

It assumes that you have the following local file available /home/hadoop/esigelec-ue-lsp-hdp/hadoop-3.2.1/mr/wordcount/src/WordCount.java.

Now, paste the following code:

```python
input_file = sc.textFile("hdfs://localhost:9000/wordcount/input/WordCount.java")
counts = input_file.flatMap(lambda line: line.split(" ")) \
             .map(lambda word: (word, 1)) \
             .reduceByKey(lambda a, b: a + b) \
             .toDF()
counts.show()
```

It assumes that you have started HDFS and uploaded the WordCount.java in /wordcount/input.

Note the difference in the file path with the address of your HDFS NameNode server.

You can also check the Spark job execution at the following URL:

  - Spark Shell Jobs: http://localhost:4040/jobs/

## Spark Jobs - Local/HDFS CSV file

Now, let's count words only in the 1500000_Sales_Records file as done previously (where you skipped dates and numbers) using the following code:

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


input_file = sc.textFile("file:////home/hadoop/esigelec-ue-lsp-hdp/1500000_Sales_Records.csv")

# add row id to each lines
input_file = input_file.zipWithIndex()
# skip the first line
input_file = input_file.filter(lambda tuple: tuple[1] > 0)

counts = input_file.flatMap(lambda tuple: tuple[0].split(",")) \
  .filter(lambda word: not isfloat(word)) \
  .filter(lambda word: not isdate(word)) \
  .map(lambda word: (word, 1)) \
  .reduceByKey(lambda a, b: a + b) \
  .toDF()

counts.show()
```

Now, let's run a different version of the code that will achieve the same:

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


input_file = sc.textFile("file:////home/hadoop/esigelec-ue-lsp-hdp/1500000_Sales_Records.csv")

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

You can notice that the second version is faster than the first one because the filtering is taking place on a reduced number of items (after reduceByKey instead of before) but required a larger amount of memory for the Shuffle.

And finally let's use your HDFS CSV file instead with the following code:

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

You can check the Spark job execution at the following URL:

- Spark Shell Jobs: http://localhost:4040/jobs/

You can notice that the HDFS version is almost as fast as the local version, but in our case we have only one DataNode and one Spark slave which add some overhead and no benefits.

## Stop Spark Master & Slave processes

You can check that your Spark Master & Slave processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

If the command returns any entries, then you can stop processes using the following commands:

```sh
stop-slaves.sh
stop-master.sh
```

You can check that your HDFS processes are started using the following command:

```sh
jps | grep -E 'Worker|Master'$
```

You can also get details about HDFS processes using the following URL:

- Spark Web UI : http://localhost:8080/


## Stop HDFS processes

You can check that your HDFS & YARN processes are started using the following command:

```sh
jps | grep -E 'Node'$
```

If the command returns any entries, then you need to stop the HDFS & YARN processes using the following commands:

```sh
export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-nn
hdfs --config $HADOOP_HOME/etc/hadoop-master-nn --daemon stop namenode

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-dn
hdfs --config $HADOOP_HOME/etc/hadoop-slave-1-dn --daemon stop datanode
```

You can check that your HDFS & YARN processes are stopped using the following command which should return no results:

```sh
jps | grep -E 'Node'$
```

If processes remains in the list then you can execute the following commands to kill them:

```sh
kill -9 $(jps -mlV | grep -E 'Node' | awk '{ print $1 }')
```
