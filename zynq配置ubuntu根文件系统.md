# 配置适合zynq的ubuntu的根文件系统

## 制作FSBL、U-Boot和内核

记住必须确认在命令petalinux-config的配置中，选择以SD卡启动，并正确填写rootfs的位置。我的SD卡为第二个设备，然后rootfs在其第二个分区，所以是mmcblk0p2

```shell
→ Image Packaging Configuration
	→ Root filesystem type()

		( ) INITRAMFS
		( ) INITRD
		( ) JFFS2
		( ) NFS
		(X) EXT4 (SD/eMMC/SATA/USB)
		( ) other

→ Image Packaging Configuration
	(/dev/mmcblk0p2) Device node of SD device 
```

配置完事后，直接编译就可以得到相关的文件，在$(petalinu-project)/images/linux下。


## 获得Ubuntu基本rootfs

首先在PC主机Ubuntu系统中安装qemu模拟器：

```shell
sudo apt-get install qemu-user-static
```

下载ubuntu20.04 base链接：https://cdimage.ubuntu.com/ubuntu-base/releases/

解压根文件系统并放入SD卡的EXT4分区：

```shell
sudo tar -xf ubuntu-20.04.3-minimal-armhf-2021-12-20.tar.xz
sudo tar xfvp ./armhf-rootfs-ubuntu-focal.tar -C /media/linuxusb/ROOT/
sync
sudo chown root:root /media/linuxusb/ROOT/
sudo chmod 755 /media/linuxusb/ROOT/
```





清华源：https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports/





安装必须包：

```shell
apt-get install sudo ssh net-tools ethtool
```





## 导入PL端设备驱动

此时的linux并没有包含PL端的设备驱动，需要把petalinux生成的rootfs里面的/lib文件夹内的内容，覆盖到ubuntu中

```shell
sudo cp -rf ./lib/module/. /media/linuxusb/ROOT/lib/module/
```



## 启动并配置

**网络配置**

启动后，以太网无法自行连接到网络。相反，它需要一些配置才能正常工作。如果您发现自己处于断开连接的情况，请执行以下步骤。

```shell
sudo vi /etc/netplan/networkmanager.yaml

## 放入以下内容（注意缩进保持下面不变）（自动获取ip）
network:
    version: 2
    renderer: NetworkManager
## 固定IP
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4

```

**然后启动配置并重启**

```shell
sudo netplan generate
sudo netplan apply
sudo reboot
```

现在，这将自动配置 ubuntu 网络设置。最后，您应该能够连接到网络/互联网



## 换源：

**第一步：备份源文件：**

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup
```

**第二步：编辑/etc/apt/sources.list文件**

在文件最前面添加以下条目(操作前请做好相应备份)：

```shell
sudo vi /etc/apt/sources.list

## :%d 删除所以内容

## 换源的时候一定要注意，对于嵌入式板子来说，必须使用 ubuntu-ports/ 的源，而PC机使用ubuntu的源

## 中科大20.04源（最稳定arm）
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse

## 清华源 20.04
# 20.04 LTS
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ focal-security main restricted universe multiverse

## 清华 22.04
# 22.04 LTS
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ jammy-security main restricted universe multiverse
```

**第三部：更新**

```shell
sudo apt-get update
```











## 修改主机名和提示

**修改登录名：**

```shell
## 修改主机名 （eg：aarch64）
sudo vim /etc/hostname
aarch64

## 修改主机配置
sudo /etc/hosts
127.0.0.1       localhost
127.0.1.1       aarch64
```

**修改提示语（用于提示用户名密码）**

```shell
sudo vim /etc/issue
```



## 添加用户

需要添加一个user用户，密码password

```shell
## 添加用户user （在root用户）
sudo adduser user

## 给user root权限
sudo usermod -aG sudo user
```



## 切换到root用户

```shell
sudo su
```





wget http://fishros.com/install -O fishros && . fishros  



# 总结

此次配置未成功

# 参考

参考Ubuntu提供的网站：https://wiki.ubuntu.com/ARM/RootfsFromScratch/QemuDebootstrap

zynqMP搭载ubuntu：https://blog.csdn.net/Markus_xu/article/details/117020452

zynq搭载ubuntu:https://www.lh123lh.gq/2019/03/11/ZYNQ%E7%A7%BB%E6%A4%8Dubuntu_18-04/

arm64野火教程：https://doc.embedfire.com/linux/rk356x/build_and_deploy/zh/latest/building_image/ubuntu_rootfs/ubuntu_rootfs.html





