# MY_驱动

**Ubuntu 22.04	arm-linux-gnueabif gcc 7.5	uboot 22.10**	**kernel 6.3.5**



## Ubuntu

### 	Ubuntu22.04_ISO：

​	阿里云镜像源：[ubuntu-releases-22.04安装包下载_开源镜像站-阿里云 (aliyun.com)](https://mirrors.aliyun.com/ubuntu-releases/22.04/)

​	百度网盘链接：[ubuntu22.04百度网盘下载 - CSDN文库](https://wenku.csdn.net/answer/6y7p3unx6e)



### 配置：

​	【VMware虚拟机安装Ubuntu22.04.2操作系统教程】https://www.bilibili.com/video/BV1pX4y1k7u1?vd_source=92221f2a9dae5263c2c9f7cbc96d7b18



### 下载VMtool：

​	ubuntu不支持本机带的vmtool工具（版本过低）。下载 open-vm-tools-desktop 即可

​	**sudo apt update -> sudo apt install open-vm-tools-desktop**

​	

### 新机上网：

​	**校园网：**电脑是无线网络的话 网络配置那块需要采用NAT，直接用桥接会无法上网。校园网一人一账号，桥接相当于多了一个账号要上网.没影响，如果需要用开发板联网，按照正点原子网络搭建的方法，网络适配器1自动桥接，ipv4手动固定（校园网就随便，wifi就处于同一网段），再添加网络适配器2NAT，ipv4自动就行，虚拟机网络编辑器那里直接还原默认配置就会弹出VM0，VM0选择开发板网线对应usb设备那里，接入网线时找到主机的网络连接那块会弹出一个以太网设备，修改ipv4网段，和网络配置1处于同一网段即可。



**ubuntu下先桥接网络，可以手动分配ip，仅对该网络上的资源使用此连接才可连接外网**

**开发板用net网络，自动分配ip，不用选择仅对该网络上的资源使用此连接**





## Uboot:

### uboot链接：

[Index of /pub/u-boot/](https://ftp.denx.de/pub/u-boot/?spm=a2c6h.12873639.article-detail.7.2c4f3c7dt45pO6)

[u-boot移植：详细讲解移植u-boot.2022.10版本到imx6ull开发板_6ull uboot2022-CSDN博客](https://blog.csdn.net/Wang_XB_3434/article/details/130794027)

[详细讲解u-boot之网络移植与调试_uboot 设备树 选择网卡-CSDN博客](https://blog.csdn.net/Wang_XB_3434/article/details/130817581?spm=1001.2014.3001.5502)

在uboot命令行里，开发板可以ping虚拟机和主机，但是主机和虚拟机ping不了开发板

使用fec2  ethphy1 	**uboot的defconfig要使能 CONFIG_PHY_REALTEK  uboot命令行内网卡一和网卡二都要设置**

要修改设备树的配置，网卡信号口会闪烁灯



### 编译：

创建一个 shell 脚本	build.sh	chmod 77 my.sh 给予权限

 #!/bin/bash

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
​make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- mx6ull_ykj_defconfig
​make V=1 ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4

​	./build.sh

**编译成功后会生成uboot.bin文件**

在Makefile文件中 **276 行** 添加

ARCH ?= arm

CROSS_COMPILE ?= arm-linux-gnueabihf-

​	**make V=1 -j4**



### 烧写：

需要 **imxdownload** 下载文件	chmod 777 imxdownload 给予权限

####  SD卡：

sudo fdisk -l 查看对应的dev设备号

​	./imxdownload u-boot.bin /dev/sdb





## 内核：

[移植Linux 6.3.5系统到imx6ull开发板_imx6 linux版本-CSDN博客](https://blog.csdn.net/Wang_XB_3434/article/details/131154571?spm=1001.2014.3001.5502)

[IMX6ULL Linux内核网络驱动修改 - 其实我只是懒 - 博客园 (cnblogs.com)](https://www.cnblogs.com/Hlc-/p/18200017)

查看自己的 imx6ull-ykj.dts	去修改所处包含的头文件

注意网卡型号 **imx_ykj_defconfig** 配置网卡型号	**CONFIG_REALTEK_PHY=y**

要修改设备树的配置，进入根文件系统时, 网卡信号口会闪烁灯

**arch/arm/configs:** 			配置文件 .defconfig

**arch//arm/boot/dts/:**	  设备树.dts文件

**arch/arm/boot/：**			 镜像文件zImage



bootcmd=tftp 80800000 zImage; tftp 83000000 imx6ull-ykj.dtb; bootz 80800000 - 83000000



### 编译：

创建一个my.sh文件	给权限 chmod 777 my.sh

#!/bin/sh

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
​make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- imx_ykj_defconfig
​make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4

​	 **./my.sh**



在底层Makefile文件中 **393行** 添加

ARCH ?= arm

CROSS_COMPILE ?= arm-linux-gnueabihf-

​	**make -j4 V=1**



编译成功后会在 **arch/arm/boot/zImage**文件夹下生成 **zImage** 文件，相应的对应的dtb文件也会编译出来 **arch/arm/boot/dts/** **.dtb** 

单独编译设备树时   make dtbs	其他的包括菜单配置 make -j4



### 烧写：

​	**tftp启动时：**

​		cp arch/arm/boot/zImage ~/my_linux/tftpboot/ -f

​		cp arch/arm/boot/dts/imx6ull-ykj.dtb ~/my_linux/tftpboot/ -f

​		uboot命令行:	setenv bootcmd ‘tftp 80800000 zImage; tftp 83000000 imx6ull-ykj.dtb; bootz 80800000 - 83000000’	







## 根文件系统：

### buidroot 21.2.6

bootargs=console=ttymxc0,115200 root=/dev/nfs nfsroot=192.168.10.100:/home/ykj/my_linux/nfs/buildrootfs,proto=tcp rw ip=192.168.10.50:192.168.10.100:192.168.10.1:255.255.255.0::eth0:off



每次编译后生成的压缩包在 **/output/image**	将压缩包拷贝到nfs



#### 具体过程：

​		[驱动程序开发：Buildroot根文件系统构建并加载驱动文件xxx.ko测试_make busybox-menuconfig-CSDN博客](https://blog.csdn.net/morecrazylove/article/details/129257764)

​		[Buildroot根文件系统构建_buildroot制作uboot怎么加载文件系统到emmc-CSDN博客](https://blog.csdn.net/lengyuefeng212/article/details/119848347)



![image-20241022152415304](C:/Users/86135/AppData/Roaming/Typora/typora-user-images/image-20241022152415304.png)

![image-20241022152435354](C:/Users/86135/AppData/Roaming/Typora/typora-user-images/image-20241022152435354.png)





#### buildroot添加ssh：

​	[buildroot添加ssh功能_buildroot ssh-CSDN博客](https://blog.csdn.net/qq_45668408/article/details/138668218)

​	当QT编译的文件要拷贝给开发板时，远程主机需要有ssh



#### QT：

##### 编译工程：	

​	先启动交叉编译器：  source /opt/fsl-imx-x11/4.1.15-2.1.0/environment-setup-cortexa7hf-neon-poky-linux-gnueabi（将x86架构	进行ARM交叉编译）

​	在工程编译： qmake->amke -j4

​	再拷贝到远程主机： scp xxx(编译出来的工程)root@192.168.10.50(远程主机IP):/root

​	



## 软件包：

### 	ifocnfig:

​	sudo apt install net-tools

### VI/VIM:

​	sudo apt install vim

### gcc:

​	sudo apt install gcc

### FTP服务（互传文件）：

​	sudo apt-get install vsftpd

### NFS网络挂载：

​	sudo apt-get install nfs-kernel-server rpcbind

### TFTP网络挂载：

​	sudo apt-get install tftp-hpa tftpd-hpa

​	sudo apt-get install xinetd

### SSH远程连接终端：

​	sudo apt-get install openssh-server





## Problem:

### uboot启动环境变量警告：

​	env default -a			清除之前的变量

​	saveenv

### tftp 启动失败：

​	缺少软件包：

​	**ykj@ykj-ubuntu:/etc/xinetd.d$ sudo service tftpd-hpa start**

​	**Failed to start tftpd-hpa.service: Unit tftpd-hpa.service not found.**



**sudo apt-get install tftp-hpa tftpd-hpa**
**sudo apt-get install xinetd**



### 内核编译defconfig失败：

​	ykj@ykj-ubuntu:~/linux/IMX6ULL/linux/linux-6.3.5$ make imx_ykj_defconfig

​	***

​	*** Can't find default configuration "arch/x86/configs/imx_ykj_defconfig"!

​	***

​	make[1]: *** [scripts/kconfig/Makefile:94：imx_ykj_defconfig] 错误 1
​	make: *** [Makefile:695：imx_ykj_defconfig] 错误 2
​	ykj@ykj-ubuntu:~/linux/IMX6ULL/linux/linux-6.3.5$ make imx_ykj_defconfig

​	***

​	*** Can't find default configuration "arch/x86/configs/imx_ykj_defconfig"!

​	***

​	make[1]: *** [scripts/kconfig/Makefile:94：imx_ykj_defconfig] 错误 1
​	make: *** [Makefile:695：imx_ykj_defconfig] 错误 2

**他没去arch/arm下面找而是去arch/x86文件去找，要设置环境变量ARCH值为arch**

export ARCH=arm

export CROSS_COMPILE=arm-linux-gnueabihf-

[Can‘t find default configuration “arch/x86/configs/100ask_imx6ull_defconfig“!----编译内核时找不到配置文件_can't find default configuration-CSDN博客](https://blog.csdn.net/u013171226/article/details/132754283)



### nfs一挂载就直接TTT：

​	[在uboot下使用nfs下载失败，一直“Loading: T T T T”-CSDN博客](https://blog.csdn.net/qq_68610482/article/details/129270343)



### nfs下载镜像报错File lookup fail、“TTTTTTTTTTTTTTT”：

​	[nfs下载镜像报错File lookup fail、“TTTTTTTTTTTTTTT”-CSDN博客](https://blog.csdn.net/qq_41709234/article/details/123160029)



### 编译根文件系统Try passing init= option to kernel：

Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance.
[  100.403488] ---[ end Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance. ]---



首先要**make defconfig -> make ->  make install CONFIG_PREFIX=/home/ykj/linux/nfs/rootfs/** 将文件安装到nfs目录下,nfs启动根文件系统



[  100.389127] Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance.
[  100.403363] ---[ end Kernel panic - not syncing: No working init found.  Try passing init= option to kernel. See Linux Documentation/admin-guide/init.rst for guidance. ]---





### nfs无法下载文件：

​	[嵌入式Linux开发——解决uboot无法使用nfs服务从ubuntu中下载文件（TTT、cannot mount等错误）-CSDN博客](https://blog.csdn.net/weixin_56646002/article/details/127388021)



### 加载根文件系统 VFS: Unable to mount root fs via NFS

[解决Ubuntu 22.04不支持nfs 2服务导致开发板挂载失败的问题_ubuntu不支持nfs2-CSDN博客](https://blog.csdn.net/weixin_43501534/article/details/133935539)

ubuntu17以上nfsnfs默认只支持协议3和协议4，而kernel中默认支持协议2.

可以降低ubuntu版本



### 根文件系统can't open /dev/tty4: No such file or directory can't open /dev/tty2: No such file or directory can't open /dev/tty3: No such file or directory：

​	是可以进入根文件系统，只是被输出信息覆盖了，添加点文件就行

​	[linux根文件系统编译和移植过程-CSDN博客](https://blog.csdn.net/q1449660223/article/details/108228645)



### 编译内核设备树弹出一堆选项：

​	[imx6ull开发板在ubuntu命令行中make dtbs时，会出现很多选项 xx yes no解决-CSDN博客](https://blog.csdn.net/jacobi6/article/details/131926856)

​	没添加顶层Makefile  交叉编译器



### 在buildroot编译QT error: ‘numeric_limits’ is not a member of ‘std’：

​	是QT源码缺少头文件

​	/output/build/qt5base-5.15.2/src/corelib/text/qbytearraymatcher.h  在该文件处添加 \#include <limits>



### qt.qpa.plugin: Could not find the Qt platform plugin "eglfs" in ""This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.Available platform plugins are: linuxfb, minimal, offscreen,vnc.Aborted：

​	手动添加 export QT_QPA_PLATFORM=linuxfb:tty=/dev/fb0

​	在 etc/profile 最后一行添加



### Linux下scp命令出现Permission denied, please try again：

​	[Linux下scp命令出现Permission denied, please try again_linux scp permission denied, please try again.-CSDN博客](https://blog.csdn.net/ZPeng_CSDN/article/details/103319158)



### 根文件系统内不会显示文件路径 ：

在/etc/profile 添加

PS1='[\u@\h]:\w$:'

export PS1

并且注释下面语句



### buildroot 编译Incorrect selection of kernel headers: expected 4.0.x, got 4.10.x

​	[buildroot 编译出错_incorrect selection of kernel headers: expected 4.-CSDN博客](https://blog.csdn.net/qq_32605451/article/details/104003846)

  /usr/local/arm/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/arm-linux-gnueabihf/libc/usr/include/linux 下的 version.h 	转换值



## 链接：

### gcc交叉编译器官网：

​	[Linaro Releases](https://releases.linaro.org/components/toolchain/binaries/)

### uboot官网：

​	[Index of /pub/u-boot/ (denx.de)](https://ftp.denx.de/pub/u-boot/?spm=a2c6h.12873639.article-detail.7.2c4f3c7dt45pO6)

### linux官网：

​	[Linux 内核下载地址 - MyCPlusPlus - 博客园 (cnblogs.com)](https://www.cnblogs.com/zoneofmine/p/13062594.html)

### BusyBox官网：

​	[BusyBox](https://busybox.net/source.html)

### Buildroot官网：

​	[Index of /downloads/ (buildroot.org)](https://buildroot.org/downloads/)





## Git:

### 安装：

​	**采用阿里云镜像网速更快**：	[CNPM Binaries Mirror](https://registry.npmmirror.com/binary.html?path=git-for-windows/)

​	采用vi编辑的话，傻瓜式安装即可



### 命令：

#### **创建一个版本库**：

​							   git init

初始化个人信息：git config --global user.name "Kaijing Yang"   

​							  git config --global user.email 267056851@qq.com

查看配置：			git config --global --list (自己添加的)		git config -l（系统的）

#### **查看库的状态**：		

​									git status

#### **保存到暂存区**：		

​									git add xxx/. (.是该目录下的全部文件)

#### **提交到仓库**：

​									git commit -m “xxx” 		(提交暂存区的全部文件)	

​									git commit xxx -m “xxx”（指定提交xxx文件）

#### **查看版本记录**：		

​									git log（显示全部信息）	

​									git log --pretty=oneline（精简显示信息）

#### **回退版本**：

​									git reset --hard HEAD^ 	(回退一个版本 多少个^表示回退多少个版本)

​									git reset --hard HEAD~1（ ~后的数字表示回退几个版本）

#### 回退之前的版本：

​									git reset --hard ceaa0de8cd (哈希值)

**查找之前回退后的版本**：

​									git reflog 

#### 丢弃工作区的改动：

​									git checkout -- xxx

​	当删除了文件想要复原时：rm xxx -> git status((红色)删除：xxx) -> git checkout -- xxx 即可复原

​	要删除文件时：					rm xxx -> git status((红色)删除：xxx) -> git rm xxx -> git status((绿色)删除：xxx) -> git commit -m "xxx"

#### 对比文件不同：

​									git diff HEAD -- xxx

​									git diff HEAD HEAD^ -- xxx

#### 切换分支：

​	查看分支：			git branch

​	切换新的分支：	git checkout -b xxx

​	切换原来的分支：git checkout xxx

#### 合并分支：

​									git merge xxx

#### 删除分支：

​									git branch -d xxx

#### 分支冲突：

​	当多个分支修改并提交了同一个文件会期=起冲突，需要手动修改多余的内容，并重新提交



### 上传到github：

​	先获取ssh：	ssh-keygen -t rsa -C "2670568531@qq.com"   在.ssh目录下 cat id_rsa.pub, 内容复制到github ssh下

​	上传到仓库： git remote add origin git@github.com:yangkaijing/git-test.git

​							git push -u origin main

