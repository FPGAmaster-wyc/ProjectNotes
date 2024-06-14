# 1、FPGA中如果使用高电平复位，always语句中要注意，要采复位上升沿

```verilog
//例如：	
always @ （posedge clk，posedge reset） begin

end
```

# 2、在FPGA例化中，输出的端口不能外接取反信号

```verilog
//例如：	
		wire locked;
		wire reset;
		
		/*错误写法*/
		get_clk  GET_CLK(
			.reset(reset),  
			.lvds_clk(LVDS_CLK), 
			.locked(!reset)
		; 
		
		/*正确写法*/
		get_clk  GET_CLK(
			.reset(reset),  
			.lvds_clk(LVDS_CLK), 
			.locked(locked)
		; 
		assign reset = !locked;
```

# 3、如果要使用PLL，不建议直接调用PLL 的 IP ，可以使用PLL的源码

# 4、Vivado常用源语

**差分转单端：**

```verilog
IBUFDS lvdsclk(.I(clkin_p),.IB(clkin_n),.O(clkin));

IBUFDS lvdsdata_1(.I(dout_p[0]),.IB(dout_n[0]),.O(LVDS_DATA1));

IBUFDS lvdsdata_2(.I(dout_p[1]),.IB(dout_n[1]),.O(LVDS_DATA2));

IBUFDS lvdssync(.I(sync_p),.IB(sync_n),.O(LVDS_SYNC));
```

**单端转差分：**

```verilog
OBUFDS test (.O(clk_in_p),    .OB(clk_in_n),    .I(clk)  );
```

**增加ila时钟约束：**

```verilog
set_property C_CLK_INPUT_FREQ_HZ 300000000 [get_debug_cores dbg_hub]

set_property C_ENABLE_CLK_DIVIDER false [get_debug_cores dbg_hub]

set_property C_USER_SCAN_CHAIN 1 [get_debug_cores dbg_hub]

connect_debug_port dbg_hub/clk [get_nets lvds_clk] （lvds_clk需要修改为你ila的时钟）
```

**绑定引脚约束：**

```verilog
set_property PACKAGE_PIN T5 [get_ports clk_in_p]         ##引脚号        

set_property IOSTANDARD LVDS_25 [get_ports clk_in_p]    ##引脚属性  
```











