# petalinux设计流程：

1、搭建vivado，导出hdf文件（带文件夹传到linux中）

2、创建petalinux

3、将hdf文件导入到petalinux

4、配置petalinux工程

5、编译petalinux工程

6、启动开发板

# petalinux实战：

```shell
petalinux-boot 		##启动开发板

petalinux-build 	##编译

petalinux-config 	##配置

petalinux-create 	##创建

petalinux-package 	##打包

petalinux-build -x mrproper -f	## 清除生成的文件

## 首先source setting.sh，对petalinux环境进行配置（每次打开一个新的终端都需要配置）
```

# petalinux流程例子：

## 	1、创建petalinux工程

```shell
## 创建一个名字为LED-ZYNQ的，ZYNQ模板的工程
petalinux-create -t project - -template zynq -n LED-ZYNQ （通过hdf文件 模板zynqMP）  

##（通过bsp）
petalinux-create -t project -s avnet-digilent-zedboard-v2018.3-final.bsp -n my_zed
```



## 	2、导入hdf文件，并配置petalinux（需要进入工程目录）

```shell
## （.xsa文件路径的上级目录）导入.xsa文件，.xsa文件位置在xsa文件夹里面
petalinux-config - -get-hw-description /home/hwusr/test/

## 如果想单独打开配置petalinux界面
petalinux-config
```



## 	3、配置uboot

```shell
petalinux-config -c u-boot
```



## 	4、配置kernel（内核）

```shell
petalinux-config -c kernel
```



## 	5、配置rootfs（根文件系统）

```shell
petalinux-config -c rootfs
```



## 	6、编译工程

（1）编译整个petalinux工程

```shell
petalinux-build
```

（2）单独编译（u-boot、kernel、rootfs）

```shell
## 编译BOOT
petalinux-build -c bootloader

## （编译根文件）
petalinux-build -c rootfs 
```

**注：**如何一直卡着不动，很有可能是bitbake配置失败，解决办法：删除build文件夹



## 	7、制作启动镜像BOOT.bin文件（进入image文件夹）

petalinux-package - -boot - -fsbl - -fpga - -u-boot - -force

​	\- -boot：通过package命令生成BOOT.bin文件

​	\- -fsbl： 指定fsbl镜像文件路径 zynq_fsbl.elf

​	\- -fpga: 指定bit文件路径 system.bit

​	\- -u-boot：指定u-boot文件路径（用户程序路径） u-boot.elf

​	\- -force：强制覆盖（如果当前路径下面有.bin文件，则进行覆盖）

```shell
## zynq
petalinux-package --boot --fsbl ./zynq_fsbl.elf --fpga ./system.bit --u-boot ./u-boot.elf --force
petalinux-package --boot --fsbl --fpga --u-boot --force

## zynqMP
petalinux-package --boot --format BIN --fsbl --u-boot --pmufw --fpga --force
petalinux-package --boot --format BIN --fsbl ./zynqmp_fsbl.elf --u-boot ./u-boot.elf --pmufw ./pmufw.elf --fpga system.bit --force
```



**FLASH启动的时候**：把内核和boot.src如果都放到BOOT里面的话，需要对地址进行分配

```shell
petalinux-package --boot --force --format BIN --fsbl --fpga --pmufw --u-boot --kernel images/linux/Image --offset 0x1940000 --cpu a53-0 --boot-script --offset 0x3240000
```

**FLASH启动ramdisk**：（即根文件系统也在FLASH）（需要修改根文件位置为INRD并配置地址）

```shell
petalinux-package --boot --force --format BIN --fsbl --pmufw --u-boot --kernel images/linux/Image --offset 0x240000 --cpu a53-0 --boot-script --offset 0x1740000 --add images/linux/rootfs.cpio.gz.u-boot --offset 0x1780000 --cpu a53-0 --file-attribute partition_owner=uboot
```



## 	8、制作SD启动卡

**2019.2版本**

​	**第一种：只需分一个分区FAT32**

​		对于ZYNQ硬件平台来说，启动嵌入式Linux系统需要这些文件

​		1、BOOT.bin（fsbl镜像文件、bit流文件、u-boot文件）

​		2、image.ub（kernel、设备树、rootfs）或者（uImage + system.dtb）

​		将镜像文件（BOOT.bin、image.ub）拷贝到SD卡的FAT32分区，插入开发板启动

**第二种：把SD卡分为两个分区FAT32和ext4，需要用linux系统操作 **（zynqMP）

​		1、把BOOT.bin、Image、system.dtb文件放到FAT32分区

​		2、把rootfs.tar.gz压缩包解压缩到ext4分区

**2021.2版本**

​	**把SD卡分为两个分区FAT32和ext4，需要用linux系统操作**

​		1、把BOOT.BIN, boot.scr, Image, and ramdisk.cpio.gz.u-boot(MicroBlaze才需要) 放到FAT32分区	（zynqMP）

​			如果是zynq系列的话，需要把Image换成uImage，image.ub

​		2、把rootfs.tar.gz压缩包解压缩到ext4分区

​			$ sudo tar xvf rootfs.tar.gz -C /media/rootfs

如果跟换debian根文件系统，记得把lib 下的module文件夹替换为petalinux的才可以运行



## 	9、创建APP应用、驱动

```shell
petalinux-creat -t apps --template c -n first-app --enable （添加APP程序）

petalinux-creat -t modules -n first-modules --enable （添加驱动程序）
```

# petalinux生产的系统设置固态网络IP：

**1、设置静态IP**

1.编辑`/etc/network/interfaces`文件

```shell
sudo vim /etc/network/interfaces
```

2.注释掉`iface eth0 inet dhcp`这行，注意要和电脑主机保持同一网段，比如我的电脑主机网口IP地址是：`192.168.137.1`，子网掩码是`255.255.255.0`，修改对应网口的配置如下

```shell
auto eth0
iface eth0 inet static
address 192.168.137.10
gateway 192.168.137.1
netmask 255.255.255.0
# iface eth0 inet dhcp
```

3.置DNS

```shell
mkdir -p /etc/resolvconf/resolv.conf.d/ & vim /etc/resolvconf/resolv.conf.d/head

domain mshome.net
nameserver 192.168.137.1
nameserver 8.8.8.8
```

修改完其实不会起作用，这个文件只是用来备份，修改了`/etc/resolv.conf`才会起作用，我们需要将它复制到`/etc/resolv.conf`文件中：

```shell
cp /etc/resolvconf/resolv.conf.d/head /etc/resolv.conf 
```

为什么这样做呢？因为每次系统重启以后，`/etc/resolv.conf`这个文件会被自动清空，因此我们要将这个复制命令作为开机自启命令，使系统每次启动都会执行。如何设置开机自启会在下一小节介绍。

4.重启网络

```shell
sudo /etc/init.d/networking restart
```

5.测试是否生效

```shell
ifconfig
```

如果看到IP地址已经变为我们设置的静态IP，说明静态IP修改成功。

再看看是否能上网：

```shell
ping www.baidu.com
```

**2 设置开机自启脚本**

我们先在`/etc/init.d`目录下添加我们想开机自启动的脚本，我们将上面说的`cp /etc/resolvconf/resolv.conf.d/head /etc/resolv.conf `文件复制命令放在这个脚本中。

```shell
sudo vim /etc/init.d/<your_startup_script>.sh 
sudo chmod +x /etc/init.d/<your_startup_script>.sh
```

再将这个脚本做一个软链接到`/etc/rcS.d`目录中，并在链接文件名前附上优先级`S99`（这个很重要），代表这个脚本会优先执行：

```shell
ln -s /etc/init.d/<your_startup_script>.sh /etc/rcS.d/S99<your_startup_script>.sh
```

还有一种每次进入终端就会执行一次脚本的方式，将写好的脚本（.sh文件）放到目录`/etc/profile.d/`下，进入终端后就会自动执行该目录下的所有shell脚本，要和上面的方式区分开，有的只需要开机启动一次的脚本就不需要用这种方式了。



# 在线更新FLASH：

## 内核配置

（zynq）

The following config options need to be enabled
CONFIG_ZYNQ_SPI_QSPI
It depends on SPI_MEM, SPI_MASTER and ARCH_ZYNQ

![img](https://xilinx-wiki.atlassian.net/wiki/download/thumbnails/18842262/image2023-11-21_16-8-21.png?version=1&modificationDate=1700563104044&cacheVersion=1&api=v2&width=680&height=164)

If required, enable MTD block devices support - MTD_BLKDEVS

（zynqMP）

Kernel Configuration Options

The following config options need to be enabled
CONFIG_SPI_ZYNQMP_GQSPI
It depends on SPI_MASTER, SPI_MEM and HAS_DMA

![img](https://xilinx-wiki.atlassian.net/wiki/download/thumbnails/18841901/image2023-11-21_16-29-9.png?version=1&modificationDate=1700564352306&cacheVersion=1&api=v2&width=680&height=160)
If required, enable MTD block devices support - MTD_BLKDEVS

## QSPI设备树：

zynqMP

```dtd
&qspi {
        status = "okay";
        flash@0 {
        compatible = "n25q512a", "jedec,spi-nor";
        reg = <0x0>;
        spi-tx-bus-width = <4>;
        spi-rx-bus-width = <4>;
        spi-max-frequency = <10000000>;
        #address-cells = <1>;
        #size-cells = <1>;
        partition@0x00000000 {
                label = "boot";
                reg = <0x0 0x1E00000>;
        };

        partition@0x1E40000 {
                label = "bootenv";
                reg = <0x1E400000 0x40000>;
        };

  };
};
```

## Linux指令：

```shell
## 查询FLASH分区：
cat /proc/mtd

dev:    size   erasesize  name
mtd0: 00400000 00020000 "qspi-fsbl-uboot"
mtd1: 01a00000 00020000 "qspi-linux"
mtd2: 00010000 00020000 "qspi-device-tree"
mtd3: 00500000 00020000 "qspi-rootfs"
mtd4: 005e0000 00020000 "qspi-bitstream"

## 写入FLASH （包含擦除、写入、校验）
flashcp -v ./smaple.bin /dev/mtd0

Erasing block: 32/32 (100%)
Writing kb: 4088/4096 (99%)
Verifying kb: 4088/4096 (99%)
```

## **参考文献：**

ZYNQ QSPI驱动：https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841901/Linux+ZynqMP+GQSPI+Driver#LinuxZynqMPGQSPIDriver-Device-tree

https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842262/Zynq+QSPI+Driver



