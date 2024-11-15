# 新创建虚拟机需要安装的几个必须

## 1、安装网络工具包

```shell
sudo apt install net-tools
```



## 2、安装SSH服务

```shell 
 sudo apt update

 sudo apt install openssh-server
```



## 3、更换国内镜像源

```shell
 ## 1） 备份---
 sudo cp /etc/apt/sources.list /etc/apt/[sources.list.back](http://sources.list.back/)

## 2）修改源---
sudo vim /etc/apt/sources.list （从网上找到国内源）

 ## 3） 更新---
 sudo apt-get update
```



##  4、在线安装VMware tools

```shell
sudo apt-get install open-vm-tools-desktop -y
```

 

 

# linux基本操作

## scp传输文件

​     scp [选项] [来源文件] [目标文件]

​     eg：scp file.txt user@10.0.0.1:/home/user/

## 解压解压文件

**压缩**

```shell
sudo tar -czvf test_contents.tar.gz -C /path/to/test .
```

**解压**     

```shell
tar -zxvf 文件名.tar.gz

eg：tar -zxvf Xilinx_Vivado_Vitis_Update_2021.1.1_0728_1534.tar.gz

unzip 文件名.zip

eg：unzip lvds.zip
```



 

## 查询内存

​     df

## 删除文件

​     rm -rf [文件夹]

## 设置启动软件指令：（vivado）

1、在软件安装目录下创建脚本文件

​          sudo vim vivado-launcher.sh

​          脚本内容：

​              \#!/bin/bash

/path/to/vivado/bin/vivado （软件可执行文件位置）

2、添加可执行权限

​          sudo chmod +x vivado-launcher.sh

3、永久添加到 PATH 环境变量

​          sudo vim ~/.bashrc 

​          在最后一行添加：

export PATH=$PATH:/path/to/script_directory （/path/to/script_directory为脚本所在文件夹）

4、脚本重命名

sudo mv vivado-launcher.sh vivado （在脚本文件夹内操作）

 

## 格式化内存卡

```shell
## EXT4
mkfs.ext4 /dev/mmcblk0p2

## FAT32
mkfs.vfat /dev/mmcblk0p1
```

​	 



## 复制文件

```shell
cp -r ./test ./down  把test文件夹及其里面的内容，复制到down
```

 

## 修改简短密码

```shell
sudo passwd username
```

 

##  创建快捷方式

手动创建 vivado18_lab.desktop

创建快捷方式  vivado18_lab.desktop 到 /usr/share/applications 中

```shell
cd /usr/share/applications
sudo vim vivado18_lab.desktop
```

 在Vim中打开vivado18_lab.desktop空文件，按i进行编辑。输入一下文字

```tcl
#!/usr/bin/env xdg-open
[Desktop Entry]
Version=1.0
Type=Application		 # 快捷方式类型
Terminal=false			# 是否启用终端
Exec=/snap/bin/skype	# 应用程序的执行路径	
Name=Skype				# 软件名字
Comment=Skype			#对该应用程序的简短描述
Icon=/snap/skype/101/meta/gui/skypeforlinux.png		# 显示出的图标
```

