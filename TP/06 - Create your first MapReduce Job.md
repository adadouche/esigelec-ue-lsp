# Create your first MapReduce Job

## Goal

In this tutorial, using your ***simulated*** a multi node Hadoop cluster, you will now start write your first MapReduce job using the Word Count example.

To run the Word Count example, you will use the Word Count java class program it and see how many words it contains and how occurrences.

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

git reset --hard origin/new-step-05
git clean -dfq

export ENV_FILE=~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh
source $ENV_FILE

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc

rm -rf $HADOOP_HOME/tmp/*
rm -rf $HADOOP_HOME/data/*
rm -rf $HADOOP_HOME/logs/*
rm -rf $HADOOP_HOME/pid/*

hdfs --config $HADOOP_HOME/etc/hadoop-master-nn namenode -format -force -clusterID local
```

## The WordCount example

This is the Word Count example code that you will be using.

For more details about the example, you can check the following link:

 - [Map Reduce Tutorial](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html#Example:_WordCount_v1.0)

Execute the following commands:

```sh
mkdir -p $HADOOP_HOME/mr/wordcount/src

nano $HADOOP_HOME/mr/wordcount/src/WordCount.java
```

Save the following code in the file:

```java
import java.io.IOException;
import java.util.StringTokenizer;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {

  public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }

  public static class IntSumReducer extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(
      Text key,
      Iterable<IntWritable> values,
      Context context
    ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }

  public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
  }
}
```

## Compile the WordCount code

Execute the following commands:

```
export HADOOP_CLASSPATH=$JAVA_HOME/lib/tools.jar

mkdir -p $HADOOP_HOME/mr/wordcount/classes
hadoop com.sun.tools.javac.Main -d $HADOOP_HOME/mr/wordcount/classes $HADOOP_HOME/mr/wordcount/src/*.java

mkdir -p $HADOOP_HOME/mr/wordcount/jar

jar cf $HADOOP_HOME/mr/wordcount/jar/WordCount.jar -C $HADOOP_HOME/mr/wordcount/classes .
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


## Create the directory structure and data

Execute the following commands:

```
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /wordcount/input

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put $HADOOP_HOME/mr/wordcount/src/WordCount.java /wordcount/input
```

## Execute the MapReduce Job

Execute the following commands:

```
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -rm -r /wordcount/output

hadoop --config $HADOOP_HOME/etc/hadoop-client jar $HADOOP_HOME/mr/wordcount/jar/WordCount.jar WordCount /wordcount/input /wordcount/output
```

As the file is relatively small, only one ma and one reduce task will be used.

The output of your job should look like this:

```
2019-12-01 14:12:59,603 INFO client.RMProxy: Connecting to ResourceManager at localhost/127.0.0.1:8032
2019-12-01 14:13:00,075 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
2019-12-01 14:13:00,093 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1575205900027_0001
2019-12-01 14:13:00,202 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2019-12-01 14:13:00,341 INFO input.FileInputFormat: Total input files to process : 1
2019-12-01 14:13:00,379 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2019-12-01 14:13:00,428 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2019-12-01 14:13:00,449 INFO mapreduce.JobSubmitter: number of splits:1
2019-12-01 14:13:00,587 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2019-12-01 14:13:00,610 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1575205900027_0001
2019-12-01 14:13:00,610 INFO mapreduce.JobSubmitter: Executing with tokens: []
2019-12-01 14:13:00,753 INFO conf.Configuration: resource-types.xml not found
2019-12-01 14:13:00,754 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2019-12-01 14:13:00,961 INFO impl.YarnClientImpl: Submitted application application_1575205900027_0001
2019-12-01 14:13:01,002 INFO mapreduce.Job: The url to track the job: http://localhost:8088/proxy/application_1575205900027_0001/
2019-12-01 14:13:01,003 INFO mapreduce.Job: Running job: job_1575205900027_0001
2019-12-01 14:13:08,107 INFO mapreduce.Job: Job job_1575205900027_0001 running in uber mode : false
2019-12-01 14:13:08,108 INFO mapreduce.Job:  map 0% reduce 0%
2019-12-01 14:13:14,170 INFO mapreduce.Job:  map 100% reduce 0%
2019-12-01 14:13:20,197 INFO mapreduce.Job:  map 100% reduce 100%
2019-12-01 14:13:20,204 INFO mapreduce.Job: Job job_1575205900027_0001 completed successfully
2019-12-01 14:13:20,276 INFO mapreduce.Job: Counters: 54
        File System Counters
                FILE: Number of bytes read=2089
                FILE: Number of bytes written=455219
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=2169
                HDFS: Number of bytes written=1700
                HDFS: Number of read operations=8
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
                HDFS: Number of bytes read erasure-coded=0
        Job Counters
                Launched map tasks=1
                Launched reduce tasks=1
                Data-local map tasks=1
                Total time spent by all maps in occupied slots (ms)=3345
                Total time spent by all reduces in occupied slots (ms)=3441
                Total time spent by all map tasks (ms)=3345
                Total time spent by all reduce tasks (ms)=3441
                Total vcore-milliseconds taken by all map tasks=3345
                Total vcore-milliseconds taken by all reduce tasks=3441
                Total megabyte-milliseconds taken by all map tasks=3425280
                Total megabyte-milliseconds taken by all reduce tasks=3523584
        Map-Reduce Framework
                Map input records=61
                Map output records=160
                Map output bytes=2486
                Map output materialized bytes=2089
                Input split bytes=117
                Combine input records=160
                Combine output records=96
                Reduce input groups=96
                Reduce shuffle bytes=2089
                Reduce input records=96
                Reduce output records=96
                Spilled Records=192
                Shuffled Maps =1
                Failed Shuffles=0
                Merged Map outputs=1
                GC time elapsed (ms)=116
                CPU time spent (ms)=790
                Physical memory (bytes) snapshot=605380608
                Virtual memory (bytes) snapshot=2266464124928
                Total committed heap usage (bytes)=1221066752
                Peak Map Physical memory (bytes)=311832576
                Peak Map Virtual memory (bytes)=1229532835840
                Peak Reduce Physical memory (bytes)=293548032
                Peak Reduce Virtual memory (bytes)=1036931289088
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters
                Bytes Read=2052
        File Output Format Counters
                Bytes Written=1700
```

It will include a tracker URL that you can check to understand the number of mapper/reducer used during the process etc.

## Check the results

Execute the following commands:

```
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -ls /wordcount/output/

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -cat /wordcount/output/part-r-00000
```

You can also check the status of your job in the Resource Manager using the following links:

 - Resource Manager	: http://localhost:8088/

And the applications:

 - http://localhost:8088/cluster/apps

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

<div style="background-color: #D3D3D3; padding: 20px;  border: 1px solid black;" >

## Quiz

Now that you have run successfully your first MapReduce job, let's try to write a new MR job using the 1500000 Sales Records as an input:

First, create a new MapReduce Java program named `WordCountSalesRecord` and use the previous `WordCount` code as an example.

In the `WordCount` example, the `StringTokenizer` use the default delimiter (space), but the ***1500000 Sales Records*** file uses a comma.

So, you can adjust the `StringTokenizer` instantiation using the following constructor:

```java
StringTokenizer (String str, String delim)
```

Then instead of counting all words, you will exclude all numeric and date values like (Order Date, Order ID, Ship Date, Units Sold, Unit Price, Unit Cost, Total Revenue, Total Cost, Total Profit).

To do so, try to cast the value into an `numeric` or a `date`, and if no exception is thrown then count it.

The date format is 'm/d/yyyy' and you can use the ```SimpleDateFormat.parse(String s)``` method.

The ```SimpleDateFormat``` can be instantiated with a constructor taht will take toe date format ("M/d/yyyy").

Recompile your class and generate the JAR file again before executing the MapReduce job.

#### Hint

Once your ```WordCountSalesOrder``` class code is completed, here is the commands to compile and execute it as a MapReduce job:

```
cd ~/esigelec-ue-lsp-hdp/

hadoop com.sun.tools.javac.Main -d $HADOOP_HOME/mr/wordcount/classes $HADOOP_HOME/mr/wordcount/src/*.java
jar cf $HADOOP_HOME/mr/wordcount/jar/WordCount.jar -C $HADOOP_HOME/mr/wordcount/classes .

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /wordcountsales
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /wordcountsales/input

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put /home/hadoop/1500000_Sales_Records.csv /wordcountsales/input

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -rm -r /wordcountsales/output

hadoop --config $HADOOP_HOME/etc/hadoop-client jar $HADOOP_HOME/mr/wordcount/jar/WordCount.jar WordCountSalesOrder /wordcountsales/input /wordcountsales/output

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -ls /wordcountsales/output

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -cat /wordcountsales/output/part-r-00000
```

How many time the word **`France`** is mentioned?
</div>
