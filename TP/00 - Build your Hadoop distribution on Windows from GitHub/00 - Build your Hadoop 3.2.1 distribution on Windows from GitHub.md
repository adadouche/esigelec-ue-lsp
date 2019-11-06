# Build your Hadoop 3.2.1 distribution on Windows from GitHub

## Prerequisites

You will need to download and install the following software on your Windows system (Windows 10 x64):

- Git: https://git-scm.com/downloads (x64 executable)

- Java JDK 1.8 : https://www.oracle.com/technetwork/java/javase/downloads/index.html (executable [jdk-8u221-windows-x64.exe](https://download.oracle.com/otn/java/jdk/8u221-b11/230deb18db3e4014bb8e3e8324f81b43/jdk-8u221-windows-x64.exe))

- Maven : https://maven.apache.org/download.cgi (Zip archive [apache-maven-3.6.2-bin.zip](http://mirrors.standaloneinstaller.com/apache/maven/maven-3/3.6.2/binaries/apache-maven-3.6.2-bin.zip))

- Protocol Buffers v2.5.0: https://github.com/protocolbuffers/protobuf/releases/tag/v2.5.0 (Zip archive [protoc-2.5.0-win32.zip](https://github.com/protocolbuffers/protobuf/releases/download/v2.5.0/protoc-2.5.0-win32.zip))

- CygWin x64: https://www.cygwin.com/ (executable [setup-x86_64.exe](https://www.cygwin.com/setup-x86_64.exe))

  - select:
    - doxygen & doxygen-doxywizard
    - dos2unix

  - avoid installing cMake and other packages from CygWin as it will collide with Visual Studio

- Visual Studio Community 2019 & Build Tools for Visual Studio 2019: https://visualstudio.microsoft.com/vs/community/

  - You will need to install the following packages:

    - .NET desktop development
    - Desktop development with C++

> ### Note:
> - Make sure that **MSVC v140 - VS 2015 C++ build tools** is selected in the items to install under **Desktop development with C++**
> - It is not recommended to have Windows Subsystem for Linux install

## Clone the Hadoop Git Repository

Open a "x64_x86 Native Tools Command Prompt for VS 2019" (from the Start menu) and paste the following commands (that you can adjust if needed):

```sh
mkdir C:\MyWork
cd C:\MyWork
git clone https://github.com/apache/hadoop.git C:\MyWork\hadoop-git-latest

cd C:\MyWork\hadoop-git-latest
git checkout --track origin/branch-3.2.1
```

This will clone the Git repository on your machine under a folder named **C:\MyWork\hadoop-git-latest**

## Install VCPkg

```sh
mkdir C:\MyWork
cd C:\MyWork
git clone https://github.com/Microsoft/vcpkg.git

cd C:\MyWork\vcpkg

set PATH=%PATH%;C:\MyWork\vcpkg

vcpkg install zlib:x64-windows
vcpkg install openssl:x64-windows

vcpkg integrate install
```

## Set the target environment in the POM

Run the following command:

```sh
powershell -Command "(gc C:\MyWork\hadoop-git-latest\hadoop-hdfs-project\hadoop-hdfs\pom.xml) -replace 'Visual Studio 10 Win64', 'Visual Studio 16 2019' | Out-File -encoding ASCII C:\MyWork\hadoop-git-latest\hadoop-hdfs-project\hadoop-hdfs\pom.xml"
```

## Set the shell executable to sh

Run the following command:

```sh
powershell -Command "(gc C:\MyWork\hadoop-git-latest\pom.xml) -replace '<shell-executable>bash</shell-executable>', '<shell-executable>sh</shell-executable>' | Out-File -encoding ASCII C:\MyWork\hadoop-git-latest\pom.xml"
```

-- ## Convert to unix the native libs script

Run the following command:

```sh
set PATH=%PATH%;C:\cygwin64\bin

sh -c "dos2unix /cygdrive/c/MyWork/hadoop-git-latest/dev-support/bin/dist-copynativelibs"
sh -c "dos2unix /cygdrive/c/MyTools/protoc-3.10.1-win64/include/google/protobuf/compiler/*"
```

## Set the Maven repository for a shorter path

Create a file named:

```sh
C:\MyWork\settings.xml
```

with the following content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>C:/MyWork/.m2</localRepository>
</settings>
```

## Build Hadoop from the source

Close you previous "x64 Native Tools Command Prompt for VS 2019" command prompt and open a new one (to avoid path being too long).

Then, you can execute the following commands:

```sh
set Platform=x64
set M2_HOME=C:\MyTools\apache-maven-3.6.2
set JAVA_HOME=C:\Progra~1\Java\jdk1.8.0_221
set PROTOBUF_ROOT_DIR=C:/MyTools/protoc-2.5.0-win32

set PATH=%PATH%;C:\cygwin64\bin
set PATH=%PATH%;%JAVA_HOME%\bin
set PATH=%PATH%;%M2_HOME%\bin
set PATH=%PATH%;%PROTOBUF_ROOT_DIR%

set OPENSSL_ROOT_DIR=C:/MyWork/vcpkg/packages/openssl-windows_x64-windows
set OPENSSL_INCLUDE_DIR=%OPENSSL_ROOT_DIR%/include

cd C:\MyWork\hadoop-git-latest

mvn clean
mvn package -gs C:/MyWork/settings.xml -Pdist,native-win -DskipTests -Dtar -Dmaven.javadoc.skip=true -rf :hadoop-hdfs-native-client

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
C:\MyWork\hadoop-git-latest\hadoop-dist\target\hadoop-3.2.1-SNAPSHOT.tar.gz
```

Extract the tar ball using the following commands:

```sh
cp C:\MyWork\hadoop-git-latest\hadoop-dist\target\hadoop-3.2.1-SNAPSHOT.tar.gz cd C:\MyWork\hadoop-3.2.1.tar.gz

cd cd C:\MyWork
tar -xvf C:\MyWork\hadoop-3.2.1.tar.gz
```

## Extra

### Reset your Hadoop Git repository

In case, you want to reset your local copy of the Hadoop Git repo, you can run the following series of commands:

```
cd C:\MyWork\hadoop-git-latest

git reset --hard
git clean -f -d
git pull -f
```
