# Install a standalone single node Hadoop cluster on Windows

## Prerequisites

You will need to download and install the following software on your Windows system (Windows 10 x64):

- Git: https://git-scm.com/downloads (x64 executable)

- Java JDK 1.8 : https://www.oracle.com/technetwork/java/javase/downloads/index.html (executable [jdk-8u221-windows-x64.exe](https://download.oracle.com/otn/java/jdk/8u221-b11/230deb18db3e4014bb8e3e8324f81b43/jdk-8u221-windows-x64.exe))

- Maven : https://maven.apache.org/download.cgi (Zip archive [apache-maven-3.6.2-bin.zip](http://mirrors.standaloneinstaller.com/apache/maven/maven-3/3.6.2/binaries/apache-maven-3.6.2-bin.zip))

- Protocol Buffers v2.5.0: https://github.com/protocolbuffers/protobuf/releases/tag/v2.5.0 (Zip archive [protoc-2.5.0-win32.zip](https://github.com/protocolbuffers/protobuf/releases/download/v2.5.0/protoc-2.5.0-win32.zip))

- CygWin x64: https://www.cygwin.com/ (executable [setup-x86_64.exe](https://www.cygwin.com/setup-x86_64.exe))

  - select zlib & zlib-devel (1.2.11-1)

  - avoid installing cMake from CygWin as it will collide with Visual Studio


- Visual Studio Community 2019: https://visualstudio.microsoft.com/vs/community/

>
> #### **Note**:
> I recommend you to create a directory **C:\MyTools** directory to extract the Zip archives
>

## Clone the Hadoop Git Repository

Open a "x64_x86 Cross Tools Command Prompt for VS 2019" (from the Start menu) and paste the following commands (that you can adjust if needed):

```sh
mkdir C:\MyWork
cd C:\MyWork
git clone https://github.com/apache/hadoop-common.git
```

This will clone the Git repository on your machine under a folder named **C:\MyWork\hadoop-common**

Once completed you can close the command prompt.

## Retarget the VS solution files

In order to build Hadoop on Windows, you will need to use native libraries. To do so, Visual Studio tooling will be used (CMake etc.).

However, in the Git repository, older Visual Studio solution files are made available (Visual Studio 2010).

Therefore you will need to "retarget" them.

Open a "x64_x86 Cross Tools Command Prompt for VS 2019" (from the Start menu) and paste the following commands (that you can adjust if needed):

```sh
devenv C:\MyWork\hadoop-common\hadoop-common-project\hadoop-common\src\main\native\native.sln /upgrade

devenv C:\MyWork\hadoop-common\hadoop-common-project\hadoop-common\src\main\winutils\winutils.sln /upgrade
```
You can also manuall open the SLN file in VS 2019 to apply the configuration you want.

## Set the target environment in the POM

Edit the following file :

```sh
C:\MyWork\hadoop-common\hadoop-hdfs-project\hadoop-hdfs\pom.xml
```

at around line **409** replace :

```
-G 'Visual Studio 10 Win64'"
```

by

```
-G 'Visual Studio 16 2019'"
```

Save the file.


Or you can run the following command:

```sh
powershell -Command "(gc C:\MyWork\hadoop-common\hadoop-hdfs-project\hadoop-hdfs\pom.xml) -replace 'Visual Studio 10 Win64', 'Visual Studio 16 2019' | Out-File -encoding ASCII C:\MyWork\hadoop-common\hadoop-hdfs-project\hadoop-hdfs\pom.xml"
```

## Build Hadoop from the source

In your "x64_x86 Cross Tools Command Prompt for VS 2019"

```sh
set Platform=x64
set M2_HOME=C:\MyTools\apache-maven-3.6.2
set JAVA_HOME=C:\Progra~1\Java\jdk1.8.0_221

set PATH=%PATH%;%JAVA_HOME%\bin
set PATH=%PATH%;%M2_HOME%\bin
set PATH=%PATH%;C:\MyTools\protoc-2.5.0-win32
set PATH=%PATH%;C:\cygwin64\bin

cd C:\MyWork\hadoop-common

mvn package -Pdist,native-win -DskipTests -Dtar -Dmaven.javadoc.skip=true
```

Once the build process completes, you can proceed with the deployment of the generated tar ball.

You should get the following trace/log displayed:

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  03:49 min
[INFO] Finished at: 2019-11-02T09:14:23+01:00
[INFO] ------------------------------------------------------------------------
```

## Deploy your Hadoop snapshot build

The generated tar ball will be stored in the following path (unless you changed some of the parameters):

```sh
C:\MyWork\hadoop-common\hadoop-dist\target\hadoop-3.0.0-SNAPSHOT.tar.gz
```

Extract the tar ball using the following commands:

```sh
cd C:\MyWork
tar -xvf C:\MyWork\hadoop-common\hadoop-dist\target\hadoop-3.0.0-SNAPSHOT.tar.gz
```

## Configure the Hadoop environment

##### Extend the environment variables

Open the following file:

```sh
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc\hadoop\hadoop-env.cmd
```

Locate the following line:

```
@rem Set Hadoop-specific environment variables here.
```

and add the following content right after:

```sh
set HADOOP_HOME=C:\MyWork\hadoop-3.0.0-SNAPSHOT

set HADOOP_BIN_PATH=%HADOOP_HOME%\bin
set HADOOP_CONF_DIR=%HADOOP_HOME%\etc\hadoop

set JAVA_HOME=C:\Progra~1\Java\jdk1.8.0_221
set PATH=%PATH%;%HADOOP_BIN_PATH%

set "HADOOP_HOME_OPTS=%HADOOP_HOME:\=/%"

set HADOOP_OPTS=-Dhadoop.home=%HADOOP_HOME_OPTS% %HADOOP_OPTS%
```

##### Configure the core-site.xml file

Open the following file:

```sh
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc/hadoop/core-site.xml
```

Paste in the following configuration:

```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/${hadoop.home}/tmp/hadoop-${user.name}</value>
    </property>
</configuration>
```

For more details about the configuration file, you can check the following link:

 - https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/core-default.xml

##### Configure the hdfs-site.xml file

Open the following file:

```sh
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc/hadoop/hdfs-site.xml
```

Paste in the following configuration:

```xml
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>1</value>
  </property>
</configuration>
```

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/hdfs-default.xml

##### Configure the mapred-site.xml file

Copy the `mapred-site.xml.template` into `mapred-site.xml` located in `C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc/hadoop/`.

Open the following file:

```sh
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc/hadoop/mapred-site.xml
```
Paste in the following configuration:

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
</configuration>
```

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml

##### Configure the yarn-site.xml file

Open the following file:

```sh
C:\MyWork\hadoop-3.0.0-SNAPSHOT\etc/hadoop/yarn-site.xml
```

Paste in the following configuration:

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
</configuration>
```

For more details about the configuration file, you can check the following link:

- https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-common/yarn-default.xml

## Initialize HDFS

Before starting using the HDFS, you will need to format it.

In a command prompt, execute the following commands to set the environment variables:

```
cd C:\MyWork\hadoop-3.0.0-SNAPSHOT

.\etc\hadoop\hadoop-env.cmd
```

Then, execute the following commands:

```
.\bin\hdfs namenode -format
```

This will format the name node `dfs`.

## Start HDFS daemons

In a command prompt, execute the following commands:

```
.\sbin\start-dfs.cmd
```

## Interact with the File System

Let's first create a directory:

```sh
.\bin\hdfs dfs -mkdir /config
```

Now let's copy the configuration file into it

```sh
.\bin\hdfs dfs -put %HADOOP_HOME%\etc\hadoop\*.xml /config
```

And finally, let's check the files are here:

```sh
.\bin\hdfs dfs -ls /config
```

Visualize one:

```sh
hdfs dfs -cat /config/yarn-site.xml
```

## Get the NameNode

You can also get details about HDFS and the NameNode using the following URL:

 - http://localhost:50070/

And access the file system:

 - http://localhost:50070/explorer.html#/

---

## Extra

### Reset your Hadoop Git repository

If you want to reset your local copy of the Hadoop GIT repo, you can run the following series of commands:

```
cd C:\MyWork\hadoop-common
git reset --hard
git clean -f -d
git pull -f
```

### Documentation generation

Since Java 1.8, the syntax to generate the doc has become more strict which will make to build to fail.

However if you want to get the documentation generate, you can check the following article:

  - https://blog.joda.org/2014/02/turning-off-doclint-in-jdk-8-javadoc.html

It will require to add the following configuration properties to every occurrences of the **maven-javadoc-plugin** artificat in all pom.xml files.
```
<properties>
    <additionalparam>-Xdoclint:none</additionalparam>
</properties>
```
