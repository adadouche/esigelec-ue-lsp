# Create your first MapReduce Job

## Prerequisites

You will need to have a Hadoop cluster properly configured and up and running (HDFS + YARN):

```sh
C:\MyWork\hadoop
```

## Your ```

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
SET HADOOP_CLASSPATH=%JAVA_HOME%/lib/tools.jar
SET PATH=%JAVA_HOME%/bin;%JAVA_HOME%

.\bin\hadoop com.sun.tools.javac.Main WordCount.java
jar cf WordCount.jar WordCount*.class
```

## Create the directory structure and data


Execute the following commands:

```
.\bin\hadoop --config %HADOOP_HOME%\etc\hadoop\client-1 fs -mkdir -p /wordcount/input

.\bin\hadoop --config %HADOOP_HOME%\etc\hadoop\client-1 fs -put WordCount.java /wordcount/input
```

## Execute the MapReduce Job

Execute the following commands:

```
.\bin\hadoop --config %HADOOP_HOME%\etc\hadoop\client-1 fs -rmdir /wordcount/output

.\bin\hadoop --config %HADOOP_HOME%\etc\hadoop\client-1 jar WordCount.jar WordCount /wordcount/input /wordcount/output
```


## Check the results

Execute the following commands:

```
.\bin\hadoop --config %HADOOP_HOME%\etc\hadoop\client-1 fs -ls /wordcount/output/

.\bin\hadoop --config %HADOOP_HOME%\etc\hadoop\client-1 fs -cat /wordcount/output/part-r-00000
```
