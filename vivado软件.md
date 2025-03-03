# linux下 vivado安装和卸载

## 安装

直接运行./xsetup

## 卸载

进入下面目录：

/2019.2/.xinstall/Vivado_2019.2

然后卸载

sudo ./xsetup -b Uninstall



# linux检测不到JTAG

关闭这个，重新打开

![image-20250225171440828](./media/image-20250225171440828.png)



# vivado烧写FLASH

## 一、通过BIN文件

**1、生成BIN文件**

![image-20250226111312690](./media/image-20250226111312690.png)

**2、烧写BIN文件**

添加FLASH器件

![image-20250226161930655](./media/image-20250226161930655.png)

![image-20250226162046778](./media/image-20250226162046778.png)

![image-20250226162153105](./media/image-20250226162153105.png)





## 二、通过MCS文件

**1、生成MCS文件**

打开创建存储配置文件窗口

Tools -> Generate Memory Configuration File…

![image-20250225171823773](./media/image-20250225171823773.png)

配置生成文件

![image-20250225172340681](./media/image-20250225172340681.png)



**2、烧写MCS文件**

同烧写BIN文件，只不过把BIN文件换成MCS文件



