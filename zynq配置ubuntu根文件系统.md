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


## 获得Ubuntu基本rootfs（22.04）

首先在PC主机Ubuntu系统中安装qemu模拟器：

```shell
sudo apt-get install qemu-user-static
```

下载ubuntu22.04 base链接：https://mirrors.bfsu.edu.cn/ubuntu-cdimage/ubuntu-base/releases/22.04.4/release/

​	https://cdimage.ubuntu.com/ubuntu-base/releases/22.04.4/release/

```shell
sudo tar -xvf ubuntu-base-22.04.4-base-armhf.tar.gz
# 配置网络，复制本机 resolv.conf 文件
sudo cp /etc/resolv.conf ./ubuntu/etc/resolv.conf
# 换源
sudo vim ./ubuntu/etc/apt/sources.list
# 拷贝 qemu-arm-static 到 ubuntu_rootfs/usr/bin/ 目录下。
sudo cp $(which qemu-arm-static) ./ubuntu/usr/bin
```

ubuntu22.04国内源：

```shell
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb http://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb http://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb http://mirrors.bfsu.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ jammy-security main restricted universe multiverse

## 清华源
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ noble-backports main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports/ noble-security main restricted universe multiverse
```

**注：**清华源镜像：https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports/

注意不要使能https格式



### **挂载根文件系统**

首先编写挂载脚本 mount.sh，用于挂载根文件系统运行所需要的设备和目录。

```sh
#!/bin/bash
mnt() {
	echo "MOUNTING"
	sudo mount -t proc /proc ${2}proc
	sudo mount -t sysfs /sys ${2}sys
	sudo mount -o bind /dev ${2}dev
	sudo mount -o bind /dev/pts ${2}dev/pts
	# sudo chroot ${2}
}
umnt() {
	echo "UNMOUNTING"
	sudo umount ${2}proc
	sudo umount ${2}sys
	sudo umount ${2}dev/pts
	sudo umount ${2}dev
}
 
if [ "$1" == "-m" ] && [ -n "$2" ] ;
then
	mnt $1 $2
elif [ "$1" == "-u" ] && [ -n "$2" ];
then
	umnt $1 $2
else
	echo ""
	echo "Either 1'st, 2'nd or both parameters were missing"
	echo ""
	echo "1'st parameter can be one of these: -m(mount) OR -u(umount)"
	echo "2'nd parameter is the full path of rootfs directory(with trailing '/')"
	echo ""
	echo "For example: ch-mount -m /media/sdcard/"
	echo ""
	echo 1st parameter : ${1}
	echo 2nd parameter : ${2}
fi
```

保存退出后，给脚本增加执行权限，并挂载。

```shell
# 增加脚本执行权限
sudo chmod +x mount.sh
# 挂载文件系统
./mount.sh -m ./ubuntu/
# 进入根文件系统
sudo chroot ./ubuntu/
# 卸载文件系统
./mount.sh -u ./ubuntu/
```

### **安装必须包**

```shell
apt-get update
apt-get install sudo ssh net-tools ethtool vim openssh-server tzdata iputils-ping ifupdown iproute2 -y
```

### **用户设置**

添加用户

```shell
# 根据提示设置密码
adduser user
```

修改/etc/sudoers里面的内容，在root行下加上这句，然后你创建的用户就可以用sudo获得root权限了。

```shell
vim /etc/sudoers
xxx(用户名)    ALL=(ALL:ALL) ALL
```

设置主机名称

```shell
echo "ubuntu-arm-zynq">/etc/hostname
```

设置本机入口ip：

```shell
echo "127.0.0.1 localhost">>/etc/hosts
echo "127.0.0.1 ubuntu-arm-zynq">>/etc/hosts
echo "127.0.0.1 localhost ubuntu-arm-zynq" >> /etc/hosts
```

### **DNS配置问题**

如果出现apt-get update报错误，需要手动修改DNS配置

```shell
## 手动更新
sudo vim /etc/resolv.conf

## 添加 DNS 服务器地址
nameserver 8.8.8.8
nameserver 8.8.4.4

## 自动 
dpkg-reconfigure resolvconf
## 卸载
sudo apt-get remove resolvconf
```

**设置时区**

```shell
dpkg-reconfigure tzdata
```

### **配置串口调试服务**

```shell
## 如果没有/etc/init/ttyPS0.conf，则自行创建
mkdir /etc/init
vim /etc/init/ttyPS0.conf

start on stoppedrc or RUNLEVEL=[12345]
stop on runlevel[!12345]
respawn
exec /sbin/getty -L 115200 ttyPS0 vt102

## 建立一个软连接
ln -s /lib/systemd/system/getty@.service /etc/systemd/system/getty.target.wants/getty@ttyPS0.service
```

### **配置网络**

**network服务**

```shell
## 创建并打开`/etc/network/interfaces`文件   （必须安装ethtool）
vim /etc/network/interfaces

# 本地回环
auto lo 
iface lo inet loopback 

# 两种方法任选一个

# 1、获取动态配置： 
auto eth0 
iface eth0 inet dhcp 

# 2、获取静态配置： 
auto eth0 
iface eth0 inet static 
address 192.168.3.124 
netmask 255.255.255.0 
gateway 192.168.3.1 
```

**netplan服务**

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



实际测试中网口必须接入网线系统才能正常启动，就是在不联网的情况下，每次开机都要等待很久，卡在网络连接上5分钟。

```shell
修改下面这个文件
vim /lib/systemd/system/networking.service
将里面的TimeoutStartSec=5min 修改为：

TimeoutStartSec=30sec
```

### **清除缓存**

```shell
## 清理APT缓存
apt-get clean

## 自动清理无用的依赖项
apt-get autoremove
```



### **压缩根文件系统**

把生成的根文件系统，压缩为.tar.gz的压缩包

```shell
sudo tar -zcvf ubuntu22.04-arm-zynq.tar.gz -C ./ubuntu/ .
```





### **额外功能**

指令tab补全 （需要在开发板操作）

```shell
## 安装bash-completion
sudo apt install bash-completion

## 启用 bash-completion
vim ~/.bashrc

if [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
fi

## 重新加载 .bashrc 配置文件
source ~/.bashrc
```











































## 导入PL端设备驱动

此时的linux并没有包含PL端的设备驱动，需要把petalinux生成的rootfs里面的/lib文件夹内的内容，覆盖到ubuntu中

同时需要把硬件驱动firmware复制到ubuntu

```shell
sudo cp -rf ./lib/module/. /media/linuxusb/ROOT/lib/module/
sudo cp -rf ./lib/firmware/ /media/linuxusb/ROOT/lib/
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



wget http://fishros.com/install -O fishros && . fishros  



# 总结

此次配置未成功

# 参考

参考Ubuntu提供的网站：https://wiki.ubuntu.com/ARM/RootfsFromScratch/QemuDebootstrap

zynqMP搭载ubuntu：https://blog.csdn.net/Markus_xu/article/details/117020452

zynq搭载ubuntu:https://www.lh123lh.gq/2019/03/11/ZYNQ%E7%A7%BB%E6%A4%8Dubuntu_18-04/

arm64野火教程：https://doc.embedfire.com/linux/rk356x/build_and_deploy/zh/latest/building_image/ubuntu_rootfs/ubuntu_rootfs.html

基于 RK3588 构建 Ubuntu 22.04 根文件系统：https://blog.csdn.net/qq_34117760/article/details/130909986





