# 开发环境：

vivado 2021.2

petalinux 2021.2、petalinux2019.2

xenomai 3.1

内核版本：4.9.24

ubuntu18.04

使用petalinux2021.2生成BOOT.bin、boot.scr、rootfs.tar.gz，然后用petalinux2019.2的环境生成linux4.9.24的内核文件

# xenomai系统构建步骤

在官网下载xenomai内核，注意版本

注意zynq下载arm版本，zynqMP下载arm64版本

xenomai镜像下载：https://source.denx.de/Xenomai/xenomai

补丁下载链接：https://source.denx.de/Xenomai/download-archive/-/tree/main/ipipe/v4.x/arm?ref_type=heads



说明：xenomai 3.1以及以前的必须打ipip补丁，xenomai3.2及以后的要打dovetail补丁

# linux系统下载

```shell
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.9.24.tar.xz
tar -xvf linux-4.9.24.tar.xz
```

然后需要把zynq的配置文件放到linux系统里面 /linux-4.9.24/arch/arm/configs

zynq配置文件下载：https://wyc-yun.lanzn.com/iT5QV24ptesd

# xenomai操作步骤

## 1、向linux内核打入补丁

进入补丁包下载目录，将其解压：

```shell
$ cd /home/luo/linux
$ bzip2 -d patch-4.19-dovetail1.patch.bz2
```


而后进入xenomai目录，打入补丁：

```shell
./scripts/prepare-kernel.sh --arch=arm --ipipe=../ipipe-core-4.9.24-arm-2.patch --linux=../linux-4.9.24

./scripts/prepare-kernel.sh --arch=arm --dovetail=../patch-5.10.25-dovetail1.patch --linux=../linux-5.10.25/
```

## 2、加载zynq配置文件

切换到linux内核源码路径，然后输入下面指令加载zynq配置文件

```shell
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- xilinx_zynq_defconfig
```

## 3、配置内核

```shell
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

需要关闭的有

```shell
CPU Power Management --->
	CPU Frequency scaling  --->
		[] CPU Frequency scaling
	CPU Idle  --->
		[] CPU idle PM support

Kernel Features  --->
	[ ] Contiguous Memory Allocator
```

## 4、编译内核

```shell
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage
```

## 5、修改内核文件格式为uImage

```shell
mkimage -A arm -O linux -T kernel -C none -a 0x00200000 -e 0x00200000 -n "Linux-4.9.24" -d arch/arm/boot/zImage arch/arm/boot/uImage
```

## 6、启动开发板

使用petalinux2021.2，利用BSP创建一个工程，然后把生成BOOT.bin和boot.scr

制作一个SD卡，然后分为两个区，

FAT32：BOOT.bin，boot.scr，uImage（第五步生成的）

EXT4：解压rootfs.tar.gz

## 7、编译xenomai

创建我们的Linux系统需要运行的Xenomai库和工具，切换到xenomai目录

```shell
./scripts/bootstrap
```

编译xenomai，其中编译器指定为aarch64-linux-gnu-gcc， 链接器指定为aarch64-linux-gnu-ld。

```shell
./configure CFLAGS="-march=armv7-a -mfpu=vfp3 -mfloat-abi=hard" LDFLAGS="-march=armv7-a" --build=i686-pc-linux-gnu --host=arm-none-linux-gnueabi --with-core=cobalt --enable-smp --enable-tls CC=arm-linux-gnueabihf-gcc LD=arm-linux-gnueabihf-ld
```

创建xenomai

```shell
make -j$(nproc) DESTDIR=`pwd`/build-arm install
```

此时即可在build-arm目录下找到xenomai编译后的相关库及可执行文件。

把生成的xenomai文件夹及里面的内容复制到根文件系统的/usr下面

```shell
sudo cp -r ./xenomai/ /media/linuxusb/ROOT/usr/
```



## 8、修改全局变量

使用vim打开/etc/profile，添加xenomai环境变量如下：

```shell
vim /etc/profile
export PATH$PATH:/usr/xenomai/bin:/usr/xenomai/1ib/:/usr/xenomai/sbin
```

执行如下指令使能全局变量即可。

```bash
source /etc/profile
```



## 9、 测试

### 对实时内核进行自动调优

进入/usr/xenomai/sbin目录，运行下面指令可以测量和设置系统中断、内核和用户态的延迟。

```shell
cd /usr/xenomai/sbin
sudo chmod 777 ./autotune
sudo ./autotune --period 100000
```

**说明：**

autotune是Xenomai 提供的一个工具，用于自动调整和优化系统的实时性能参数。通过运行 autotune，可以测量和设置系统中断、内核和用户态的延迟。

采样周期为 100,000 纳秒

最终获得的参数为：

gravity: irq=498ns kernel=2490ns user=4980ns

中断处理延迟 (irq gravity): 表示从中断发生到中断服务程序开始执行的时间。

内核态延迟 (kernel gravity): 表示在内核态下执行任务的延迟时间。

用户态延迟 (user gravity): 表示在用户态下执行任务的延迟时间。

### 延迟性能测试

xenomai运行需要root权限，因为我是通过root用户登录的，故执行如下指令即可测试xenomai实时性。

还要给latency添加权限

```shell
cd /usr/xenomai/bin 
sudo chmod 777 ./latency
sudo ./latency
```

**说明**

说明：

latency 工具用于测量 Xenomai 实时内核的延迟性能。

采样周期设置为 1000 微秒

测试模式为周期性用户态任务

所有结果均以微秒为单位

周期性用户态任务，周期为 1000 微秒，优先级为 99

各列解释：各列从左到右依次是：

lat min: 最小延迟时间。

lat avg: 平均延迟时间。

lat max: 最大延迟时间。

overrun: 超时次数（任务未能在周期内完成的次数）。

msw: 模式切换次数（Mode Switches），通常是从用户态切换到内核态的次数。

lat best: 最佳延迟时间（整个测试期间的最小值）。

lat worst: 最差延迟时间（整个测试期间的最大值）

RTT: 实时任务的摘要行，显示测试开始时间。

RTH: 表示实时任务的头部（Real-Time Header）。

RTD: 表示实时数据（Real-Time Data）。

RTS: 实时统计（Real-Time Statistics）。

















# 参考文档：

zynqMP上构建xenomai：https://blog.csdn.net/qq_43741719/article/details/128800791

zynq7020上构建xenomai：https://fpga.eetrend.com/content/2018/100015540.html