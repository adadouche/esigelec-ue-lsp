# Prepare your system for Hadoop & Hive on WSL

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
sudo bash -c 'echo "hadoop ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers'
sudo bash -c 'echo "umask 022" >> ~/.bashrc'
```

## re-mount your C drive

By default your C drive is not mounted properly to allow certain operation, therefore you will need to unmount then mount it back.

In your Ubuntu terminal, execute the following command:

```sh
sudo umount /mnt/c
sudo mount -t drvfs C: /mnt/c -o metadata
```

## Update your Ubuntu distro

Now, you will need to update your Ubuntu system with the latest updates using the following command:

```sh
sudo apt-get update
```

## Install the openJDK8

Then, you will need to install the openJDK on your Ubuntu system using the following command:

```sh
sudo apt-get install openjdk-8-jdk
```

> ### **Note**:
> This step takes about 10 minutes.

The JDK will be available at:

 - /usr/lib/jvm/java-8-openjdk-amd64/

## Install SSH

Now, you need to install ssh on your Ubuntu system using the following command:

```sh
sudo apt-get install ssh
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
rm -r .ssh

ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa -q
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

Then connect to your localhost:

```sh
ssh localhost
```

Once connected, you can then exit the ssh and the bash session by executing:

```sh
exit
```
