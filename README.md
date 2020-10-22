# Create-HadoopHA
搭建Hadoop集群用于大数据项目学习

一、安装三台虚拟机
1、centos7
2、VMware12

二、配置网络环境
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

3、关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
