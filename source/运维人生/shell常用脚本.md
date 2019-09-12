## shell常用脚本

### centos7新装设置IP

ifcfg-eno1根据自己的名称变化，$1第一个参数为ip地址最后一位

使用方法: sh xx.sh 12

```shell

#! /bin/bash

sed -i 's/DHCP/none/g' /etc/sysconfig/network-scripts/ifcfg-eno1
sed -i 's/ONBOOT=no/ONBOOT=yes/g' /etc/sysconfig/network-scripts/ifcfg-eno1

cat>> /etc/sysconfig/network-scripts/ifcfg-eno1 <<EOF
IPADDR="10.255.201.$1"
PREFIX="24"
GATEWAY="10.255.201.1"
DNS1="10.255.201.1"
EOF

```