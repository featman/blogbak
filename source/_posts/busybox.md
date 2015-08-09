title: 移植busybox到开发板   
date: 2015-4-11 14:01:01
categories: 嵌入式linux 
tags: [文件系统,busybox,移植] 

---

折腾了两天，终于把这个做好，移植的工作还是很辛苦的，目前自己对/etc/下的配置文件的逻辑关系还不是太清晰，这还需要后续的努力，一些心得，更多的是备忘，把这些东西记录下来。

<!--more-->
## 1.我的需求

由于目前的实验平台没有telnet和ftp，所以在软件调试过程中，只能用串口和SD卡来交互和传送文件。于是就想到自己定制一个文件系统，此文件系统包含这两个功能，另外由于项目的需求为一个演示系统，所以还需要在主板自带的屏幕上输出 程序的打印信息。

## 2.实验环境
1.PC机位ubuntu 12.04 , 交叉编译工具gcc version 4.4.1 (Sourcery G++ Lite 2009q3-67)

2.嵌入式主板为：DevKit 8000 

3.busybox版本为：1.13.1

4.vsftp 版本为：2.0.5
### 3.实验过程
1.首先需要编译busybox，编译的过程网上有很多的教程。telnet就再busybox中编译。

make install 之后会在 当前目前下的_install/目录下生成所需要的几个目录。
bin sbin linuxrc usr(这个我并没有编译)

2.把上边的三个文件夹放到一个新建的rootfs/下，剩下的文件我并没有自己去建。 把原来devkit8000的文件系统解压之后，我放到了rootfs下，（需要注意的是，/etc目录下的文件我没有拷贝，因为之前拷贝过很多次，会出 现各种各样的问题，所以/etc下的文件，我用的是天嵌的板子tq335x文件系统中的/etc/）

3.值得注意的是，profile和init.d/rcS中的文件要进行一些调整，以下是调整之后的。
```
/etc/profile  
```

```
# Ash profile   
# vim: syntax=sh  
  
# No core files by default  
#ulimit -S -c 0 > /dev/null 2>&1  
  
export set HOME=/root  
export set QTDIR=/opt/PDA  
export set QPEDIR=/opt/PDA  
export set QWS_DISPLAY="LinuxFB:/dev/fb0"  
export set QWS_KEYBOARD="TTY:/dev/tty1"  
if [ -f /sys/devices/virtual/input/mice/subsystem/input$ts/uevent ] ; then  
        export set TSLIB_CONFFILE=/etc/ts.conf  
        if [ -n "$pointer" ] ; then  
                export set TSLIB_CALIBFILE=$pointer  
        fi  
        export set TSLIB_TSDEVICE=/dev/event$ts  
        export set TSLIB_PLUGINDIR=/lib/ts  
        export set QWS_MOUSE_PROTO="TSLIB:/dev/event$ts MouseMan:/dev/mice"  
fi  
export set QT_PLUGIN_PATH=$QTDIR/plugins/  
export set QT_QWS_FONTDIR=$QTDIR/lib/fonts/  
export set PATH=$QPEDIR/bin:$PATH  
export set LD_LIBRARY_PATH=$QTDIR/lib:$QPEDIR/plugins/imageformats:$QPEDIR/plugins/accessible:$LD_LIBRARY_PATH  
  
#USER="`id -un`"  
#LOGNAME=$USER  
#PS1='[\u@\h \W]# '  
#PATH=$PATH  
  
#HOSTNAME=`/bin/hostname`  
  
/bin/hostname FlyingGames  
PS1="[""$(whoami)"@"$(hostname): "'$PWD'"]# "  
  
  
export USER LOGNAME PS1 PATH  

[cpp] view plaincopyprint?
/etc/init.d/rcS  
[cpp] view plaincopyprint?
#!/bin/sh  
  
  
PATH=/sbin:/bin:/usr/sbin:/usr/bin  
runlevel=S  
prevlevel=N  
umask 022  
export PATH runlevel prevlevel  
  
#  
#       Trap CTRL-C &c only in this shell so we can interrupt subprocesses.  
#  
  
mount -a  
mkdir -p /dev/pts  
mount -t devpts devpts /dev/pts  
mount -n -t usbfs none /proc/bus/usb  
echo /sbin/mdev > /proc/sys/kernel/hotplug  
mdev -s  
mkdir -p /var/lock  
  
#modprobe rt5370sta  
  
hwclock -s  
#EmbedSky_wdg &  
  
  
ifconfig lo 127.0.0.1  
net_set &  
  
/etc/rc.d/init.d/netd start  
/etc/rc.d/init.d/httpd start  
/etc/rc.d/init.d/leds start  
  
InputAdapter  
#pda &  
#/bin/hostname -F /etc/sysconfig/HOSTNAME  
  
#telnet  stream  tcp     nowait  root    /usr/sbin/telnetd       /usr/sbin/telnetd -i  
/usr/sbin/telnetd &  
/usr/sbin/vsftpd &  
  
  
insmod /usr/lib/rt3070sta.ko  
sleep 1  
```
至此，telnet已经做完。

4.接下来要做的是vsftp的移植，vsftp不同于telnetd，telnetd是在busybox自带的就有了，但是ftpd我在make menuconfig后找了好半天，都找不到有ftpd，无奈，只能用vsftp这个第三方的程序。[参考](http://blog.163.com/sunshine_linting/blog/static/44893323201391010522601/)

5.接下来要做的是把文件系统打包，拷贝到主板上。

```
/home/embest/work/tools/mkfs.ubifs -r rootfs -m 2048 -e 129024 -c 812 -o ubifs.img
/home/embest/work/tools/ubinize -o ubi.img -m 2048 -p 128KiB -s 512 /home/embest/work/tools/ubinize.cfg
cp ubi.img /media/LABEL1
```
6.把sd卡插到主板上


```
mmcinit
fatload mmc 0:1 81000000 ubi.img
nand unlock
nand ecc sw
nand erase 680000
band write.i 81000000 680000 $(file size)
//开机即可。
```
**附：开机之后在主板的屏幕上是没有任何登陆信息的。这时候需要通过登陆telnet然后启动程序。启动程序的时候记得加个重定向 。**

```
xxxx > /dev/tty0

Wifi-Module-804  /   WifiModule
```
