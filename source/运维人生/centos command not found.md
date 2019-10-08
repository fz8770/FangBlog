# centos command not found

## command not found 解决办法

当不知道某个命令是哪个包装时，可以在已经有这个命令的主机上用下面的命令确定是哪个安装包安装的

yum whatprovides 命令路径或者命令的绝对路径

```shell

[root@db14 ~]# yum whatprovides /usr/sbin/ss
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * elrepo: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
iproute-4.11.0-14.el7.x86_64 : Advanced IP routing and network device configuration tools
源    ：base
匹配来源：
文件名    ：/usr/sbin/ss



iproute-4.11.0-14.el7_6.2.x86_64 : Advanced IP routing and network device configuration tools
源    ：updates
匹配来源：
文件名    ：/usr/sbin/ss


```

## 常见command not found

```shell

ss:bash:command not found

yum install iproute -y

ifconfig:bash:command not found

yum install net-tools -y

vim:bash:command not found

yum install vim -y

sar:bash:command not found

yum install sysstat

brctl: command not found

yum install bridge-utils -y

arp: command not found
net-tools-2.0-0.24.20131004git.el7.x86_64 : Basic networking tools

```