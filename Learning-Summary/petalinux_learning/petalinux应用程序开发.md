# 一、通过SDK开发应用程序

​	1.打开SDK，创建Linux app应用程序

​	2.编写代码

​	3.编译代码

​	4.将可执行文件拷贝到开发板根文件系统去执行（/home/root）



# 二、通过petalinux开发linux应用程序

​	1\. 在petalinux工程下，创建app工程

​		petalinux-create -t apps -n my-app --template c

​			apps： 创建的类型为app

​			my-app：应用程序的名字（不能出现“_”）

​			c : 编程语言

​	2\. 编写代码

​		产生的文件位置： v2018.3/11/my_zed/project-spec/meta-user/recipes-apps/my-app

​		进行代码编写（vim）：输入i进入编写，然后esc，输入“:wq”保存

​	3\. 编译工程

​		petalinux-build -c my-app -x do_compile

​			my-app： 编译的应用程序名

​			do_compile：进行编译

​	4\. 得到可执行文件

​		可执行文件：/build/tmp/work/cortexa9hf-neon-xilinx-linux-gnueabi/my-app/1.0-r0

​	5\. 在开发板运行

​	（1）通过scp命令（基于SSH协议），把可执行文件传递到开发板目录

​		scp my-app [root@192.168.3.108:/home/root](mailto:root@192.168.3.108:/home/root)

​			my-app： 可执行文件名

[			root@192.168.3.108](mailto:root@192.168.3.108)： 开发板ip

​			/home/root： 开发板存放可执行文件路径

​	（2）通过挂载NFS网络文件系统 ：mount

​		1\. 在虚拟机搭建nfs系统

​			sudo apt-get install nfs-kernel-server ：安装nfs系统

​			sudo vi /etc/exports ：配置nfs系统

​			内容：	/home/zynq/linux/nfs \*(rw,sync,no_root_squash)

​					   /home/zynq/linux/nfs：创建nfs的路径（根据自己的更改）

​			sudo service nfs-kernel-server restart ：重启nfs系统

​		2\. 把可执行文件拷贝到进去

​		3\. 在开发板系统，进行挂载nfs

​			mount -t nfs -o nolock 192.168.3.175:/home/hwusr/server/nfs /mnt

​			192.168.3.175:/home/hwusr/server/nfs：为虚拟机的nfs路径



# 三、通过vim开发linux

​	1\. 编写代码

​		vim test.c

​	2\. 编译代码

​		arm-linux-gnueabihf-gcc -o test test.c

​		\-o test：表示编译完成生成的可执行文件名

​	3\. 通过nfs或者scp传到开发板



