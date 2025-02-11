# 自己总结：

## 查询rootfs大包里面包含的东西路径：

 /components/yocto/source/aarch64/layers/meta-petalinux/recipes-core/packagegroups/packagegroup-petalinux-gstreamer.bb  

## 在 PetaLinux 中，你还可以单独编译其他的组件,包括:

1. u-boot:

   ```shell
petalinux-build -c u-boot
   ```
   
   编译 U-Boot 引导程序。

2. linux:

   ```shell
petalinux-build -c linux
   ```
   
   编译 Linux 内核。

3. device-tree:

   ```shell
petalinux-build -c device-tree
   ```
   
   编译设备树。

4. rootfs:

   ```shell
petalinux-build -c rootfs
   ```
   
   编译根文件系统。

5. apl:

   ```shell
petalinux-build -c apl
   ```
   
   编译应用程序层。

6. bootloader:

   ```shell
   petalinux-build -c bootloader
   ```
   
   编译引导加载程序(包括 FSBL 和 U-Boot)。









# 网上总结：

#source settings.sh

#source components/yocto/source/aarch64/environment-setup-aarch64-xilinx-linux

#source components/yocto/source/aarch64/layers/core/oe-init-build-env

#export PATH=/home/work/petalinux/tools/hsm/bin:$PATH

#bitbake fsbl -c cleansstate

#bitbake fsbl


ZYNQMP_CONSOLE=cadence1



$cat QSPI_R5_0.bif
the_ROM_image:
{
[fsbl_config] r5_single
[bootloader] R5_FSBL.elf
[destination_cpu=r5-0] R5_core0_hello_world.elf
}

$ bootgen -r -w –image ./QSPI_R5_0.bif -o Boot.bin


$ cat SD.bif
the_ROM_image:
{
[fsbl_config] a5x_x64
[bootloader] ron_a53_fsbl.elf
[destination_cpu=a5x-0] A53_core0_hello_world.elf
}

$ bootgen -r -w -image SD.bif -o Boot.bin

UltraZed IO Carrier Card

#/etc/init.d/openbsd-inetd restart

petalinux的一些命令：

消除编译时的警告信息：

petalinux-util --webtalk off

创建新工程：
#petalinux-create --type project --template zynqMP --name /home/work/tp0805
bsp创建工程
#petalinux-create -t project -s <path-to-bsp>

配置命令：
#petalinux-config --get-hw-description=/home/work/tp0805/hdf
#petalinux-config --get-hw-description=/home/ucas/yinhonggen/hdf

清理：
#petalinux-build -x distclean

#petalinux-build -x mrproper  /*清理的最彻底，包括build, image文件夹都将被清理掉*/

打包BOOT的命令：
#petalinux-package --force --boot --fsbl zynqmp_fsbl.elf --fpga design_1_wrapper.bit --pmufw pmufw.elf --atf bl31.elf --uboot

#petalinux-package --boot --fpga system.bit --u-boot --kernel

#petalinux-package --boot --fpga system.bit --u-boot

#petalinux-package --boot --fpga system.bit --u-boot --add system.dtb --offset 0x01440000 --kernel --add rootfs.jffs2 --offset 0x02460000  --force

#petalinux-build -s
#petalinux-package --sysroot


petalinux-build -x package  //To regenerate the image.ub, Image and rootfs.cpio.gz

petalinux-build -c device-tree -x mrproper

petalinux-build -c device-tree

petalinux-build -c arm-trusted-firmware

petalinux-build -c bootloader

petalinux-build -c kernel


petalinux-build -c u-boot

petalinux-build -c device-tree -x mrproper

First create the application
$ petalinux-create -t apps -n myapp --enable
petalinux-build -c rootfs

petalinux-create -t apps --template install --name myapp --enable


Rebuild PetaLinux project for the Linux application
You can rebuild the whole project,rootfs or just the application
$ petalinux-build
$ petalinux-build -c rootfs
$ petalinux-build -c rootfs/myapp

To add Linux user libraries to your rootfs.
$ petalinux-create -t libs -n mylib --enable
The above command will create a Linux user library "mylib" in "components/libs/mylib"


$ petalinux-build -c rootfs/mylib


PetaLinux uses library priorities to decide the compilation sequence of the user libraries.
To specify the priority of you library
$ petalinux-create -t libs -n mylib --enable --priority X
X is the priority of your library."1" has the highest priority which will be built first.
Please find the available priorities from with the "--help" of "petalinux-create -t libs".

 

 

#zcat rootfs.cpio.gz | cpio -idmv 

#zcat rootfs.cpio.gz | fakeroot cpio -idmv 


#cpio -idmv < rootfs.cpio 


#find ./* | cpio -H newc -o > rootfs.cpio （或者 find ./* | cpio -H tar -o > rootfs.cpio）

#gzip rootfs.cpio

System Configuration/Yocto Settings中，Add pre-mirror url、Local sstate feeds settings设置为本地地址，格式如下：
Add pre-mirror url：
file:///petalinux/sstate-rel-v2018.2/downloads
Local sstate feeds settings：
/petalinux/sstate-rel-v2018.2/aarch64
本地资源为sstate-rel-v2018.2文件夹。