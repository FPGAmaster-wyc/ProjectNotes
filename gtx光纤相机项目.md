# 325T开发板基本介绍



## QSFP的bank

![fd7cc72de7714baf61d5f0b274aed39](E:\my_work\ia63301\media\fd7cc72de7714baf61d5f0b274aed39.png)



## 高速bank的约束

![ac44d3f8b93e8487cd254b1df2e1c05](E:\my_work\ia63301\media\ac44d3f8b93e8487cd254b1df2e1c05.png)

## GTX IP 配置信息

使用7 Series FPGAs Transceivers Wizard 的IP

![image-20240729155000103](E:\my_work\ia63301\media\image-20240729155000103.png)

选择 逻辑在example里面，这样做节省资源

![image-20240729155232468](E:\my_work\ia63301\media\image-20240729155232468.png)

选择 TX 和 RX 的 Line Rate 速度，这个 Line Rate 速度是10.3125Gbps，选择参考时钟为156.25Mhz

选中 GTX_X0Y8, GTX_X0Y9, GTX_X0Y10,GTX_X0Y11, 然后选择右边的**"Use GTX X0Y8/9/10/11"**前面的钩，这样QPLL 都连接到了 4 个 GTXChannel 模块  

![image-20240729155619619](E:\my_work\ia63301\media\image-20240729155619619.png)

数据位宽选择64bit，然后把RX的时钟改为RXOUTCLK

![image-20240729161316677](E:\my_work\ia63301\media\image-20240729161316677.png)

关闭comma（K 码的控制字符 ）检测功能，并使用DFE自动rx均衡

![image-20240729163814171](E:\my_work\ia63301\media\image-20240729163814171.png)

其余的保持默认不变

然后导出example 工程，这个例子工程中，程序会在 gt0_frame_gen 模块中产生测试数据进行 GTX 的数据传输，在 gt0_frame_check 模块接收并检查是否正确，如果不正确，错误统计值增加  

![image-20240729164005380](E:\my_work\ia63301\media\image-20240729164005380.png)

在前面我们已经生成过 gtx IP 的 example 工程，这里删除了 gt0_frame_gen.v 模块和gt0_frame_check.v 模块。 因为这两个是测试数据的产生和检查模块，本例程用不上。另外对gtp_exdes.v 文件进行修改，主要是删除 gt0_frame_gen.v 模块和 gt0_frame_check.v 模块的例化  





## 修改例子工程

增加一个个userclk3，然后添加了drpclk(60M)

![image-20240807173526573](E:\my_work\ia63301\media\image-20240807173526573.png)

对应源代码CLHSXSerdes_GT_USRCLK_SOURCE中进行修改，调用PLL参数

```verilog
CLHSXSerdes_CLOCK_MODULE #
(
    .MULT                           (2.0),
    .DIVIDE                         (1),
    .CLK_PERIOD                     (3.103),
    .OUT0_DIVIDE                    (1),
    .OUT1_DIVIDE                    (2),
    .OUT2_DIVIDE                    (2.0624),
    .OUT3_DIVIDE                    (5.3733)
```

3.103ns代表输入时钟周期，及322Mhz

Outclk0 = 322 / 1 =322

Outclk1 = 322 / 2 =161

Outclk2 = 322 / 2.0624 = 156

Outclk3 = 322 / 5.3733 = 60



8.7号 版本的clk有问题，需要修改





# 相机bug问题

## 启动问题

先启动主机，然后启动相机，如果没有图像，需要两个设备都重启





# ubuntu安装xdma驱动

## 安装ubuntu虚拟机

前提主机硬盘有未分配区域

1、使用u盘启动主机（选择UEFI开头的）

2、安装类型选择something else （利用未分配磁盘安装）

3、对磁盘进行分区

```shell
## 第一个分区类型为根目录：命名为 ’/‘ ，相当于Win的系统盘C盘
容量大小：总容量的30%

## 第二个分区类型为启动分区：命名为 ’/boot‘ ，Ubuntu的启动项，开机时引导系统启动
容量大小：512 MB

## 第三个分区类型为交换分区（swap）：无命名，相当于Win的虚拟内存
容量大小：内存条大小的2倍

## 第四个分区类型为用户目录：命名为 ’/home‘，相当于windows下的用户文件夹（桌面、文档、下载、图片、音乐等）
容量大小：划分剩下的所有分区
```

4、设置启动引导位置

在Device for boot loader installation下拉框中，选择 ext4 /boot 所在的分区名称

5、继续安装，直至安装完成



## XMDA驱动安装

1、首先把xdma内核驱动文件拷贝到linux

2、由于提供的都是源码，需要编译安装然后才能使用

```shell
cd ~/XDMA-kernel/xdma/
#（如果之前没有安装过 build-essential，需要安装）
sudo apt install build-essential 
sudo make install

## 安装完之后 重启linux
sudo reboot
```

3、下载bit文件 然后重启

4、加载xdma驱动

```shell
cd ~/XDMA-kernel/tests
chmod +x load_driver.sh
sudo ./load_driver.sh
```

5、编译测试应用

```shell
cd ~/XDMA-kernel/tools
make
```

6、测试

```shell
cd ~/XDMA-kernel/tests
chmod +x run_test.sh
chmod +x dma_memory_mapped_test.sh
sudo ./run_test.sh
```

显示两个pass表示测试成功



## 设置自启动加载xdma驱动

```shell
## 修改load_driver.sh文件
sudo vim ~/XDMA/linux-kernel/tests/load_driver.sh

修改xdma.ko路径为绝对路径：insmod /home/pcie/XDMA/linux-kernel/xdma/xdma.ko

## 添加启动服务
sudo vim /etc/systemd/system/load_driver.service

[Unit]
Description=Load XDMA Driver at Boot
After=network.target

[Service]
ExecStart=/home/pcie/XDMA/linux-kernel/tests/load_driver.sh
ExecStartPre=/bin/sleep 2
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target

## 重新加载 systemd 配置
sudo systemctl daemon-reload
## 启用服务，使其在启动时自动运行
sudo systemctl enable load_driver.service
## 手动启动服务以进行测试
sudo systemctl start load_driver.service
## 检查服务状态
sudo systemctl status load_driver.service


## 如需停止自启服务
## 立即停止服务
sudo systemctl stop load_driver.service
## 禁用服务自启动
sudo systemctl disable load_driver.service
## 重新加载 Systemd 配置
sudo systemctl daemon-reload
```



## 配置软件环境

安装conda

```shell
## 下载conda安装脚本
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O Miniconda3-latest-Linux-x86_64.sh

## 运行 Miniconda 安装脚本
chmod +x Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh

## 激活 Conda 环境
export PATH="$HOME/miniconda3/bin:$PATH"
echo 'export PATH="$HOME/miniconda3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

## 验证 Conda 安装
conda --version

## 禁用自动激活 base 环境
conda config --set auto_activate_base false
## 手动激活
conda activate base
```

创建软件需要的环境

```shell
## 新建一个脚本文件
vim conda_create_env.sh


#!/bin/bash

# 创建 Conda 环境
conda create -n myenv python=3.8 -y

# 激活环境
source activate myenv

# 使用 pip 安装缺失的包
pip install qt5-tools==5.15.2.1.3
pip install qt5-applications==5.15.2.2.3
pip install python-dotenv==1.0.0
pip install pyqtdoc==5.15.0
pip install pyqt5-tools==5.15.9.3.3
pip install pyqt5-sip==12.12.1
pip install pyqt5-qt5==5.15.2
pip install pyqt5-plugins==5.15.9.2.3
pip install pyqt5==5.15.9
pip install opencv-python-headless==4.7.0.72
pip install numpy==1.24.3
pip install hexdump==3.3
pip install click==8.1.3

## 运行脚本
./conda_create_env.sh
```



# linux下vivado检测不到硬件



进入vivado安装目录下的这个位置：

```shell
cd /vivado_install/vivado_lab/Vivado_Lab/2018.3/data/xicom/cable_drivers/lin64
```

运行命令：

```shell
sudo cp -i -r install_script /opt
```

进入文件夹

```shell
cd /opt/install_script/install_drivers
```

运行

```shell
sudo ./ install_drivers
或者
sudo ./install_digilent.sh
```

然后重启linux即可









