# kvm常用操作

## 创建虚拟机

```shell

cp swarm03.xml vm-denis-cmdb1.xml

#name修改为 ssd/vm-denis-cmdb1
#cpu内核调整, 内存格式改为GIB(2处), 容量调整, ssdname修改为 ssd/vm-denis-cmdb1
#如不需指定mac地址,则需去掉<mac address='52:54:00:4F:BB:CF'/>, 如需指定则设置成网络中唯一的地址

vi vm-denis-cmdb1.xml

rbd list ssd
rbd snap list ssd/CentOS7Templet
rbd clone ssd/CentOS7Templet@base ssd/vm-denis-cmdb1  #复制快照
rbd resize  ssd/vm-denis-cmdb1 --size 30G   或者  virsh blockresize --domian vm-denis-cmbd1 --path vda --size 30G #调整硬盘大小 
virsh define  vm-denis-cmdb1.xml #从XML文件定义（但不启动）一个虚拟机
virsh start vm-denis-cmdb1

virsh list
virsh dumpxml <id>  #查看用于vnc view连接的端口
```

用vnc viewer连接192.168.252.82:<端口>后查看ip, 即可用xshell连接


## 登录虚拟机后操作

更改默认密码

关闭防火墙

增加公钥

关闭sshd里dns

## 快照操作

```shell
virsh -list -all

 -     vm-denis-k8s1                  shut off

rbd snap create ssd/vm-denis-k8s1@snapshot1   #创建快照(首先关闭虚拟机)
rbd snap ls ssd/vm-denis-k8s1  
virsh start vm-denis-k8s1  #启动虚拟机


rbd snap rollback ssd/vm-denis-k8s1@snapshot1  #恢复快照(关闭虚拟机后操作)
rbd snap rm ssd/vm-denis-k8s1@snapshot1  #删除快照
rbd snap purge ssd/vm-denis-k8s1 #删除多个快照
```

## 其它操作

```shell
virsh destroy <id> #停止虚拟机
virsh undefine vm-denis-cmdb1 #取消虚拟机

virsh create vm-denis-cmdb1.xml #一次性启动

virsh blockresize分情况
- 如果虚拟机是开机状态下，不用调rbd resize，直接调virsh blockresize就行，它会帮忙自动执行rbd resize或qemu-img resize
- 如果虚拟机是关机状态，则不用调该命令，直接调底层的resize，比如rbd就调rbd resize，qcow2就调qemu-img resize

rbd rm  ssd/vm-denis-cmdb1  #删除镜像



```

## 调整虚拟机硬盘大小



```shell
# 虚拟机查看原大小
[root@test ~]# fdisk /dev/vda -l

Disk /dev/vda: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000bc509

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    41943039    20970496   83  Linux


#在82上调整成30G
[root@rxserver3 ~]# rbd resize  ssd/vm-denis-cmdb1 --size 30G
Resizing image: 100% complete...done.

[root@rxserver3 ~]# rbd --image ssd/vm-denis-cmdb1 info
rbd image 'vm-denis-cmdb1':
	size 30 GiB in 7680 objects
	order 22 (4 MiB objects)
	id: 7650c6b8b4567
	block_name_prefix: rbd_data.7650c6b8b4567
	format: 2
	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
	op_features: 
	flags: 
	create_timestamp: Wed Sep  4 17:04:55 2019
	parent: ssd/CentOS7Templet@base
	overlap: 20 GiB


# 虚拟机poweroff后重启, 再次查看大小

[root@localhost ~]# fdisk -l

Disk /dev/vda: 32.2 GB, 32212254720 bytes, 62914560 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000bc509

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    41943039    20970496   83  Linux

#重新分区, 这是在只有一个分区的情况下 

[root@localhost ~]# fdisk /dev/vda 
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): d
Selected partition 1
Partition 1 is deleted

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p  
Partition number (1-4, default 1): 1
First sector (2048-62914559, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-62914559, default 62914559): 
Using default value 62914559
Partition 1 of type Linux and of size 30 GiB is set

Command (m for help): p

Disk /dev/vda: 32.2 GB, 32212254720 bytes, 62914560 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000bc509

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1            2048    62914559    31456256   83  Linux

Command (m for help): w  
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.


# 重启虚拟机后操作
[root@localhost ~]# reboot

[root@localhost ~]# resize2fs /dev/vda1 
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/vda1 is mounted on /; on-line resizing required
old_desc_blocks = 3, new_desc_blocks = 4
The filesystem on /dev/vda1 is now 7864064 blocks long.

[root@localhost ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        30G  1.1G   27G   4% /
devtmpfs        1.9G     0  1.9G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G  8.3M  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
tmpfs           380M     0  380M   0% /run/user/0

```

## 参考文章

https://blog.51cto.com/wutou/1782931

https://blog.51cto.com/speakingbaicai/1161964

https://www.cnblogs.com/chenjiahe/p/5919426.html