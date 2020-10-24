# Create-HadoopHA
搭建Hadoop集群用于大数据项目学习

## 一、安装三台虚拟机

### 1、centos7

### 2、VMware12

### 3、最小化安装
  
  好处：1、系统干净  2、消耗资源少
 
### 4、详细过程略


## 二、配置网络环境

### 1、设置为静态IP

#### 第一台Master

IPADDR=192.168.138.101

NETMASK=255.255.255.0

GATEWAY=192.168.138.2

DNS1=192.168.138.2

#### 第二台node1

IPADDR=192.168.138.102

NETMASK=255.255.255.0

GATEWAY=192.168.138.2

DNS1=192.168.138.2

#### 第三台node2

IPADDR=192.168.138.103

NETMASK=255.255.255.0

GATEWAY=192.168.138.2

DNS1=192.168.138.2

### 2、添加IP地址和主机名映射

192.168.138.101  master

192.168.138.102  node1

192.168.138.103  node2

注意：3、4、5、6、7步骤所有节点都要操作

### 3、关闭防火墙

systemctl stop firewalld

systemctl disable firewalld

### 4、关闭NetWorkManager

systemctl stop NetworkManager

systemctl disable NetworkManager

### 5、关闭SELinux服务

vi /etc/sysconfig/selinux

SELINUX=enforcing

SELINUX=disabled

### 6、配置时间同步

查看是否系统有chrony

rpm -qa|grep chrony  (如果没有，请挂着centos镜像文件进行安装)

#### 主节点

vi /etc/chrony.conf

注释

\#server 0.centos.pool.ntp.org iburst

\#server 1.centos.pool.ntp.org iburst

\#server 2.centos.pool.ntp.org iburst

\#server 3.centos.pool.ntp.org iburst

添加

server master iburst

修改

\# Allow NTP client access from local network.

allow 192.168.138.0/24

修改

\# Serve time even if not synchronized to a time source.

local stratum 10

systemctl enable chronyd.service

systemctl start chronyd.service

timedatectl　set-timezone　Asia/Shanghai

#### node1、node2操作（与maser的时间同步）

vi /etc/chrony.conf

注释

\#server 0.centos.pool.ntp.org iburst

\#server 1.centos.pool.ntp.org iburst

\#server 2.centos.pool.ntp.org iburst

\#server 3.centos.pool.ntp.org iburst

添加
server master iburst

systemctl enable chronyd.service

systemctl start chronyd.service


### 7、配置免密登录

ssh-keygen -t rsa

ssh-copy-id -i ~/.ssh/id_rsa.pub 目标用户@目标主机名或IP地址

ssh　目标主机名

## 三、安装jdk(使用的jdk为1.8)

每一台节点都要安装jdk

#### master节点

上传jdk-8u131-linux-x64.tar.gz到/opt

tar -xzvf jdk-8u131-linux-x64.tar.gz

mkdir /opt/java

mv /opt/jdk1.8.0_131/ /opt/java/java1.8

配置java的环境变量

vi /etc/profile

#set java environment

JAVA_HOME=/opt/java/java1.8

CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

PATH=$JAVA_HOME/bin:$PATH

export JAVA_HOME CLASSPATH PATH

source /etc/profile

验证java

java -version

#### 将master安装好的java复制到node1、node2

scp -r /opt/java/java1.8/ root@node1:/opt/java/

scp /etc/profile root@node1:/etc/

scp -r /opt/java/java1.8/ root@node2:/opt/java/

scp /etc/profile root@node2:/etc/

在node1、node2执行source /etc/profile

### 四、安装Hadoop(使用的是Hadoop2.7)

Hadoop安装过程和jdk流程一致

上传hadoop-2.7.3.tar.gz到/opt

tar -xzvf hadoop-2.7.3.tar.gz

mv hadoop-2.7.3 hadoop/hadoop2.7

#### 配置Hadoop环境变量

vi /etc/profile

\#set hadoop environment

HADOOP_HOME=/opt/hadoop/hadoop2.7

PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

export HADOOP_HOME PATH

#### 编辑Hadoop的配置文件

进入Hadoop配置文件夹

cd /opt/hadoop/hadoop2.7/etc/hadoop

##### 修改 core-site.xml

vim core-site.xml

添加配置参数

\<property>
  
  \<name>fs.default.name\</name>
  
  \<value>hdfs://master:9000\</value>
 
 \</property>
 
 \<property>
 
 \<name>hadoop.tmp.dir\</name>
 
 \<value>/opt/hadoop/tmp\</value>
 
 \</property>
 
 \<property>
 
 \<name>io.file.buffer.size\</name>
 
 \<value>131702\</value>
 
 \</property>

##### 修改 hadoop-env.sh

\# The java implementation to use.

export JAVA_HOME=/opt/java/java1.8

##### 修改 hdfs-site.xml

\<!-- namenode数据的存放地点。也就是namenode元数据存放的地方，记录了hdfs系统中文件的元数据-->

\<property>

\<name>dfs.namenode.name.dir\</name>

\<value>/opt/hadoop/name\</value>

\</property>

\<!-- datanode数据的存放地点。也就是block块存放的目录了-->

\<property>

\<name>dfs.datanode.data.dir\</name>

\<value>/opt/hadoop/data\</value>

\</property>

 \<!-- hdfs的副本数设置。也就是上传一个文件，其分割为block块后，每个block的冗余副本个数-->

\<property>

\<name>dfs.replication\</name>

\<value>2\</value>

\</property>

 \<!-- secondary namenode的http通讯地址-->

\<property>

\<name>dfs.secondary.http.address\</name>

\<value>master:50090\</value>

\</property>

 \<property>

\<!-- 开启hdfs的web访问接口。默认端口是50070 , 一般不配 , 使用默认值-->

\<name>dfs.webhdfs.enabled\</name>

\<value>true\</value>

\</property>

##### 修改mapred-site.xml

复制mapred-site.xml.template文件并重命名为mapred-site.xml

\<property>

\<!-- 指定mr框架为yarn方式,Hadoop二代MP也基于资源管理系统Yarn来运行 -->

\<name>mapreduce.framework.name\</name>

\<value>yarn\</value>

\</property>

 \<!-- JobHistory Server ============================================================== -->

\<!-- 配置 MapReduce JobHistory Server 地址 ，默认端口10020 -->

\<property>

\<name>mapreduce.jobhistory.address\</name>

\<value>master:10020\</value>

\</property>

\<!-- 配置 MapReduce JobHistory Server web ui 地址， 默认端口19888 -->

\<property>

\<name>mapreduce.jobhistory.webapp.address\</name>

\<value>node1:19888\</value>

\</property>

##### 修改yarn-site.xml文件

\<property>

\<name>yarn.resourcemanager.hostname\</name>

\<value>master\</value>

\</property>

 \<property>

\<name>yarn.resourcemanager.address\</name>

\<value>${yarn.resourcemanager.hostname}:8032\</value>

\</property>

 \<property>

\<name>yarn.resourcemanager.scheduler.address\</name>

\<value>${yarn.resourcemanager.hostname}:8030\</value>

\</property>

 \<property>

\<name>yarn.resourcemanager.webapp.address\</name>

\<value>${yarn.resourcemanager.hostname}:8088\</value>

\</property>

 \<property>

\<description>The https adddress of the RM web application.\</description>

\<name>yarn.resourcemanager.webapp.https.address\</name>

\<value>${yarn.resourcemanager.hostname}:8090\</value>

\</property>

 \<property>
 
 \<name>yarn.resourcemanager.resource-tracker.address\</name>
 
 \<value>${yarn.resourcemanager.hostname}:8031\</value>
 
 \</property>

 \<property>
 
 \<name>yarn.resourcemanager.admin.address\</name>
 
 \<value>${yarn.resourcemanager.hostname}:8033\</value>
 
 \</property>

 \<property>
 
 \<name>yarn.nodemanager.aux-services\</name>
 
 \<value>mapreduce_shuffle\</value>
 
 \</property>

 \<property>
 
 \<name>yarn.scheduler.maximum-allocation-mb\</name>
 
 \<value>2048\</value>
 
 \</property>

 \<property>
 
 \<name>yarn.nodemanager.vmem-pmem-ratio\</name>
 
 \<value>2.1\</value>
 
 \</property>

 \<property>

\<name>yarn.nodemanager.resource.memory-mb\</name>

\<value>2048\</value>

\</property>

 \<property>

\<name>yarn.nodemanager.vmem-check-enabled\</name>

\<value>false\</value>

\</property>

##### 修改slaves

node1

node2

##### 同步到node1、node2

scp -r /opt/hadoop/hadoop2.7/ root@node2:/opt/hadoop

scp -r /opt/hadoop/hadoop2.7/ root@node1:/opt/hadoop
   
scp -r /opt/hadoop/hadoop2.7/ root@node2:/opt/hadoop

scp -r /opt/hadoop/hadoop2.7/ root@node1:/opt/hadoop

node1、node2执行source /etc/profile

##### 在master /opt/hadoop 创建 name data tmp 三个文件夹

### 到此Hadoop就安装完成，开始验证

master 执行hadoop namenode -format 初始化

执行start-dfs.sh 、 start-yarn.sh 启动hadoop(也可以使用start-all.sh)

在各个节点执行jps查看服务是否启动

停止Hadoop 执行 stop-all.sh

# 其他组件的安装，在其他文件讲解
