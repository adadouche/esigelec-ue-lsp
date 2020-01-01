# Prepare your system for WSL

## Goal

In order to reduce the resource footprint on your local machines (below 8GB of RAM), we will be using Windows Subsystem For Linux which provides a true Linux kernel since version 2.0 and enable more less anything you could do on a Linux Virtual Machine.

The main steps to install WSL on you Windows 10 machine are:

  - Enable the Windows Subsystems for Linux feature using PowerShell
  - Install the Ubuntu environment
  - Create a dedicated Ubuntu user (hadoop)
  - Add the hadoop to the ***sudoers***
  - Install & Configure the following packages:
  - openJDK 8
  - Install 7zip

## Install Windows Subsystems for Linux

If you don't have it already installed, you will Windows Subsystems for Linux installed on your Windows computer.

You can follow the instructions available at:

 - https://docs.microsoft.com/en-us/windows/wsl/install-win10

Then, make sure to install the Ubuntu 16.04 LTS distribution:

 - https://www.microsoft.com/fr-fr/p/ubuntu-1604-lts/9pjn388hp8c9?rtc=1&activetab=pivot:overviewtab

> ### **Note**:
> The download is about 200MB.

Make sure to complete [Initializing a newly installed distro](https://docs.microsoft.com/en-us/windows/wsl/initialize-distro).

You can choose a new user name like ***hadoop*** and the associated passowrd.

Once completed, you can open a new command prompt (cmd) and execute ***bash*** or simply find the Ubuntu entry in the Start menu.

Open an Ubuntu prompt where the command prompt should look like:

```sh
hadoop@ABC:~$
```

Where XYZ represents your user name and ABC your machine name.

## Add your user to the sudoers

Before getting started, let's add your new user to the sudoers and prevent password to be needed when using sudo.

In your Ubuntu terminal, execute the following commands:

```sh
export LINE="$USER ALL=(ALL) NOPASSWD: ALL"
sudo grep -qF "$LINE" /etc/sudoers || sudo USER=$USER bash -c 'echo "$USER ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers'
```

**Make sure to adjust the user if yu choose not to use hadoop as your username.**

## Re-mount your C drive

By default your C drive is not mounted properly to allow certain operations, therefore you will need to unmount then mount it back.

In your Ubuntu terminal, execute the following command:

```sh
sudo umount /mnt/c
sudo mount -t drvfs C: /mnt/c -o metadata
```

**You might need to redo this operation if you hit errors  on folder permissions.**

## Update your Ubuntu distro

Now, you will need to update your Ubuntu system with the latest updates using the following command:

```sh
sudo apt-get -y update
```

## Install the openJDK 8

Then, you will need to install the openJDK on your Ubuntu system using the following command:

```sh
sudo apt-get -y install openjdk-8-jdk
```

> ### **Note**:
> This step takes about 10 minutes.

The JDK will be available at:

 - /usr/lib/jvm/java-8-openjdk-amd64/


## Install the 7zip

Then, you will need to install 7zip on your Ubuntu system using the following command:

```sh
sudo apt-get -y install p7zip-full
```

## Install Python

In your **Ubuntu** terminal, execute:

```sh
sudo apt-get -y install python
```

## Install SSH

Now, you need to install ssh on your Ubuntu system using the following command:

```sh
sudo apt-get -y install ssh
```

After the ssh install completes, you will need to stop the service:

```sh
sudo service ssh stop
```

Then generate the root key:

```sh
sudo /usr/bin/ssh-keygen -A
```

Then restart the service:

```sh
sudo service ssh restart
```

Create the user key:

```sh
rm -rf ~/.ssh

ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa -q
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys

ssh-keyscan -t rsa -H localhost >> ~/.ssh/known_hosts
ssh-keyscan -t rsa -H 127.0.0.1 >> ~/.ssh/known_hosts
```

Then connect to your localhost:

```sh
ssh localhost 'exit'
```
## Assign a host name / ip automagically

Now, you need to get the host ip address and assign a name like hadoop-host to ease the use of web addresses.

Execute the following command:

```sh
export ENV_FILE=~/esigelec-ue-lsp-hdp/.set_sys_env.sh

rm -f $ENV_FILE
echo -e "export host_name=\$(hostname)"                                                          >> $ENV_FILE
echo -e "export host_fqdn=\$(hostname -A)"                                                       >> $ENV_FILE
echo -e "export host_ipv4=\$(hostname -I)"                                                       >> $ENV_FILE
echo -e "export host_ipv6=\$(ip addr show dev eth0 | sed -e's/^.*inet6 \([^ ]*\)\/.*$/\1/;t;d')" >> $ENV_FILE

echo -e "sudo cp -n /etc/hosts          /etc/hosts.original" >> $ENV_FILE
echo -e "sudo cp    /etc/hosts.original /etc/hosts         " >> $ENV_FILE
echo -e "sudo sed -i -e \"s|127.0.0.1|# 127.0.0.1|g\" /etc/hosts" >> $ENV_FILE
echo -e "sudo sed -i -e \"s|127.0.1.1|# 127.0.1.1|g\" /etc/hosts" >> $ENV_FILE
echo -e "sudo sed -i -e \"s|::1|# ::1|g\"             /etc/hosts" >> $ENV_FILE

echo -e "sudo bash -c 'echo \"${host_ipv4} ${host_name} ${host_fqdn} localhost\" >> /etc/hosts'" >> $ENV_FILE

grep -qF "source $ENV_FILE" ~/.bashrc || echo -e "source $ENV_FILE" >> ~/.bashrc

source $ENV_FILE
```
