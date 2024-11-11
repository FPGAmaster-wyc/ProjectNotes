

1、首先查看emmc设备：
	fdisk -l	查询到emmc的名称

2、如果挂载了这个emmc，需要先卸载 （df查询是否挂载）
	卸载emmc：	umount /dev/mmcblk0*（第一步查询到的名称）
	
3、使用fdisk格式划分区
	fdisk /dev/mmcblk0

4、格式划分区选择：
	（1）使用帮助m

​	（2）删除所以分区d		（No partition is defined yet！ 表示删除完毕）

​	（3）创建新的分区n

​	（4）选择分区类型p		（也可以直接回车，默认）

​			Partition type
​			   p   primary partition (1-4) 
​			   e   extended

​	（5）输入分区号
​			Partition number (1-4, default 1): 1 

​	（6）输入分区起始地址
​		First sector (2048-31116287, default 2048): 2048 

​	（7）输入分区结束地址   （如果想创建多个分区，请合理分配地址）
​		Last sector, +sectors or +size{K,M,G} (2048-31116287, default 31116287): 31116287 

​	（8）打印分区情况 p
​	（9）修改分区类型 t		（83为根文件系统分区类型，0C为FAT32）
​	（10）保存并退出 w	
​	

5、记得格式化一下分区
	 mkfs.ext4 /dev/mmcblk0p2 	EXT4
	 mkfs.vfat /dev/mmcblk0p1    	FAT32
	 