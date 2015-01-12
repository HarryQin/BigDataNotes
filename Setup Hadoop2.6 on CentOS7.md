Set up Hadoop 2.6 on CentOS 7
=============================
#Install CentOS 7 (core)
#JDK & Java Environment Variable
```shell
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u25-b17/jdk-8u25-linux-x64.rpm
```
```shell
yum localinstall jdk.rpm
```
```shell
vi /etc/profile
```
```txt
JAVA_HOME=/usr/java/jdk1.8.0_25
JRE_HOME=/usr/java/jdk1.8.0_25/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

#Add a User & SSH

```shell
useradd hadoop
passwd hadoop
su - hadoop
```
```shell
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@data2
ssh-copy-id -i ~/.ssh/id_rsa.pub hadoop@data3
```
#Add Host Entries
```shell
vi /etc/hosts
```
write the following entries, for example
```txt
10.10.1.1 router
10.10.1.2 master
10.10.1.3 data2
10.10.1.4 data3
```

#Hadoop 2.6 & Environment Variable

```shell
wget http://*/hadoop-2.6.0.tar.gz
```
```shell
tar xzf hadoop-2.6.0.tar.gz
```
```shell
vi ~/.bashrc
```
```txt
export HADOOP_HOME=/home/hadoop/hadoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```
edit the $HADOOP_HOME/etc/hadoop/hadoop-env.sh and set the JAVA_HOME directory accordingly.Put the property info below between the “configuration” tags for each file tags for each file
Edit $HADOOP_HOME/etc/hadoop/core-site.xml
```xml
<property>
  <name>fs.default.name</name>
    <value>hdfs://localhost:9000</value>
</property>
### Edit $HADOOP_HOME/etc/hadoop/hdfs-site.xml
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
```
copy template
```shell
$ cp $HADOOP_HOME/etc/hadoop/mapred-site.xml.template $HADOOP_HOME/etc/hadoop/mapred-site.xml
```
Edit $HADOOP_HOME/etc/hadoop/mapred-site.xml
```xml
<property>
  <name>mapreduce.framework.name</name>
   <value>yarn</value>
</property>
```
### Edit $HADOOP_HOME/etc/hadoop/yarn-site.xml
```xml
<property>
  <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```
#Start Hadoop
format namenode to keep the metadata related to datanodes

```shell
hdfs namenode -format
```
run start-dfs.sh script
```shell
./start-dfs.sh
```
check that HDFS is running
check there are 3 java processes:
namenode
secondarynamenode
datanode

<pre><code>./start-yarn.sh</pre></code>
check there are 2 more java processes:
resourcemananger
nodemanager

#Test Hadoop

access hadoop via the browser on port 50070

put a file
```shell
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/hadoop
hdfs dfs -put /var/log/boot.log
```
check in your browser if the file is available

#allow the port
```shell
firewall-cmd --zone=public --add-port=8088/tcp --permanent
firewall-cmd --reload
```
