# 一、字符图形化配置界面的操作方法

## 1\. 界面划分

    纵向是配置菜单，可通过上下按键进行切换

    横向是5个按钮，可通过左右按键进行切换，通过回车按键进行触发

## 2\. 按键功能

    Select： 选择、确认

    Exit： 退出、返回

    Help： 帮助，用于查看配置项的帮助信息

    Save： 保存，保存当前配置信息，将配置信息写入到一个配置文件中

    Load： 加载，从外部加载一个配置文件

## 3\. 快捷键

    /： 搜索配置项

    ?： 帮助

    Esc：连续按2次，返回的意思，和Exit一样

# 二、字符图形化配置

**1\. 使能/禁止配置项 （符号：[ ]）**

        2选1，要么使能它，要么禁用它，用空格进行切换

        [ ] （\*：使能 空：禁止）

**2\. 多选一配置项 （符号：后面有（））**

        必须选择其中一个配置

        配置项字符串后面有一个小括号，括号里面就是选择的配置值

**3\. 可编辑的配置项（符号：前面有（））**

        配置项字符串前面有一个小括号，可以自己编辑配置值，小括号里面对应当前配置值

# 三、config配置项

Linux Components Selection ---\>

    First Stage Bootloader（fsbl）： 第一阶段的启动代码
    Auto update ps_init ： ps_init是fsbl程序中的一个函数，根据导入的hdf文件自动更新

u-boot (u-boot-xlnx) ---\> ： 配置u-boot源码的来源

    u-boot-xlnx : 默认的，是xilinx提供的源码

    remote ： 远程仓库中的源码

    ext-local-src ： 本地源码

linux-kernel (linux-xlnx) ---\>： 配置kernel源码的来源 （和u-boot一样）

Auto Config Settings ---\>

    fsbl autoconfig ，Device tree autoconfig，kernel autoconfig，u-boot autoconfig

    使能这几个自动配置，保持默认

Subsystem AUTO Hardware Settings ---\> ： 硬件设置

Memory Settings ---\> ：寄存器地址配置

Serial Settings ---\> ：串口设置

Ethernet Settings ---\> ：网口设置

Flash Settings ---\> ：flash分区设置（size：分区大小）

    'boot'存储'‘BOOT.BIN’ 。 ‘bootenv’存储 u-bootenv vars。 ‘kernel’存储‘image.ub’ 。

SD/SDIO Settings ---\> ：SD卡设置

RTC Settings ---\> ：RTC设置

Advanced bootable images storage Settings ---\> ：image存储位置设置

boot image settings ---\> ：配置BootLoader镜像（BOOT.BIN - FSBL， PMU固件，ATF，u-boot）存储位置

u-boot env partition settings ---\> ：配置u-boot环境变量分区设置

kernel image settings ---\> ：配置linux内核镜像（image.ub - Linux 内核，DTB）存储位置

jffs2 rootfs image settings ---\> ：配置jffs2根文件系统镜像位置

dtb image settings ---\> ：配置设备树Blob（dtb）存储位置

DTG Settings ---\>

Kernel Bootargs ---\> ：内核bootargs变量

    Remove PL from devicetree ：从设备树中移除PL

FPGA Manager ---\>

u-boot Configuration ---\>

    u-boot config target ：用于配置u-boot时的配置文件

    TFTP Server IP address ：设置TFTP服务器地址

    Image Packaging Configuration ---\>

Root filesystem type () ---\> ：设置rootfs根文件系统类型 （根文件系统存储位置）

1、INITRAMFS 模式（默认）  
            RootFS 被包含在内核镜像中。zImage → zImage（内核） + rootfs.cpio（用于 Zynq-7000 器件）

            此时用SD卡启动的时候，只需放入BOOT和image.ub  
2、INITRD模式（不常用）  
            设置 RAMDISK loadaddr。 确保 loadaddr 不与内核或 DTB 地址重叠， 并且是有效的 DDR 地址。  
3、NFS模式（需要安装NFS和TFTP）  
            把image.ub放到tftp服务目录下把rootfs.tar.gz解压到nfs服务目录下  
4、SD card模式 （常用）  
            把SD卡分为两个分区  
                BOOT（FAT32）：BOOT.BIN、image.ub  
                ROOTFS（ext4）：解压rootfs.tar.gz

Copy final images to tftpboot ：编译完的petalinux工程，镜像自动拷贝到tftp服务器目录

tftpboot directory ：配置tftp服务器目录

Firmware Version Configuration ---\> ：配置工程名字（几乎没用）



## 离线包配置

<mark>Yocto Settings ---\> （petalinux的底层就是Yocto ）（主要设置离线包）</mark>

Add pre-mirror url ---\> （离线编译包download）

修改为file://\<path\>/downloads，\<path\>为sstate下载包解压后的地址。

```tcl
## downloads
file:///home/peta19/sstate_2019.2/downloads

file:///home/peta18/sstate2018/downloads

file:///home/peta21/sstate_petalinux/downloads

原本：http://petalinux.xilinx.com/sswreleases/rel-v\${PETALINUX_VER%%.\*}/downloads
```

    Local sstate feeds settings ---\> （离线下载包sstate，zynq是arm，zynqMP是aach64）

```tcl
## sstate
/home/peta18/sstate2018/arm

/home/peta19/sstate_2019.2/aarch64

/home/peta21/sstate_petalinux/aarch64
```



Enable Debug Tweaks ：这一项如果使能，那么板子启动时，在串口就不需要输入账号、密码

Enable Network sstate feeds ：选择不使能，如果使能就会在网上下载资源

Enable BB NO NETWORK ：选择不使能。如果使能有些编译会出错，经验之谈



# 四、rootfs工程配置介绍



**如果内核或 RootFS 的大小增加， 并且大于 128 MB，则需要执行以下操作：**

a. 在 \<plnx_proj\>/project-spec/meta-user/recipes-bsp/u-boot/files/platform-top.h 中提到 Bootm 长度。

\#define CONFIG_SYS_BOOTM_LEN \<value greater then image size\>

b. 取消 CONFIG_SYS_BOOTMAPSZ 在

\<plnx_proj\>/project-spec/meta-user/recipes-bsp/uboot/files/platform-top.h 中的定义。

问题：qt编译不通过

Please use below workaround to build qt5 in 2022.2 or lower versions.

**Workaround:**

Step1:Create bbappend as shown below for two recipes

step2:

vim project-spec/meta-user/recipes-qt/qt5/qt3d_%.bbappend

SRC_URI = "git://[code.qt.io/qt/qt3d.git;name=qt3d;branch=5.15;protocol=git](https://code.qt.io/qt/qt3d.git;name=qt3d;branch=5.15;protocol=git)"

SRCREV = "92853c6e1aa95dfb7d605959ff44ccc124fbd62c"

step3:

vim project-spec/meta-user/recipes-qt/qt5/qtserialbus_%.bbappend

SRC_URI = "git://[code.qt.io/qt/qtserialbus.git;name=qt3d;branch=5.15;protocol=git](https://code.qt.io/qt/qtserialbus.git;name=qt3d;branch=5.15;protocol=git)"

SRCREV = "d3394c81f10e5d5c40663e88e185335549e4bc12"

step4:petalinux-build --sdk

Thanks and Regards

Raviteja

Open SSH 冲突问题：

https://support.xilinx.com/s/question/0D52E00006hpdzISAQ/petalinux-20172-cant-use-openssh-instead-of-dropbear?language=en_US
