# petalinux安装：

**1、安装依赖项**

```shell
sudo apt-get install -y gcc git make net-tools libncurses5-dev tftpd zlib1g-dev libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib build-essential zlib1g:i386 screen pax gzip
```

**2、修改.run文件的属性**

chmod a+x ppetalinux-v2019.2-final-installer.run

**3、安装petalinux安装到当前文件夹下的v2019.2里面**

./petalinux-v2019.2-final-installer.run ./petalinux

> 安装过程会弹出协议，按“q” 跳
>
> 过详情，然后输入“y” 表示同意协议内容

**4、修改到bash**

        查看：ls -lh /bin/sh （初始为dash）

        修改：sudo dpkg-reconfigure dash 然后跳出对话框，点击No即可

**5、运行环境变量**

        一次生效：在刚刚安装目录下面 source settings.sh

        永久生效：在home目录下打开.bashrc文件（如果没有Ctrl+H），在最后一行添加 source /home/ （你的目录） /settings.sh

# vivado安装和vivado卸载

**安装**

直接运行./xsetup

**卸载**

进入下面目录：

/2019.2/.xinstall/Vivado_2019.2

然后卸载

sudo ./xsetup -b Uninstall

# linux安装vivado

linux安装vivado的时候需要提前安装好 ncurses库

打开终端

```shell
sudo apt install libncurses5
```



# vivado远程下载程序

**连接开发板的主机设置**

打开vivado安装路径， {vivado安装路径}\bin\hw_server.bat

**远程的主机设置**

打开Hardware Target，

选择Connect to Remote server，然后输入Host name：（目标主机ip），port：3121







