# Create your first MapReduce Job

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

git reset --hard origin/new-step-05
git clean -dfq
```

## Set your Hadoop environment variables

In your **Ubuntu** terminal, execute:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh
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

The output of your job should look like this:

```
INFO client.RMProxy: Connecting to ResourceManager at localhost/127.0.0.1:8032
WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1574232311821_0001
INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
INFO input.FileInputFormat: Total input files to process : 1
INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
INFO mapreduce.JobSubmitter: number of splits:1
INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1574232311821_0001
INFO mapreduce.JobSubmitter: Executing with tokens: []
INFO conf.Configuration: resource-types.xml not found
INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
INFO impl.YarnClientImpl: Submitted application application_1574232311821_0001
INFO mapreduce.Job: The url to track the job: http://localhost:8088/proxy/application_1574232311821_0001/
INFO mapreduce.Job: Running job: job_1574232311821_0001
INFO mapreduce.Job: Job job_1574232311821_0001 running in uber mode : false
INFO mapreduce.Job:  map 0% reduce 0%
INFO mapreduce.Job:  map 100% reduce 0%
INFO mapreduce.Job:  map 100% reduce 100%
INFO mapreduce.Job: Job job_1574232311821_0001 completed successfully
INFO mapreduce.Job: Counters: 54
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
                Total time spent by all maps in occupied slots (ms)=3996
                Total time spent by all reduces in occupied slots (ms)=2825
                Total time spent by all map tasks (ms)=3996
                Total time spent by all reduce tasks (ms)=2825
                Total vcore-milliseconds taken by all map tasks=3996
                Total vcore-milliseconds taken by all reduce tasks=2825
                Total megabyte-milliseconds taken by all map tasks=4091904
                Total megabyte-milliseconds taken by all reduce tasks=2892800
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
                GC time elapsed (ms)=122
                CPU time spent (ms)=890
                Physical memory (bytes) snapshot=605081600
                Virtual memory (bytes) snapshot=1193300152320
                Total committed heap usage (bytes)=1211105280
                Peak Map Physical memory (bytes)=314200064
                Peak Map Virtual memory (bytes)=775164583936
                Peak Reduce Physical memory (bytes)=290881536
                Peak Reduce Virtual memory (bytes)=418135568384
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

## Stop YARN processes

You can check that your YARN processes are started using the following command:

```sh
jps | grep Manager$
```

If the command returns any entries, then you need to stop the YARN processes using the following commands:

```sh
source ~/esigelec-ue-lsp-hdp/.set_hadoop_env.sh

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-master-rm
yarn --config $HADOOP_HOME/etc/hadoop-master-rm --daemon stop resourcemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-1-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-1-nm --daemon stop nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-2-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-2-nm --daemon stop nodemanager

export HADOOP_PID_DIR=$HADOOP_HOME/pid/hadoop-slave-3-nm
yarn --config $HADOOP_HOME/etc/hadoop-slave-3-nm --daemon stop nodemanager
```

You can check that your YARN processes are stopped using the following command which should return no results:

```sh
jps | grep Manager$
```
