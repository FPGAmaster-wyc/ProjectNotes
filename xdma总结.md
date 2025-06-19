# xdma中断

## FPGA配置

XDMA IP配置

![image-20250508151638480](./media/image-20250508151638480.png)

**User Interrupts:**用户中断， XDMA提供16条中断线给用户逻辑， 这里面可以配置使用几条中断线。
**Legacy Interrupt:**XDMA 支持 Legacy 中断,我们这么不选 （此中断会一直触发）
**MSI Capabilities:**选择支持MSI中断 ,支持16个中断消息向量
	注意： MSI 中断和 MSI-X 中断只能选择一个， 否则会报错， 如果选择了 MSI 中断， 则可以选择 Legacy 中断， 如果选择了 MSI-X 中断， 那么 MSI 必须取消选择， 同时 Legacy 也必须选择 None。 此 IP 对于 7 系列设置有这个问题， 如果使用 Ultrascale 系列， 则可以全部选择
**MSI-X Capabilities:**不选
**Miscellaneous:**选 Extended Tag Field
**Link Status Register:**选 Enable Slot Clock Configuration  



## xdma中断说明

1)、 Legacy Interrupts：

对于 Legacy Interrupts 中断， 当 user_irq_ack 第一次为 1 的时候 usr_irq_req 可以清 0， 当 user_irq_ack 第二次为 1，的时候， 可以重新设置 usr_irq_req 发起中断。

外部可以由按键进行触发16位![image-20250508151717740](./media/image-20250508151717740.png)

## PC软件代码

```bash
## 检查中断号
cat /proc/interrupts | grep xdma
285:   11197285          0          0          0          0          0          0          0   PCI-MSI 537395200 Edge      xdma
## 解释
285：中断号（IRQ 285）。
8855271：该中断已触发的次数（说明 FPGA 已经成功发送了中断）。
PCI-MSI 537395200 Edge：中断类型为 PCIe MSI（边缘触发）。

xdma：驱动名称（Xilinx DMA 驱动）。

## 查看中断的触发次数
watch -n 1 "cat /proc/interrupts | grep xdma"
```

扫描中断触发c程序

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/select.h>
#include <stdint.h>

#define MAX_EVENTS 16  // 假设最多 16 个中断通道

int main() {
    int event_fds[MAX_EVENTS];
    char dev_path[64];
    fd_set read_fds;
    int max_fd = 0;
    int i, ret;

    // 1. 打开所有 event 设备文件
    for (i = 0; i < MAX_EVENTS; i++) {
        snprintf(dev_path, sizeof(dev_path), "/dev/xdma0_events_%d", i);
        event_fds[i] = open(dev_path, O_RDONLY | O_NONBLOCK);
        if (event_fds[i] < 0) {
            perror("Failed to open event device");
            continue;
        }
        if (event_fds[i] > max_fd) {
            max_fd = event_fds[i];
        }
    }

    // 2. 使用 select() 监听所有 event 文件
    while (1) {
        FD_ZERO(&read_fds);
        for (i = 0; i < MAX_EVENTS; i++) {
            if (event_fds[i] > 0) {
                FD_SET(event_fds[i], &read_fds);
            }
        }

        ret = select(max_fd + 1, &read_fds, NULL, NULL, NULL);
        if (ret < 0) {
            perror("select failed");
            break;
        }

        // 3. 检查哪个 event 触发了中断
        for (i = 0; i < MAX_EVENTS; i++) {
            if (event_fds[i] > 0 && FD_ISSET(event_fds[i], &read_fds)) {
                uint32_t event_count;
                if (read(event_fds[i], &event_count, sizeof(event_count)) > 0) {
                    printf("Interrupt on event %d, count=%u\n", i, event_count);
                }
            }
        }
    }

    // 4. 关闭所有文件描述符
    for (i = 0; i < MAX_EVENTS; i++) {
        if (event_fds[i] > 0) {
            close(event_fds[i]);
        }
    }

    return 0;
}
```



# AXI LITE寄存器

## FPGA配置

通过XDMA的AXI LITE接口连接寄存器模块，代码在`"F:\my_work\verilog_commonModule\src\regfile模块\axi_lite_regfile.v"`

## PC软件代码

1、直接通过/dev/xdma0_user 映射地址

读取0x50地址的内容

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <stdint.h>

#define XDMA_DEVICE "/dev/xdma0_user"
#define TARGET_ADDR  0x50  // 要读取的寄存器地址（16进制）
#define MAP_SIZE     4096  // 映射大小（通常 4KB）

int main() {
    int fd;
    volatile uint32_t *mapped_regs;
    uint32_t value;

    // 1. 打开 XDMA 用户设备
    fd = open(XDMA_DEVICE, O_RDWR);
    if (fd < 0) {
        perror("Failed to open XDMA device");
        return -1;
    }

    // 2. 将寄存器空间映射到用户内存
    mapped_regs = mmap(NULL, MAP_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (mapped_regs == MAP_FAILED) {
        perror("mmap failed");
        close(fd);
        return -1;
    }

    // 3. 读取 0x50 地址的数据（32位）
    value = mapped_regs[TARGET_ADDR / sizeof(uint32_t)];
    printf("Value at 0x%08x: 0x%08x\n", TARGET_ADDR, value);

    // 4. 解除映射并关闭设备
    munmap((void*)mapped_regs, MAP_SIZE);
    close(fd);

    return 0;
}
```

2、通过udma驱动进行读取

具体代码为：https://github.com/FPGAmaster-wyc/libudma 





# 不断电重新扫描PCIE

建立一个脚本文件：rescan_pcie.sh

```bash
vim rescan_pcie.sh

## 内容如下

#!/bin/sh

echo "Stop PCIE bus-0"
echo 14160000.pcie | sudo tee /sys/bus/platform/drivers/tegra194-pcie/unbind

echo "Rescan PCIE bus-0"
echo 14160000.pcie | sudo tee /sys/bus/platform/drivers/tegra194-pcie/bind

echo "Listing Xilinx PCIe devices"
lspci -v -d10ee:

#添加权限
chmod +x rescan_pcie.sh

##运行即可
sudo ./rescan_pcie.sh
```



# Linux读写xdma

## 安装xdma驱动（linux版）

### 驱动下载

可以从官方下载驱动，也可以从我这里下载我使用的版本

官方驱动下载：[GitHub - Xilinx/dma_ip_drivers: Xilinx QDMA IP Drivers](https://github.com/Xilinx/dma_ip_drivers)

本设计使用的驱动：[xdma_linux驱动下载](https://download.csdn.net/download/w18864443115/89504072?spm=1001.2014.3001.5503)

### 驱动安装

### 1、检查pci驱动

打开 Linux 终端，输入“lspci”命令并执行， 如下图所示：

![img](./media/0773fabc1c8b4c71abf04811a7b7fc35.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

可以看到，没有 Xilinx 相关的信息，这是因为没有安装 XDMA 的 Linux 系统驱动。

###  2、驱动安装

下载好的驱动，存放到Linux上面

由于提供的都是源码，需要编译安装然后才能使用。打开终端，输入如下命令：

```bash
cd ./dma_ip_drivers/XDMA/linux-kernel/xdma/
sudo apt install build-essential #（如果之前没有安装过 build-essential，需要安装）
sudo make install
```

驱动安装完成后，重启 Linux 系统主机。

### 3、加载驱动

现在打开终端，输入如下命令进入 xdma 驱动目录下的 tests 目录：

```bash
cd ./dma_ip_drivers/XDMA/linux-kernel/tests
```

该目录提供了驱动加载脚本及应用测试脚本，其中的 load_driver.sh 即为 XDMA 的驱动加载脚本。执行该脚本前需要先给该脚本赋予可执行权限，然后以 root 身份执行，命令如下：

```bash
chmod +x load_driver.sh
sudo ./load_driver.sh
```

![img](./media/f0c106e72dc5ac58ce18f29f7c837830.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

可以看到 XDMA 驱动已正确加载。

###  4、检测XDMA设备

在终端中输入“lspci”命令

![img](./media/35591b33c303b841b7cf14643bb8c8a7.png)

在终端中输入“ls /dev”命令并执行，可以在/dev 目录下看到以 xdma0 开头的设备文件，如下图所示：

![img](./media/dd7cdb7492a5d0d8178d3c12010e4baf.png)

以下是每个设备文件的简要说明：

> 1. `/dev/xdma0_control`：用于控制和配置DMA设备。
> 2. `/dev/xdma0_user`：用于用户自定义用途。
> 3. `/dev/xdma0_xvc`：用于虚拟JTAG功能。
> 4. `/dev/xdma0_events_*`：用于处理DMA事件（中断）。
> 5. `/dev/xdma0_c2h_*`：用于从卡到主机（Card to Host，简称C2H）的DMA数据传输。
> 6. `/dev/xdma0_h2c_*`：用于从主机到卡（Host to Card，简称H2C）的DMA数据传输。

这表明Linux系统已正确安装 XDMA 驱动并检测到了 XDMA 设备。

## 官方测试

打开终端，输入如下命令进入 xdma 目录下的tests 目录：

```bash
cd ./dma_ip_drivers/XDMA/linux-kernel/
cd tests/
```

需要给他们添加可执行权限：输入如下命令以执行 run_test.sh 脚本：

```bash
chmod +x run_test.sh
chmod +x dma_memory_mapped_test.sh
sudo ./run_test.sh
```

> run_test.sh 脚本用于测试基本的 XDMA 传输，并具有如下功能：
>  ✓ 检测设计是基于 AXI-MM 接口还是 AXI_ST 接口，并查看启用了多少个通道；
>  ✓ 对所有启用的通道进行基本传输测试；
>  ✓ 检查数据完整性；
>  ✓ 报告通过或失败

![img](./media/af8c4ecee7e3ed53859c748fa2e11086.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

测试结果的最后两行 passed 表明我们搭建的基于 XDMA 的 PCIe 通信子系统正确，且 XDMA 驱动安装和驱动示例程序运行正常。



Linux 的 XDMA 测试应用是源码提供的，需要先编译。打开终端，输入如下命令进入 xdma 目录下的tools 目录并编译：

```bash
cd ./dma_ip_drivers/XDMA/linux-kernel/
cd tools/
make
```

编译后， tools 目录下有四个 XDMA 相关的测试应用，如下图所示：（第一次进来没有data_rd.bin文件）

![img](./media/077373d909aeb371a91bae7764662b44.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)编辑

> **dma_to_device**
>
> - 这是一个可执行文件，用于将数据从主机内存写入到 FPGA 设备。
>
> **dma_from_device**
>
> - 这是一个可执行文件，用于从 FPGA 设备读取数据到主机内存。
>
> **performance**
>
> - 这是一个可执行文件，可能用于测试和评估 DMA 性能
>
> **reg_rw**
>
> - 这是一个可执行文件，可能用于读取和写入 FPGA 寄存器

## 写入数据

打开终端，进入tools目录，然后输入以下指令（首先准备好一个测试数据datafile4K.bin）：

```bash
sudo ./dma_to_device -d /dev/xdma0_h2c_0 -a 0x00000000 -s 2048 -f datafile4K.bin
```

这个命令将 data_to_write.bin 文件中的数据写入到 /dev/xdma0_h2c_0 设备的地址 0x00000000，长度为 2048 字节。

## 读取数据

打开终端，进入tools目录，然后输入以下指令

```bash
sudo ./dma_from_device -d /dev/xdma0_c2h_0 -a 0x00000000 -s 2048 -f data_rd.bin
```

这个命令从 /dev/xdma0_c2h_0 设备开始的地址 0x00000000 读取 2048 字节的数据，并保存到 data_rd.bin 文件中。









































