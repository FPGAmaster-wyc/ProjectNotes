# 引脚约束

```tcl
set_property PACKAGE_PIN V7 [get_ports UART_0_rxd]
set_property IOSTANDARD LVCMOS33 [get_ports UART_0_txd]

## 一个指令
set_property -dict {PACKAGE_PIN R21 IOSTANDARD LVDS_25} [get_ports tx_frame_out_n]
set_property -dict {PACKAGE_PIN N17 IOSTANDARD LVDS_25} [get_ports {tx_data_out_p[0]}]

## 时钟约束
create_clock -period 5.000 -name ddr_clk -waveform {0.000 2.500} [get_ports i_sys_clk_p]
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





# XDC约束

**设置端口的差分终端属性DIFF TERM**

假如时钟信号和FPGA之间没有加入电阻，需要进行DIFF TERM，作用是启用差分终端电阻

set_property DIFF_TERM <true> [get_ports <ports>]

**启用差分终端：** 当你在 FPGA 中设计使用差分信号的接口时（例如 LVDS、TMDS 等），这些信号需要差分终端电阻来确保信号完整性，减少反射。使用 `set_property DIFF_TERM true` 可以在端口上启用这一特性。

**改善信号质量：** 对于高速差分信号，启用差分终端可以优化信号传输，减少信号丢失和反射，保证信号到达接收端时的质量。





# FLASH速度等级：

```shell
set_property BITSTREAM.CONFIG.SPI_BUSWIDTH 4 [current_design]
set_property CONFIG_MODE SPIx4 [current_design]

set_property CFGBVS VCCO [current_design]          #当 CFGBVS 连接至 Bank 0 的 VCCO 时，Bank 0 的 VCCO 必须为 2.5V 或 3.3V
set_property CONFIG_VOLTAGE 3.3 [current_design]   #设置CONFIG_VOLTAGE 也要配置为3.3V
set_property BITSTREAM.GENERAL.COMPRESS true [current_design]  #设置bit是否压缩
set_property BITSTREAM.CONFIG.CONFIGRATE 33 [current_design]   #设置QSPI的加载时钟
set_property BITSTREAM.CONFIG.SPI_BUSWIDTH 4 [current_design]  #设置QSPI的位宽
set_property BITSTREAM.CONFIG.SPI_FALL_EDGE Yes [current_design]  #设置QPSI的数据加载时钟边沿
set_property CONFIG_MODE SPIx4 [current_design]
```

**说明：**

在固化FLASH的时候，就可以选择SPI x4
