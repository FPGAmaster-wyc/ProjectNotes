

没有使用到的引脚，会被综合的时候优化掉，所以不用引脚分配

打开引脚分配界面，需要关闭搜狗输入法





## PLL IP配置说明

hardware I/O 必须特定的I/O管脚才能作为输入，external I/O则可以使用任意的I/O作为CCC输入



# 注意事项

输入法不能使用搜狗输入法，否则libero软件有些功能不能使用





# 下载文件报错

Error: A programming file must be loaded before running the command 'set_programming_action'. 

Error: The command 'set_programming_action' failed. 

Error: Failure when executing Tcl script. [ Line 2 ] 

Error: The Execute Script command failed.

解决办法：删除designer\impl1 下的 xx_fp文件夹，然后重新编译

# 破解说明

尽量安装在C盘

license放在C盘

**添加环境变量：**

变量名：LM_LICENSE_FILE

变量值：C:\Microsemi\License.dat