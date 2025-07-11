# 取位宽函数：

```verilog
function integer clogb2 (input integer bit_depth);              
	  begin                                                           
	    for(clogb2=0; bit_depth>0; clogb2=clogb2+1)                   
	      bit_depth = bit_depth >> 1;                                 
	    end                                                           
endfunction  
```



# 按键消抖+边沿检测

```Verilog
// 按键消抖及边沿检测
parameter DEBOUNCE_TIME = 32'd50_000;  // 1ms @50MHz (50,000 cycles)
reg key_out,key_out_dly;
reg [31:0]  key_cnt;       // 按键消抖计数器

always @(posedge clk, negedge rst_n) begin
	if (!rst_n) {key_cnt,key_out_dly,key_out} <= 1;  //0:按键高有效 1：按键低有效 
    else begin
        key_out_dly <= key_out;
        key_out <= (key_cnt == DEBOUNCE_TIME) ? key : key_out;
        key_cnt <= (key != key_out) ? ((key_cnt == DEBOUNCE_TIME) ? 0 : key_cnt + 1) : 0;
    end
end
//按键高有效使用：key_release     按键低有效使用：key_pressed
wire key_release = key_out & ~key_out_dly;  // 检测上升沿
wire key_pressed = ~key_out & key_out_dly;  // 检测下降沿
```



# 动态位宽选择

- [基址 -: 宽度] 语法：Verilog中的动态位宽选择，用于从向量中提取固定宽度的位段，起始位动态计算，**向低位选择**。

```verilog
else if (cnt_4byte <= 2) begin
    buffer8 <= buffer[23 - cnt_4byte * 8 -: 8];
end

//等同于

case (cnt_4byte)
    0: buffer8 <= buffer[23:16];
    1: buffer8 <= buffer[15:8];
    2: buffer8 <= buffer[7:0];
    default: buffer8 <= buffer8;
endcase
```

同样也有**向高位选择**的语法

- [基址 +: 宽度] 向高位选择，适合从低位开始的索引。

```verilog
assign byte_out = buffer[index * 8 +: 8];

/* 说明：
如果 index = 0，选择 buffer[0 +: 8] = buffer[7:0]。
如果 index = 1，选择 buffer[8 +: 8] = buffer[15:8]。
*/
```





# robei开发板资料

1.在线培训：http://robei/com/read.php?id=155
2.设计案例：http://robei.com/search.php
3.课程培训：http://robei.com/read.php?id=133
4.八角板：http：//robei.com/read.php?id=152
5.《数字集成电路设计》：http://robei.com/read.php?id=139
6.常见问题：http://robei.com/read.php?id=131
7.注册码：http://robei.com/read.php?id=22
8.实验箱：http://robei.com/read.php?id=157
9.高端机器人平台：http://robei.com/read.php?id=156
10.7天精英实训：http://robei.com/read.php?id=145 



# AXI开源设计

国内网站 

AXI

https://gitcode.com/gh_mirrors/ve/verilog-axi?spm=1001.2101.3001.10289&isLogin=1

PCIE

https://gitcode.com/gh_mirrors/ve/verilog-pcie?spm=1001.2101.3001.10289&isLogin=1

网络

https://gitcode.com/gh_mirrors/ve/verilog-ethernet?spm=1001.2101.3001.10289&isLogin=1

github网站（全）

https://github.com/alexforencich?page=3&tab=repositories



# AX301开发板资料

https://github.com/dta0502/AX301





# openFPGA仓库

https://github.com/suisuisi?tab=repositories

