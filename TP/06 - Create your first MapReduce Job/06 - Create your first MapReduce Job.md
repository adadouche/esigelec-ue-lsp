# Create your first MapReduce Job

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
git reset --hard origin/step-05
git clean -dfq
```

## Set your Hadoop & Java Home

Open a DOS command prompt and execute:

```sh
set HADOOP_HOME=C:\hadoop
set JAVA_HOME=C:\Program Files\Java\jdk1.8.0_221
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

## The WordCount example

This is the Word Count example code that you will be using.

Execute the following commands:

```
notepad %HADOOP_HOME%\WordCount.java
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

  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

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

  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
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

## Compile the code

Execute the following commands:

```
cd %HADOOP_HOME%

%HADOOP_HOME%\etc\hadoop\hadoop-env.cmd

SET HADOOP_CLASSPATH=%JAVA_HOME%/lib/tools.jar
SET PATH=%JAVA_HOME%/bin;%PATH%

hadoop com.sun.tools.javac.Main %HADOOP_HOME%\WordCount.java
jar cf %HADOOP_HOME%\WordCount.jar WordCount*.class
```

## Create the directory structure and data

Execute the following commands:

```
hdfs --config %HADOOP_HOME%\etc\hadoop-client dfs -mkdir -p /wordcount/input

hdfs --config %HADOOP_HOME%\etc\hadoop-client dfs -put %HADOOP_HOME%\WordCount.java /wordcount/input
```

## Execute the MapReduce Job

Execute the following commands:

```
hdfs --config %HADOOP_HOME%\etc\hadoop-client dfs -rm -r /wordcount/output

hadoop --config %HADOOP_HOME%\etc\hadoop-client jar %HADOOP_HOME%\WordCount.jar WordCount /wordcount/input /wordcount/output
```


## Check the results

Execute the following commands:

```
hdfs --config %HADOOP_HOME%\etc\hadoop-client dfs -ls /wordcount/output/

hdfs --config %HADOOP_HOME%\etc\hadoop-client dfs -cat /wordcount/output/part-r-00000
```

You can also check the status of your job in the Resource Manager using the following links:

 - Resource Manager	: http://localhost:8088/
