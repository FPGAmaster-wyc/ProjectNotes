# 基础说明：

**zynq100板卡：**

如果使用光纤接口，那么PCIE只能使用x4

**通过FLASH启动需要对fsbl文件进行修改：**

添加一下定义在fsbl_debug.h文件：

```c++
#define FSBL_DEBUG_INFO
```

![image-20240614101734245](E:\my_work\ProjectNotes\media\image-20240614101734245.png)

# 在JTAG下载器连接时FPGA不加载flash里的程序

通过Vivado_init.tcl脚本避免情况发生

1）新建一个Vivado_init.tcl脚本，添加一下内容：

```tcl
set_param labtools.auto_update_hardware 0
```

2）将脚本放到：

installdir /Vivado/version/scripts/Vivado_init.tcl  目录下

installdir是Vivado Design Suite的安装目录。

# 取位宽函数：

```verilog
function integer clogb2 (input integer bit_depth);              
	  begin                                                           
	    for(clogb2=0; bit_depth>0; clogb2=clogb2+1)                   
	      bit_depth = bit_depth >> 1;                                 
	    end                                                           
endfunction  
```

# 修改linux为简短密码：

```shell
## user-name 改为你的用户名 
sudo passwd user-name    
```

然后直接输入新密码即可
