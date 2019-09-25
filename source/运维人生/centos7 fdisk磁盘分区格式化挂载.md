# centos7 fdisk磁盘分区格式化挂载


以 /dev/sdb 分一个区为例

```shell

fdisk -l  #查看没有分区的硬盘

fdisk /dev/sdb
n, 建立分区
p, 建立主分区
回车(使用默认分区号)
回车(使用默认起始扇区)
回车(使用默认结束扇区,如需指定分区大小可输入如 +20G), 
w, 保存

partprobe #重读分区表

mkfs -t xfs /dev/sdb1  #格式化

#临时挂载
mkdir /mnt/data
mount  /dev/sdb1 /mnt/data  

#修改/etc/fstab, 使分区自动挂载
vim /etc/fstab 添加
/dev/sdb1 /mnt/data     xfs     defaults    0 0 

#也可以使用lsblk -f 或者 blkid /dev/sdb1 查看磁盘的uuid来挂载
UUID=f502fa32-96bf-48e8-bdf0-166c1e74f8fa   /mnt/data     xfs     defaults    0 0 

mount -a

```