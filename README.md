# Create-HadoopHA
搭建Hadoop集群用于大数据项目学习

#一、安装三台虚拟机

1、centos7

2、VMware12

3、最小化安装
  
  好处：1、系统干净  2、消耗资源少
 
4、详细过程略


#二、配置网络环境

1、设置为静态IP

第一台Master

IPADDR=192.168.138.101

NETMASK=255.255.255.0

GATEWAY=192.168.138.2

DNS1=192.168.138.2

第二台node1

IPADDR=192.168.138.102

NETMASK=255.255.255.0

GATEWAY=192.168.138.2

DNS1=192.168.138.2

第三台node2

IPADDR=192.168.138.103

NETMASK=255.255.255.0

GATEWAY=192.168.138.2

DNS1=192.168.138.2

2、添加IP地址和主机名映射

192.168.138.101  master

192.168.138.102  node1

192.168.138.103  node2

注意：3、4、5、6、7步骤所有节点都要操作

3、关闭防火墙

systemctl stop firewalld

systemctl disable firewalld

4、关闭NetWorkManager

systemctl stop NetworkManager

systemctl disable NetworkManager

5、关闭SELinux服务

vi /etc/sysconfig/selinux

SELINUX=enforcing

SELINUX=disabled

6、配置时间同步

查看是否系统有chrony

rpm -qa|grep chrony  (如果没有，请挂着centos镜像文件进行安装)

主节点

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

node1、node2操作（与maser的时间同步）

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


7、配置免密登录

ssh-keygen -t rsa

ssh-copy-id -i ~/.ssh/id_rsa.pub 目标用户@目标主机名或IP地址

ssh　目标主机名

#三、安装jdk(使用的jdk为1.8)

每一台节点都要安装jdk

master节点

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

将安装好的java复制到node1、node2

scp -r /opt/java/java1.8/ root@node1:/opt/java/

scp /etc/profile root@node1:/etc/

scp -r /opt/java/java1.8/ root@node2:/opt/java/

scp /etc/profile root@node2:/etc/

在node1、node2执行source /etc/profile

