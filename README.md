#Installation of hadoop on centos 7

## Install hadoop 3.1 on centos 7

- Install JAVA(we will be installing openjdk 8 (JDK 8+ are compitable with Hadoop-3.x)
```
yum install openjdk-8-jdk
```
- Check the desired java version is installed
```
$ java -version
```

> TODO: If above command shows some different version of Java then we have installed then you will need to perform additional steps to make it work

- Create Dedicated Hadoop user
Hadoop daemons can be run from root users also, but to isolate the hadoop daemons from other users process we will configure saperate user for hadoop. Assuming usename as hadoop
```
$ adduser hadoop
$ passwd hadoop
```

- Enable passwordless login from dedicated user to self
To enable start/stop of hadoop daemons without human intervention, we should enable keybased ssh authentication on all the hadoop node (for user created earlier). Since we are having a single node we just need to enable keybased authentication for own node
```
$ su - hadoop
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa   #Press enter whenever prompted
$ ssh-copy-id localhost
```

Verify keybased login is working (below command should not prompt for any password) 
```
$ ssh localhost
$ exit
```

- Hadoop 3.1 Installation
```
switch to root user and execute 
$ wget https://archive.apache.org/dist/hadoop/core/hadoop-3.1.0/hadoop-3.1.0.tar.gz
$ tar xzf hadoop-3.1.0.tar.gz -C /opt/
$ cd /opt ; mv hadoop-3.1.0 hadoop
$ chown -R hadoop:hadoop /opt/hadoop
```

- Setup Hadoop Environment Variables
Switch to hadoop user and execute
```
$ cat <<EOF >> ~/.bashrc
export HADOOP_HOME=/opt/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_PID_DIR=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
EOF

$ source ~/.bashrc
$ sed -i -e 's/#\ export\ JAVA_HOME=/export\ JAVA_HOME=\/opt\/jdk1.8.0_171/g' $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

- Setup Hadoop Configuration Files
```
$ cat <<EOF > $HADOOP_HOME/etc/hadoop/core-site.xml
<configuration>
<property>
  <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
</property>
</configuration>
EOF

$ cat <<EOF > $HADOOP_HOME/etc/hadoop/hdfs-site.xml
<configuration>
<property>
 <name>dfs.replication</name>
 <value>1</value>
</property>

<property>
  <name>dfs.name.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
</property>

<property>
  <name>dfs.data.dir</name>
    <value>file:///home/hadoop/hadoopdata/hdfs/datanode</value>
</property>
</configuration>
EOF
```
- Format Namenode
```
$ hdfs namenode -format
```

- Start Hadoop Cluster
```
$ start-dfs.sh
```

> Note : Hadoop NameNode started on port 9870 default. Access your server on port 9870 in your favorite web browser.
> Note : Access port 9864 to get details about your Hadoop node.


- Setting up Hadoop to run at boot
1. Copy hadoop-dfs.system file in /etc/init.d/
2. rename the file to hadoop-dfs
```	$ mv /etc/init.d/hadoop-dfs-init_d /etc/init.d/hadoop-dfs ```
3. Add executable permission to script
```     $ chmod +x /etc/init.d/ ```
4. Add the script to run at boot
```	$ chkconfig hadoop-dfs on ```
5. Vertify that the hadoop script is installed fine.
```     $ service hadoop-dfs status 
    $ service hadoop-dfs stop
    $ service hadoop-dfs start 
```
