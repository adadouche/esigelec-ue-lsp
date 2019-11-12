# Get Hadoop & Hive from GitHub on WSL

## Clone the Git Repository

A Git repository is available for you to download the base version of:

 - Hadoop 3.2.1
 - Hive 3.2.1

Open an Ubuntu command prompt and execute:

```sh
mkdir /mnt/c/hadoop-hive

git clone https://github.com/adadouche/esigelec-ue-lsp-hdp.git /mnt/c/hadoop-hive
```

Now checkout the current step branch:

```
cd /mnt/c/hadoop-hive

git fetch --all
git reset --hard origin/step-07
git clean -dfq
```

## Extend your BASH RC environment

In your Ubuntu terminal, execute the following commands:

First thing, you can do is to set the JAVA_HOME in the bashrc scripts so it will be initialized for every new bash session:

```sh
rm ~/.bashrc_hadoop_env

echo -e "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> ~/.bashrc_hadoop_env
echo -e "export HADOOP_HOME=/mnt/c/hadoop-hive/hadoop-3.2.1" >> ~/.bashrc_hadoop_env
echo -e "export HIVE_HOME=/mnt/c/hadoop-hive/hive-3.1.2" >> ~/.bashrc_hadoop_env

echo -e "export HADOOP_BIN_PATH=$HADOOP_HOME/bin" >> ~/.bashrc_hadoop_env
echo -e "export HADOOP_SBIN_PATH=$HADOOP_HOME/sbin" >> ~/.bashrc_hadoop_env

echo -e "export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop" >> ~/.bashrc_hadoop_env
echo -e "export HADOOP_LOG_DIR==$HADOOP_HOME/logs" >> ~/.bashrc_hadoop_env

echo -e "export \"HADOOP_OPTS=\$HADOOP_OPTS -Dhadoop.home='\$HADOOP_HOME'\"" >> ~/.bashrc_hadoop_env
echo -e "export \"HADOOP_OPTS=\$HADOOP_OPTS -Dyarn.home='\$HADOOP_HOME'\"" >> ~/.bashrc_hadoop_env

echo -e "export PATH=\$PATH:\$HADOOP_BIN_PATH" >> ~/.bashrc_hadoop_env
echo -e "export PATH=\$PATH:\$HADOOP_SBIN_PATH" >> ~/.bashrc_hadoop_env
echo -e "export PATH=\$PATH:\$JAVA_HOME/bin" >> ~/.bashrc_hadoop_env
echo -e "export PATH=\$PATH:\$HIVE_HOME/bin" >> ~/.bashrc_hadoop_env

echo -e "source ~/.bashrc_hadoop_env" >> ~/.bashrc
```

## Start HDFS NameNode & DataNode process

If the HDFS processes are not started yet, you will need to start.

Open a DOS command prompt (not an Ubuntu one), execute the following commands to set the environment variables:

```
set HADOOP_HOME=/mnt/c/hadoop-hive/hadoop-3.2.1
```

Then, execute the following command to cleanup anything from previous experiments:

```sh
bash -ic "rm -rf %HADOOP_HOME%/tmp; rm -rf %HADOOP_HOME%/data; hdfs --config %HADOOP_HOME%/etc/hadoop-master-nn namenode -format -force"
```

Then, execute the following command to start your Name Node and 3 Data Nodes:

```sh
start "hdfs - master namenode" bash -ic "export HADOOP_PID_DIR=%HADOOP_HOME%/pid/master-nn; rm %HADOOP_HOME%/pid/*.pid; hdfs --config %HADOOP_HOME%/etc/hadoop-master-nn namenode"

start "hdfs - slave-1 datanode" bash -ic "export HADOOP_PID_DIR=%HADOOP_HOME%/pid/slave-1-dn; rm %HADOOP_HOME%/pid/*.pid; hdfs --config %HADOOP_HOME%/etc/hadoop-slave-1-dn datanode"
start "hdfs - slave-2 datanode" bash -ic "export HADOOP_PID_DIR=%HADOOP_HOME%/pid/slave-2-dn; rm %HADOOP_HOME%/pid/*.pid; hdfs --config %HADOOP_HOME%/etc/hadoop-slave-2-dn datanode"
start "hdfs - slave-3 datanode" bash -ic "export HADOOP_PID_DIR=%HADOOP_HOME%/pid/slave-3-dn; rm %HADOOP_HOME%/pid/*.pid; hdfs --config %HADOOP_HOME%/etc/hadoop-slave-3-dn datanode"
```

You can check the status of your cluster using the following links:

 - Name Node : http://localhost:9870/

## Start YARN Resource Manager and Node Manager daemon

Then, execute the following command:

```
start "yarn - master resourcemanager" bash -ic "export HADOOP_PID_DIR=%HADOOP_HOME%/pid/master-rm; rm %HADOOP_HOME%/pid/*.pid; yarn --config "%HADOOP_HOME%/etc/hadoop-master-rm" resourcemanager"

start "yarn - slave-1 nodemanager" bash -ic "export HADOOP_PID_DIR=%HADOOP_HOME%/pid/slave-1-nm; rm %HADOOP_HOME%/pid/*.pid; yarn --config "%HADOOP_HOME%/etc/hadoop-slave-1-nm" nodemanager"
start "yarn - slave-2 nodemanager" bash -ic "export HADOOP_PID_DIR=%HADOOP_HOME%/pid/slave-2-nm; rm %HADOOP_HOME%/pid/*.pid; yarn --config "%HADOOP_HOME%/etc/hadoop-slave-2-nm" nodemanager"
start "yarn - slave-3 nodemanager" bash -ic "export HADOOP_PID_DIR=%HADOOP_HOME%/pid/slave-3-nm; rm %HADOOP_HOME%/pid/*.pid; yarn --config "%HADOOP_HOME%/etc/hadoop-slave-3-nm" nodemanager"
```

You can check the status of your cluster using the following links:

 - Resource Manager	: http://localhost:8088/

## Test The WordCount example

This is the Word Count example code that you will be using.

Open an Ubuntu terminal (not a DOS command prompt).

Execute the following commands:

```
vi $HADOOP_HOME/WordCount.java
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
export HADOOP_CLASSPATH=$JAVA_HOME/lib/tools.jar:$HADOOP_CLASSPATH

cd %HADOOP_HOME%

hadoop com.sun.tools.javac.Main $HADOOP_HOME/WordCount.java

find . -type f -name '*.class' -print0 | xargs -0 jar cvf $HADOOP_HOME/WordCount.jar
```

## Create the directory structure and data

Execute the following commands:

```
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -mkdir -p /wordcount/input

hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -put $HADOOP_HOME/WordCount.java /wordcount/input
```

## Execute the MapReduce Job

Execute the following commands:

```
hdfs --config $HADOOP_HOME/etc/hadoop-client dfs -rm -r /wordcount/output

hadoop --config $HADOOP_HOME/etc/hadoop-client jar $HADOOP_HOME/WordCount.jar WordCount /wordcount/input /wordcount/output
```

## Check the results

Execute the following commands:

```
hdfs --config %HADOOP_HOME%\etc\hadoop-client dfs -ls /wordcount/output/

hdfs --config %HADOOP_HOME%\etc\hadoop-client dfs -cat /wordcount/output/part-r-00000
```

You can also check the status of your job in the Resource Manager using the following links:

 - Resource Manager	: http://localhost:8088/
