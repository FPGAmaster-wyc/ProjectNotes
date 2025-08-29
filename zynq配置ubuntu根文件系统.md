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

https://cdimage.ubuntu.com/ubuntu-base/releases/22.04.4/release/

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

SCRIPT_NAME=$(basename "$0")
MOUNT_POINTS=("proc" "sys" "dev" "dev/pts")

show_help() {
    cat << EOF
Usage: $SCRIPT_NAME [-m <path>] [-u <path>] 

Mount or unmount directories for chroot environment.

Options:
  -m, --mount <path>    Mount proc, sys, dev and dev/pts to the specified path and chroot
  -u, --umount <path>   Unmount previously mounted directories from the specified path
  -h, --help           Show this help message

Examples:
  $SCRIPT_NAME -m /media/sdcard/
  $SCRIPT_NAME -u /media/sdcard/
EOF
}

mount_directories() {
    local target_path="$1"
    echo "Mounting directories to $target_path"
    
    for mount_point in "${MOUNT_POINTS[@]}"; do
        local source_path="/$mount_point"
        local target_full_path="${target_path%/}/$mount_point"
        
        # 创建目标目录（如果不存在）
        sudo mkdir -p "$target_full_path" 2>/dev/null
        
        if mountpoint -q "$target_full_path"; then
            echo "  ⚠ Already mounted: $target_full_path"
            continue
        fi
        
        case $mount_point in
            "proc")
                sudo mount -t proc "$source_path" "$target_full_path" && \
                echo "  ✓ Mounted proc → $target_full_path" || \
                echo "  ❌ Failed to mount proc"
                ;;
            "sys")
                sudo mount -t sysfs "$source_path" "$target_full_path" && \
                echo "  ✓ Mounted sys → $target_full_path" || \
                echo "  ❌ Failed to mount sys"
                ;;
            *)
                sudo mount -o bind "$source_path" "$target_full_path" && \
                echo "  ✓ Mounted $mount_point → $target_full_path" || \
                echo "  ❌ Failed to mount $mount_point"
                ;;
        esac
    done
}

unmount_directories() {
    local target_path="$1"
    echo "Unmounting directories from $target_path"
    
    # Unmount in reverse order to handle dependencies
    for ((i=${#MOUNT_POINTS[@]}-1; i>=0; i--)); do
        local mount_point="${MOUNT_POINTS[$i]}"
        local target_full_path="${target_path%/}/$mount_point"
        
        if mountpoint -q "$target_full_path"; then
            sudo umount "$target_full_path" 2>/dev/null && \
                echo "  ✓ Unmounted $target_full_path" || \
                echo "  ⚠ Failed to unmount $target_full_path (may be busy)"
        else
            echo "  ℹ Not mounted: $target_full_path"
        fi
    done
}

check_path() {
    local path="$1"
    local operation="$2"
    
    if [[ -z "$path" ]]; then
        echo "Error: No path specified for $operation"
        return 1
    fi
    
    # 确保路径以斜杠结尾
    if [[ "$path" != */ ]]; then
        path="${path}/"
    fi
    
    if [[ ! -d "$path" ]]; then
        echo "Error: Target path '$path' does not exist or is not a directory"
        return 1
    fi
    
    echo "$path"
    return 0
}

main() {
    local target_path=""
    local operation=""
    
    # Show help if no arguments provided
    if [[ $# -eq 0 ]]; then
        show_help
        exit 1
    fi

    while [[ $# -gt 0 ]]; do
        case $1 in
            -m|--mount)
                operation="mount"
                if [[ -z "$2" || "$2" == -* ]]; then
                    echo "Error: No path specified for mount"
                    show_help
                    exit 1
                fi
                target_path=$(check_path "$2" "mount") || exit 1
                shift 2
                ;;
            -u|--umount)
                operation="umount"
                if [[ -z "$2" || "$2" == -* ]]; then
                    echo "Error: No path specified for umount"
                    show_help
                    exit 1
                fi
                target_path=$(check_path "$2" "umount") || exit 1
                shift 2
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            *)
                echo "Error: Unknown option '$1'"
                show_help
                exit 1
                ;;
        esac
    done

    case $operation in
        "mount")
            if mount_directories "$target_path"; then
                echo -e "\n✅ Mount completed successfully!"
                echo -e "You can now run: sudo chroot $target_path"
                echo -e "Or press Enter to chroot now (Ctrl+D to exit)..."
                read -p "Press Enter to continue or Ctrl+C to cancel..."
                sudo chroot "$target_path"
                echo -e "\nExited chroot environment"
            else
                echo -e "\n❌ Mount failed with errors"
                exit 1
            fi
            ;;
        "umount")
            unmount_directories "$target_path"
            echo -e "\n✅ Unmount completed!"
            ;;
    esac
}

# 检查是否是sudo运行
if [[ $EUID -ne 0 ]]; then
    echo "This script must be run as root or with sudo"
    exit 1
fi

main "$@"
```

保存退出后，给脚本增加执行权限，并挂载。

```shell
# 增加脚本执行权限
sudo chmod +x mount.sh
# 挂载文件系统
sudo ./mount.sh -m ./ubuntu/
# 进入根文件系统
sudo chroot ./ubuntu/
# 退出根文件系统
exit
# 卸载文件系统
sudo ./mount.sh -u ./ubuntu/
```

### **安装必须包**

```shell
apt-get update
apt-get install sudo ssh net-tools ethtool vim openssh-server tzdata iputils-ping ifupdown iproute2 netplan.io udev -y
```

### **用户设置**

**添加 user 用户**

```shell
# 根据提示设置密码
adduser user

# 将用户添加到sudo组
usermod -aG sudo user
```

**删除 ubuntu 用户**

```bash
# 删除用户及其主目录
userdel -r ubuntu

# 确认用户已删除
cat /etc/passwd | grep -E "(ubuntu|user)"
```

**设置root密码** （进入root用户的密码）

```bash
passwd root
```

**设置主机名称**

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

[文件内容]
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
hwaddress ether 00:11:22:33:44:55  # 设置新的 MAC 地址（不需要改 可以删除这一行）
```

**netplan服务（推荐）**

```shell
sudo vi /etc/netplan/01-network.yaml

## 放入以下内容（注意缩进保持下面不变）（自动获取ip）
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
      dhcp6: false

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

# 设置权限
chmod 600 /etc/netplan/01-network.yaml
chown root:root /etc/netplan/01-network.yaml

# 启用 systemd 服务
systemctl enable systemd-networkd
systemctl enable systemd-resolved

# 设置 DNS 链接
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf

# 然后启动配置并重启
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



### **安装 TFTP 服务**

**1、安装 TFTP 和 TFTP-HPA 服务**

在终端中输入以下命令来安装 TFTP 服务器和 TFTP-HPA：

```shell
sudo apt update
sudo apt install tftp-hpa tftpd-hpa
```

**2、配置tftp服务**

安装完成后，需要修改 TFTP 的配置文件 `/etc/default/tftpd-hpa`，以确保 TFTP 服务器正确配置。使用以下命令编辑该文件：

```shell
sudo vim /etc/default/tftpd-hpa

## 编辑文件，将内容修改为如下所示：
# /etc/default/tftpd-hpa
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/srv/tftp"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure"

## TFTP_DIRECTORY 是 TFTP 服务器使用的根目录，默认可以设置为 /srv/tftp。
## TFTP_ADDRESS 指定 TFTP 服务器监听的地址和端口，0.0.0.0:69 表示监听所有 IP 地址的 69 端口。
```

**3、启动和启用 TFTP 服务**

编辑和配置完成后，可以通过以下命令启动 TFTP 服务器：

```shell
sudo systemctl restart tftpd-hpa
```

如果需要设置 TFTP 服务器在开机时自动启动，使用以下命令：

```shell
sudo systemctl enable tftpd-hpa
```

**4、测试 TFTP 服务**

可以使用 `tftp` 命令来测试服务器是否正常工作。例如，可以上传或下载一个测试文件：

```shell
# 连接 TFTP 服务器
tftp 192.168.x.x
tftp> put testfile
tftp> get testfile

# 退出TFTP服务器
tftp> quit
```

确保 TFTP 服务器工作正常，并且可以从客户端上传和下载文件。



### **清除缓存**

```shell
## 清理APT缓存
apt-get clean

## 自动清理无用的依赖项
apt-get autoremove
```



### **解压缩 rootf.tar.gz**

压缩为.tar.gz的压缩包

```shell
sudo tar -zcvf ubuntu22.04-arm-zynq.tar.gz -C ./ubuntu/ .
```

解压根文件系统到SD

```shell
sudo tar -zxvf ubuntu.tar,gz -C /media/linuxusb/ROOT/
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



### **修改为中文语言**

1. 改成中文语言

   1. 安装中文支持包

      ```shell
      sudo apt-get install language-pack-zh-hans
      ```

   2. 修改配置

      ```shell
      /etc/default/locale
      ```

      注释掉

      ```shell
      LANG=en_US.UTF-8
      ```

      ，添加

      ```shell
      LANG=zh_CN.UTF-8
      ```



### **修改启动提示信息**

**修改 issue 文件**（输入密码前提示信息）

```bash
# 备份原始文件
cp /etc/issue /etc/issue.backup
cp /etc/issue.net /etc/issue.net.backup

# 编辑 issue 文件
vim /etc/issue

\e[1;32m===============================================\e[0m
\e[1;32m        Zynq Ubuntu Root Filesystem\e[0m
\e[1;32m===============================================\e[0m

\e[1;33m*** SYSTEM INFORMATION ***\e[0m
   �~V� Hardware: \e[1;36mXilinx Zynq-7000 ARM\e[0m
   �~V� OS: \e[1;36mUbuntu 20.04 LTS\e[0m
   �~V� Terminal: \e[1;36mttyPS0\e[0m

\e[1;33m*** LOGIN CREDENTIALS ***\e[0m
   �~V� Username: \e[1;31muser\e[0m
   �~V� Password: \e[1;31mpassword\e[0m
   \e[1;33mPlease change password after first login!\e[0m

\e[1;32m===============================================\e[0m

# 复制到 /etc/issue.net 文件
cp /etc/issue /etc/issue.net
```

**修改 motd 文件**（输入密码后提示信息,自定义）

```bash
# 现在一台ubuntu上生成motd文件
# 安装生成 ASCII 大字工具
sudo apt-get install figlet toilet

# 生成大字 到motd文件（以 FPGA master 为例）
echo -e "\e[1;36m$(figlet FPGA master)\e[0m" | ./motd

# 将 motd 文件，放入/etc目录，然后去编辑
vim /etc/motd

Welcome to Zynq system Ubuntu 20.04 LTS !!!

^[[1;36m _____ ____   ____    _                          _
|  ___|  _ \ / ___|  / \     _ __ ___   __ _ ___| |_ ___ _ __
| |_  | |_) | |  _  / _ \   | '_ ` _ \ / _` / __| __/ _ \ '__|
|  _| |  __/| |_| |/ ___ \  | | | | | | (_| \__ \ ||  __/ |
|_|   |_|    \____/_/   \_\ |_| |_| |_|\__,_|___/\__\___|_|
                                                              ^[[0m

 * Documentation:  https://blog.csdn.net/w18864443115
 * Github:         https://github.com/FPGAmaster-wyc
 * Support:        https://postimg.cc/nsWYsg8f

 * If you encounter any issues during use, feel free to reach
   out to me for discussion.Let's learn together and make progress
   side by side.

Best wishes for you !!!




```

**修改 update-motd.d 目录内的脚本**（输入密码后提示信息,系统）

`/etc/update-motd.d/` 目录下的一系列脚本（比如 `00-header`, `10-help-text`, `50-motd-news`, `90-updates-available` 等）会动态生成登录提示





### **修改串口重复打印bug**

```bash
# 修改 getty@ttyPS0.service 的 override 文件 （防止重新打开一个串口）
mkdir -p /etc/systemd/system/getty@ttyPS0.service.d
vim /etc/systemd/system/getty@ttyPS0.service.d/override.conf

# 内容写入：
[Service]
ExecStart=
ExecStart=-/sbin/agetty --noclear ttyPS0 115200 linux
TTYVHangup=no
TTYVTDisallocate=no

# 重新加载 systemd 并重启 getty
systemctl daemon-reexec
systemctl restart getty@ttyPS0.service

# mask 掉 systemd 的 serial-getty@ttyPS0.service (防止两个串口打印)
systemctl mask serial-getty@ttyPS0.service

```

















## 导入PL端设备驱动

此时的linux并没有包含PL端的设备驱动，需要把petalinux生成的rootfs里面的/lib文件夹内的内容，覆盖到ubuntu中

同时需要把硬件驱动firmware（无线网卡、显卡、声卡固件）复制到ubuntu

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





