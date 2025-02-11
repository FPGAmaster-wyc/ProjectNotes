# 2025.2.6

**项目开始**

开发板电源：12V 3A

验证k7开发板，是否能检测到芯片，通过xilinx下载器+下载夹

下载夹子引脚说明：

![image-20250206160136734](./media/image-20250206160136734.png)

![image-20250206155547043](./media/image-20250206155547043.png)

并且可以成功下载bit文件

![image-20250206160004623](./media/image-20250206160004623.png)

硬件连接图

![6775ddd6d86d304ffbb7fc9e13aaf98](./media/6775ddd6d86d304ffbb7fc9e13aaf98.jpg)

## FTDI编程

通过FTDI软件把usb编程为下载器

首先连接usb到电脑，点击Find Devices，然后选择preset为JTAG-SMT2，然后点击Program，下载即可

然后下载就可以用vivado通过usb下载程序了

![Image](./media/Image.png)







# 2025.2.7

## 测试zc706光口

使用Xilinx官方IP核ibert，**必须那光纤线连接来个光口**

![image-20250207150732448](./media/image-20250207150732448.png)

根据原理图查看SFP在BANK111，时钟在BANK112

![image-20250207151010632](./media/image-20250207151010632.png)

![image-20250207150738232](./media/image-20250207150738232.png)

![image-20250207150744185](./media/image-20250207150744185.png)

![image-20250207150811635](./media/image-20250207150811635.png)

![image-20250207150817679](./media/image-20250207150817679.png)

![image-20250207151549311](./media/image-20250207151549311.png)





**参考文章：**

https://blog.csdn.net/mcupro/article/details/139740739

《1_【正点原子】Z100 ZYNQ之FPGA开发指南V1.0.pdf》



# 测试325T光口

光口连接方向

![image-20250207171947487](./media/image-20250207171947487.png)

![image-20250207172134822](./media/image-20250207172134822.png)

下载完bit后，需要手动添加link，然后把loopback mode改为Near-End PCS模式，然后点reset即可



**参考文献**

https://blog.csdn.net/u013184273/article/details/119136716



# 2025.2.8

测试zc706光纤自回环发射数据

根据之前的Aurora测试工程，需要修改输入时钟为差分时钟，按照如下设置之后，被分出来的时钟AUXCLK和USERCLK时钟频率相同，此处都是156.25M

![image-20250208161607722](./media/image-20250208161607722.png)

还需要添加SFP使能信号，一般为低电平有效，但是电路上添加了个取反电路，所以给使能信号为1，代表使能这个SFP

![image-20250208161956227](./media/image-20250208161956227.png)

测试的发射数据和接收数据（同一个光纤口）

![image-20250208162036583](./media/image-20250208162036583.png)

![image-20250208162156649](./media/image-20250208162156649.png)
