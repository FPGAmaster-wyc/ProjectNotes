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

