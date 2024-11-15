# 引脚约束

```tcl
set_property PACKAGE_PIN V7 [get_ports UART_0_rxd]
set_property IOSTANDARD LVCMOS33 [get_ports UART_0_txd]

## 一个指令
set_property -dict {PACKAGE_PIN R21 IOSTANDARD LVDS_25} [get_ports tx_frame_out_n]
set_property -dict {PACKAGE_PIN N17 IOSTANDARD LVDS_25} [get_ports {tx_data_out_p[0]}]
```

# 差分转单端

```tcl
IBUFDS lvdsclk (.I(clkin_p),	.IB(clkin_n),	.O(clkin));
```

# 单端转差分

```tcl
OBUFDS test (.O(clk_in_p),    .OB(clk_in_n),    .I(clk)  );
```

# ila时钟约束

```tcl
set_property C_CLK_INPUT_FREQ_HZ 300000000 [get_debug_cores dbg_hub]
set_property C_ENABLE_CLK_DIVIDER false [get_debug_cores dbg_hub]
set_property C_USER_SCAN_CHAIN 1 [get_debug_cores dbg_hub]
connect_debug_port dbg_hub/clk [get_nets lvds_clk] （lvds_clk需要修改为你ila的时钟）
```



# FLASH速度等级：

set_property BITSTREAM.CONFIG.SPI_BUSWIDTH 4 [current_design]
set_property CONFIG_MODE SPIx4 [current_design]

**说明：**

在固化FLASH的时候，就可以选择SPI x4
