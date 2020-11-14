### RAID
##### 部署RAID10

添加4块硬盘设备来制作一个RAID 10磁盘阵列。mdadm中的参数中，-C参数代表创建一个RAID阵列卡；-v参数显示创建的过程，磁盘阵列的名称/dev/md0；-a yes参数代表自动创建设备文件；-n 4参数代表使用4块硬盘来部署这个RAID磁盘阵列；而-l 10参数则代表RAID 10方案；最后再加上4块硬盘设备的名称
```markdown
mdadm -Cv /dev/md0 -a yes -n 4 -l 10 /dev/sdb /dev/sdc  /dev/sdd /dev/sde
mkfs.ext4  /dev/md0
mkdir   /RAID 
mount  /dev/md0  /RAID
df -h                    //磁盘挂载信息
mdadm -D /dev/md0      //查看/dev/md0磁盘阵列的详细信息
echo "/dev/md0 /RAID ext4 defaults 0 0" >> /etc/fstab  
```

------
###### 损坏磁盘阵列及修复
移除阵列中一块硬盘来模拟一块硬盘损坏。
```markdown
 　　mdadm /dev/md0 -f /dev/sdb   　　　#把/dev/sdb从磁盘阵列/dev/md0中移除
　　 mdadm -D /dev/md0         
#查看磁盘这列/dev/md0详细信息，/dev/sdb状态从active变为faulty
 　　umount /RAID                　　　#先重启系统，卸载/RAID目录
 　　mdadm /dev/md0 -a /dev/sdb  　　　#把新硬盘添加到RAID磁盘阵列中
 　　mdadm -D /dev/md0          　　　
#查看磁盘阵列信息，/dev/sdb正在 spare  rebuilding,然后变回active
```


卸载磁盘阵列
```markdown
Umount  /dev/md0
mdadm -S /dev/md0 
```


##### RAID5部署
mdadm:参数-n  3代表创建这个RAID 5磁盘阵列所的硬盘数，参数-l 5代表RAID的级别，而参数-x 1则代表有一块备份盘。
    #mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sd[b-e]

```markdown
mkfs.ext4  /dev/md0   //初始化
echo "/dev/md0  /RAID  ext4 defaults 0 0" >> /etc/fstab    //永久挂载
mkdir /RAID    
mount -a  
mdadm /dev/md0 -f /dev/sdb
```
把硬盘设备/dev/sdb移出磁盘阵列，然后迅速查看/dev/md0磁盘阵列的状态，就会发现备份盘已经被自动顶替上去并开始了数据同步。

---------

------------

### LVM

##### 部署逻辑卷


新添加的两块硬盘设备，并让其支持LVM技术
    # pvcreate /dev/sdb /dev/sdc

把两块硬盘设备加入到storage卷组中
    # vgcreate storage /dev/sdb /dev/

切割出一个约为150MB的逻辑卷设备(两种计量方式：参数-l，是一种计量单位，每个单位4M。参数-L，直接指定容量大小。)
```markdown
# lvcreate -n vo -l 37 storage  
或 # lvcreate -n vo -L 150M storage
```

把生成好的逻辑卷进行格式化，然后挂载使用。
```markdown
# mkfs.ext4 /dev/storage/
# mkdir /linuxprobe 
# mount /dev/storage/vo /linuxprobe
# echo "/dev/storage/vo /linuxprobe ext4 defaults 0 0" >> / etc/fstab
```


##### 扩容逻辑卷
卸载-扩容-检查硬盘-挂载
```markdown
1、取消挂载
# umount /linuxprobe 
2、把逻辑卷vo扩展至300MB。
# lvextend -L 300M /dev/storage/vo  
3、检查硬盘完整性，并重置硬盘容量。
# e2fsck -f /dev/storage/vo 
# resize2fs /dev/storage/vo 
4、重新挂载硬盘设备并查看挂载状态。
# mount -a
#df -h
```
##### 缩小逻辑卷

卸载-检查硬盘-缩容-挂载（丢失数据的风险更大，在生产环境中要提前备份好数据）
```markdown
1、取消挂载
# umount /linuxprobe 
2、检查文件系统的完整性。
# e2fsck -f /dev/storage/vo 
3、把逻辑卷vo的容量减小到120MB。
# resize2fs /dev/storage/vo 120M
# lvextend -L 120M /dev/storage/vo  
4、重新挂载。
# mount -a
#df -h
```


##### 删除逻辑卷
取消挂载，然后依次删除逻辑卷、卷组、物理卷设备
```markdown
1、取消逻辑卷与目录的挂载关联，删除配置文件中永久生效的设备参数。
# umount /linuxprobe 
# vim /etc/fstab   --记得删除配置，要不开机会出错
2、删除逻辑卷设备
# lvremove /dev/storage/vo 
3、删除卷组，只写卷组名称即可
# vgremove storage  
4、删除物理卷设备。
# pvremove /dev/sdb /dev/sdc
```