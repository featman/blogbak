title: OpenWRT编译之360路由器C301    
date: 2014-10-16 14:01:01
categories: 嵌入式linux 
tags: [uboot定制,内核编译,打印机共享,openwrt] 

---

备忘录分为三部分来描述，尽量做的详细一些，也算是给后人的一个参考。
一.刷写第三方的uboot（刷不死的u-boot）

二.编译内核（定制基础功能）

三.OpenWRT挂3G网卡（华为E3276）

<!--more-->
## 一.刷写第三方的uboot（刷不死的u-boot）
今天拿到手的360 c301，在官方系统的uboot上直刷 第三方 openwrt 。
按住复位键，然后上电，10秒后待前边的灯 闪烁后 即表示 进入了uboot
然后主机端 Ip要设置成192.168.1.x网段，这时候登陆web界面192.168.1.1就可以进入了官方自带的uboot，然后选择刷入 

```
//广告: 981213 大神的360路由全功能openwrt rom链接 http://www.right.com.cn/forum/thread-147319-1-1.html
```
我用的是flash1的。刷好之后，然后固件就成为了openwrt，这时候其实已经可以用了，但是为了以防万一，
在之后的操作过程中，可能会损坏uboot，所以采用第三方的刷不死的Uboot, 这样自己编译的内核或者其他第三方的固件都
随便刷了。
之后：
> 1.进入openwrt WEB界面把初始化的密码改成自己的    
2.然后用winSCP客户端的SCP命令把我们提前下好的第三方刷不死uboot（http://www.right.com.cn/forum/thread-136444-1-1.html）传入路由器  
3.用ssh登陆，并在/tmp目录下运行mtd write -r /tmp/u-boot.bin u-boot 命令刷入boot，这时路由器会自动重启    
4.待重启好后即刷好了   
5.以后所有的更新固件的操作，都要通过这个第三方的u-boot来做。（按下RESET键，上电等待10秒，再进入192.168.1.1,刚测试下，貌似只能用自带的网线。。。。。还有要注意的事，在更新的过程中，一定要用firefox或者chrome）

## 二.编译openwrt

#### 2.1    搭建编译环境
我这边的环境是VMware下的ubuntu12.04，网上有说ubuntu版本最好用32位，64位可能有问题。另外需要apt-get一些基础的库和软件，如果缺少这一步，可能编译的过程中会报错，有可能缺少工具，如git，也有可能缺少库、头文件。
```
sudo apt-get install gcc g++ binutils patch bzip2 flex bison make autoconf gettext texinfo unzip zip unrar p7zip p7zip-rar p7zip-full sharutils subversion libncurses5-dev ncurses-term zlib1g-dev gawk git-core  libpcre3-dev libssl-dev

```

#### 2.2    首先down下一份源码（分为两种，一种是trunk版，即开发版，另外一种是Barrier Breaker，即稳定版）
由于我的这个C301官方支持只有trunk版，所以down的这个版本的，down分为两种，一种是svn，另外一种是git。

```
svn co svn://svn.openwrt.org/openwrt/trunk/
//接着到trunk目录下 
./scripts/feeds update -a      //据说这部目录是更新package目录
./scripts/feeds install -a      //安装package索引文件，这些文件个人感觉只是安装了一个索引，等编译的时候还要按需（make menuconfig）从指定地点下载
```
#### 2.3 接着就是配置了，这一步个人也许是整个编译的核心了，你所需要的功能都要在这步去做好。
```
 //在源码目录下敲
    make menuconfig 
```
(测试发现，第一次make menuconfig 必须用普通用户，不能是root，所以如果是root下载的源码，记得要把整个源码目录chmod 777 \*/\* -R下，然后切换到普通用户，make menuconfig，以后再make menuconfig就不用普通用户了，root也可以的！)

***以下是我从百度文库摘的极路由1S*，但是都大同小异，一些基础的配置按照这个来就够。**
```
选择LuCI 配置（web网页管理程序）：
LuCI  ---> 1. Collections  --->  luci                                         启用LuCI
LuCI  ---> 3. Applications  ---> luci-app-commands                         网页Shell 
LuCI  ---> 3. Applications  ---> luci-app-ddns                                动态域名
LuCI  ---> 3. Applications  ---> luci-app-firewall                              防 火 墙
LuCI  ---> 3. Applications  ---> luci-app-mwan3                              网络叠加
LuCI  ---> 3. Applications  ---> luci-app-multiwan                            网络叠加
LuCI  ---> 3. Applications  ---> luci-app-ntpc                           时间同步服务器
LuCI  ---> 3. Applications  ---> luci-app-ocserv                           VPN Server
LuCI  ---> 3. Applications  ---> luci-app-qos
LuCI  ---> 3. Applications  ---> luci-app-samba                               网络共享
LuCI  ---> 3. Applications  ---> luci-app-upnp
LuCI  ---> 3. Applications  ---> luci-app-wol                                  网络唤醒
LuCI  ---> 4. Themes  ---> luci-theme-bootstrap                             默认主题
LuCI  ---> 4. Themes  ---> luci-theme-openwrt                          openwrt主题
LuCI  ---> 5. Translations  --->  luci-i18n-chinese                            支持中文
大家可以自行增减插件，保存设置后，重新编译下即可。
 
文件系统、SD卡、USB配置：
Base system  --->  block-mount                                              USB挂载
Kernel modules  --->  Filesystems  --->  kmod-fs-ext4              支持ext4文件系统
Kernel modules  --->  Filesystems  --->  kmod-fs-nfs                支持NFS文件系统
Kernel modules  --->  Filesystems  --->  kmod-fs-nfs-common
Kernel modules  --->  Filesystems  --->  kmod-fs-ntfs              支持NTFS文件系统
Kernel modules  --->  Filesystems  --->  kmod-fs-vfat             支持FAT32文件系统
Kernel modules  --->  Native Language Support  --->  kmod-nls-cp437       支持中文
Kernel modules  --->  Native Language Support  --->  kmod-nls-iso8859-1   支持中文
Kernel modules  --->  Native Language Support  --->  kmod-nls-utf8         支持中文
Kernel modules  --->  Other modules  --->  kmod-mmc                     支持SD卡
Kernel modules  --->  Other modules  --->  kmod-sdhci                     支持SD卡
Kernel modules  --->  Other modules  --->  kmod-sdhci-mt7620            支持SD卡
Kernel modules  --->  USB Support  --->  kmod-usb-ohci                 支持USB 1.0
Kernel modules  --->  USB Support  --->  kmod-usb-storage
Kernel modules  --->  USB Support  --->  kmod-usb-storage-extras
Kernel modules  --->  USB Support  --->  kmod-usb-uhci                 支持USB 1.1
Kernel modules  --->  USB Support  --->  kmod-usb2                     支持USB 2.0
 
网络配置：
Network  --->  ppp-mod-pppoe                                           PPPOE拨号模式
Network  --->  ppp-mod-pptp                                                 VPN客户端
Network  --->  SSH  --->  openssh-client                                    SSH客户端
其实里面有很多软件，大家可以编译试一试的，番茄软件很多的哦！~
 
应用程序配置：
Utilities  --->  bzip2                                                            解压缩工具
Utilities  --->  Compression  --->  unrar                                       解压缩工具
Utilities  --->  Compression  --->  unzip                                       解压缩工具
Utilities  --->  Compression  --->  zip                                           压缩工具
Utilities  --->  Filesystem  --->  badblocks                             支持ext2文件系统
Utilities  --->  Filesystem  --->  e2fsprogs                 支持ext2/ext3/ext4格式化工具
Utilities  --->  disc  --->  fdisk                                                   分区工具
```

除了上边的配置，以下我标注几个对自己需求比较重要的，其他的省略

第一个Target System ，这里边就是要选你的路由器所对应的cpu的型号。如果路由器有两颗芯片，以第一个cpu为主，C301路由器主芯片为AR9344，所以我选择Atheros AR7XXX / AR9XXX,这个可能要靠一些经验了，我对硬件的芯片区分的也不是很好，最好去相应的论坛找找路由器对应的芯片类型。

第二个Subtarget，选择Generic即可。这一步我不是特别明白，这一选项跟你上边的targetsystem有关，如果换一个芯片的话，这里可能还要选择对应的版本，目前尚不清晰。
    
第三个target profile 选择路由器型号。一般比较通用。
    
> （我down了一份BB版的源码，在这里找qihooC301没有找到，所以就是不支持，后来算下日子才知道，恩山的HACK大哥9月份才提交的patch，正式被官方录用。bb是7月出来的，肯定没有了！这里啰嗦几句，什么Kamikaze ，backfire ，attitude adjustment， barrier breaker 和trunk都是什么意思，之前我也模糊，后来去官方wiki，https://dev.openwrt.org/wiki/GetSource，感觉标注跟linux一贯的风格差不多，ubuntu两年一个正式版，openwrt最近也是两年，但时间并不确定，所以这样来看我也就清楚了。前面的这几个都是openwrt的正式版本，从老版本到最新版本，都是稳定版。后边的这个trunk是开发版，也就是功能涵盖barrier breaker，但是还在慢慢的添加新功能。所以如果你不是特意的获取trunk的老版，你获取到的永远是最新的，trunk官方源码一两天就更新一次,更新速度特别快，可以用svn log 网址来看下）

这几个大选项 global build settings ,advaced configureation options , build the openwrt image builder ,build the openwrt sdk ,package the openwrt-based toolchain ,image configuration 目前我还没用到，所以不清楚具体含义和用法。

  在大项Base system下，基本上没有动，还是按照op默认的配置来做的，值得要说的是，这里边的busybox，大家都知道busybox是一个嵌入式的linux shell集成工具，嵌入式的linux基本上所有的命令都是出自这里边（我所说的当然不包括一些第三方的package软件包的命令了）。如果有需求，可以进去打开一些命令，比如lsusb就可以在这里边打开，不然默认是没有的，除非你在其它的package安装usbutils（在大项utils下）。

    Administrator默认。

    Boot Loaders 默认。

    Devlopment 默认。

    Firmware 默认。
    
大项Kernel modules，这里边的选项，个人感觉是最重要的一个了，有很多功能如果这里边不选择好，那么将会非常麻烦，以后的固件可能很多功能用不了。结合我自己的C301进行定制。对于需要的功能。首先是无线，C301是双频的wifi，有2.4G和5G，分为两个网卡。所以在这里边的Wireless Drivers把Kmod-ath10k和kmod-ath9k，kmod-ath9k-common都要选上！因为C301自带一个usb口，所以我们进入usb support,把kmod-usb-core kmod-usb-ohci kmod-usb-storage-extras kmod-usb-uhci kmod-usb2都选上。在network support 下，把ppp的选项都选上，不然 无法拨号上网。由于360这个路由器还有一个LED，所以还要选下LED modules，我选的是gpio defaulton ledtrig gpio timer usbdev，这几项选上，我这边led显示是正常的，不过跟官方的那种亮不一样，我这个是一闪一闪的，不过感觉还可以！！！！内核在结合上边的就差不多了。

还有比较重要的两个，这两个是个性化 定制的东西，一般的package小软件，都在这两个下面。功能非常丰富而且强大！
按照我目前的需求来的，我用到了network下的File transfer下的lftp，这是一个ftp的客户端，我这需求类似的可能比较小。还有SSH这里边我全选了。VPN也可以选。。。至于utilities里边，我用到了usbutils和usb-modeswitch（挂3G网卡）

至此。配置结束了。

> 接下来还有很重要的一步才可以编译。这也算是openwrt官方Trunk的一个BUG（至少在现在14年10月份没有解决呢），估计以后会修复的。
就是如果配置结果make，然后编译好的固件，你刷上去是没有无线的（2.4G和5G都没办法启动），所以需要替换掉一个目前源码工程目录下的mac80211下的东西。具体做法：
在现在用的trunk源码目录外新建一个文件夹testop，然后进入testop，down一份之前的代码。svn co -r42681 svn://svn.openwrt.org/openwrt/trunk，之所以用这个版本，是因为这个版本的的2.4G是正常的，（也就是说5G不正常，所以不能用，但现在wifi大多数还是2.4G的！），down下来之后，然后把test源码目录下的pakcage/kernel/mac80211/Makefile和pakcage/kernel/mac80211/patches替换到之前的trunk源码的package/kernel/mac80211下。

这时配置才算真正结束。

make V=99 或者 make j=cpu核数+1 都可以，就执行编译了。
编译完之后会在源码目录下的bin下生诚ar7XXX的文件夹，打开之后就会看到我们的固件和packages包，这个包是好东西，因为和你的路由器是配套的，里边的软件你路由器可以直接安装。并不是所有ipk结尾的文件都能在路由器上安装，也并不是同一个cpu平台的通用，也可以说同型号的路由器ipk都可能不通用。所以这个文件夹的packages还是很有用的，保存好，之后做一个局域网软件源，从路由器直接到这里下载软件就可以了。。

## OpenWRT挂3G网卡（华为E3276）

首先要说的是，华为的大部分3G网卡都支持linux，也就是说openwrt只要是配置好，是完全能够支持的（当然你需要有一个usb口）.

我们需要几个东西，PPP、usb-serial、usb-modeswitch,与此同时，usb的一些选项也要加进去，例如USB2.0、usb1.0、usb-storage等。
我们所需要的是在/dev/目录下看到ttyUSB0和ttyUSB1，虽然linux驱动能够识别3G网卡，但是如果想让其工作，还需要使之拨号，这就需要ttyUSB0这个接口了，我们可以把她想象成以太网口，例如eth1。原来广域网口wan出去是从eth1走的，现在wan出去是通过ttyUSB0。另一方面，据说在BB版本（openwrt 14.07）没有出现之前，我们即便装了usb-serial也不能看到/dev/下有ttyUSB,还需要设置才可以（老版本的设置请参照下边的某老外论坛上的参考文献，另外为了识别你网卡产品的Device ID,你在openwrt下需要一个命令，lsusb，这个命令你在编译openwrt的时候可以在busybox里选上，也可以勾选上在Utilities/下的USButils）。话说回来，现在的BB版和trunk版就方便多了，直接在内核中选上usb-serial就可以看到/dev/ttyUSB0和/dev/ttyUSB1了。

这几个东西具体的位置，usb-serial在内核目录下的USB support下，值得一提的是，你需要把Kmod-usb-serial-option和kmod-usb-serial-wwan、还有mod-us-serial-ipw选上。ppp在network模块下，一般这个都会编的。。。usb-modeswitch在Utilities下。USB的一些常用选项都在内核目录下的USB SUPPORT下。


配置好了。然后就可以编译进固件中了。

至于启动3G拨号。有两种方式，一个是shell中的操作，一个是web页面的操作。我用的是web上的操作，在网络，接口，找到wan，点击修改。
然后在协议中找到UMTS/GPRS/EV-DO这一项。点击切换协议，然后在调制解调器节点 选择/dev/ttyUSB0,服务类型选择UMTS/GPRS (即3G/2G网)其他的留空就可以直接点保存应用了。过一会就看到ISP给分配的ipv4地址了。ping下外网是可以通的。

切换到原来的上网模式，选择原来的模式即可，我这边的环境是dhcp自动分配的，我就选择dhcp自动获取ip。然后点击保存会提示物理的端口需要选择，点击物理设置，桥接接口 不选，直接选中接口中的eth1即可，因为eth1对应于路由器上的wan口。


至此3G配置结束！
副：路由器如果只有一个USB口，请慎用USB-HUB，经本人实际测试，插USB-HUB确实可以使用，3G网卡，U盘，都可以用，这几天一直有个ssh的连接在用，我发现ssh总是无故断掉，找了好半天原因找不到。ps发现pppd这个进程总是断掉重启，所以我用ping 外网ip的方式发现，一分钟到两分钟就会使3G网断掉，于是我怀疑是接了USB-HUB的原因，我果断把USB-HUB撤掉，直接插在路由器的USB接口上，在测试ping外网，发现ping完全正常，目前已经ping了4000秒左右了，没有出现网络断开。所以可以确定是USB-HUB导致3G网卡供电不足。慎用！！！如果确有需求，推荐X宝买一款带供电的hub。


## 四.对编译的问题和总结

#### 4.1    在编译的过程中难免会出现一个问题，比如说，我们down的这份源码中，make menuconfig没有这个某个软件怎么办？很多人都是通过刷好固件opkg install xx解决，但是经实验证实，很难下载成功，或者下载成功安装不上，具体原因还我并不清楚。。所以我在想有些软件，我这个trunk版本没有，以前的trunk版本有没有？整个trunk版本都没有，那么BB正式版有没有？
解：假如我们down一份之前的代码或者正式版代码，在源码工程目录下，linux下可以用find . -name xxx来找你想要的软件包（当然名字要首先明确下来），这时候如果找到的话，一般来说这个文件夹都是一个索引文件（目前个人的感觉，对源码目录及结构细节和Makefile的组织并没有深究过），里边会有Makefile和子文件夹file，file中又有几个文件。把这个软件的文件夹（一般路径不确定，比如feeds/packages/net/autossh）拷贝到我们自己的源码目录下的package下（这个路径是明确规定的，不能是其他的），所以按照这种思路，以后就不是去满世界找ipk了，自己编译一个更加方便，而且肯定兼容！

然后有两个方式来完成你想要的功能，主要看你的需求，第一个是直接编译出ipk文件，另外一个是编译出新的包含此软件功能的固件。
```
第一个方式用几个指令即可：
说先解释下 即为你拷贝来的软件包。
make package//prepare V=99 
//这个命令是根据软件中的Makefile中的路径去指定URL下载软件包源码，并放在dl目录下，具体解压到哪里还没有研究- -！
//敲完这个命令会有个警告，告诉你重新make menuconfig一下。因为这时候menuconfig就可以看到新的功能了哦
//一般新的功能都在图形配置界面的network下或者utilities下，仔细找找可以看到。
make package//compile V=99   
//这个是编译命令，编译完之后就可以在bin目录的相应位置找到，但是我发现目录下会多出好多个ipk
//可能其中还有关联关系，就是可能生成很多个ipk，可能光安装一个不能安装上。。安装的过程没测试过。


```


```
//另外提供一些命令，这些命令没有完全用过。（摘自官方wiki）
Troubleshooting

If you find your package doesn’t show up in menuconfig, try the following command to see if you get the correct description:
    TOPDIR=$PWD make -C package/ DUMP=1 V=99
If you’re just having trouble getting your package to compile, there’s a few shortcuts you can take. Instead of waiting for make to get to your package, you can run one of the following:

make package//clean V=99
make package//install V=99
Another nice trick is that if the source directory under build_dir/ is newer than the package directory, it won’t clobber it by unpacking the sources again. If you were working on a patch you could simply edit the sources under the build_dir// directory and run the install command above, when satisfied, copy the patched sources elsewhere and diff them with the unpatched sources. A warning though - if you go modify anything under package/ it will remove the old sources and unpack a fresh copy.
Other useful targets include:

make package//prepare V=99
make package//compile V=99
make package//configure V=99
```
用以上这些命令的前提是，你已经搭建好了一个完整的编译环境（就是说你make过固件，不然没有交叉编译工具，你编译不了软件）

第二种方式是，你用完
```
make package//prepare V=99
```

命令后menuconfig中已经可以看到你所需要的软件了，直接make V=99 编译新固件！


#### 4.2 对于定制文件系统


在源码目录下的package/base-files/files下就是openwrt的文件系统。这里只有简单的几个，其他的一些目录都是在编译的时候生成的。
本人实际测试，发现可以在这个目录下任意位置添加自己的配置文件或者脚本，编译固件的过程中，编译器会把这个目录全部拷贝走的。
如果需要开机启动脚本，在etc/rc.local中添加，但要记得如果调用你的脚本是一个while无限循环的，那么需要在脚本名的后边加后台运行，即&
不然会导致exit 0无法执行，到时候reboot这些命令不可用。
openwrt大部分基础文件都在软件包base-files里面。这个软件包做3件事：

1 复制package/base-files/files下的所有文件到根目录

2 复制target/linux/对应平台/base-files下的所有文件到根目录

3 复制源码根目录下的files文件夹下的所有文件到根目录（如果需要添加自己的文件那么可以通过这一步，即在源码根目录内新建一个files文件夹）
#### 4.3.解决目前trunk版无法正常使用校园web认证

这个问题困扰了很久，这几天有时间所以就找了下问题。问题找了好多，编译了很多次，最后把问题定位到了mwan上。

所在的环境为校园网，上网会跳转web认证页面，然后30秒之后认证页面就弹不出来了，打开任何网页无果。ping上层网关通，ping同级ip通，把网线拔了重新插在wan口上，又会有几十秒的时间跳转，过了这个时间就又不正常了。由于在校园，所在环境为自然的双栈地址，装了一个6relayd，按理说v4地址和v6地址都能获取到，v4不正常，v6应该正常，可是ipv6同样不正常。

经测试发现，插上wan口上，然后这时候随便刷新网页会有弹出认证页面，然后过一会就认证不了了。这时候再查看路由器web页面的系统日志信息
，发现确实有异常，于是尝试几次相同的操作，发现可以锁定问题。
```
Sun Nov  2 04:44:07 2014 user.notice mwan3: ifup interface wan (eth1)
Sun Nov  2 04:44:08 2014 daemon.info dnsmasq[1552]: reading /tmp/resolv.conf.auto
Sun Nov  2 04:44:08 2014 daemon.info dnsmasq[1552]: using local addresses only for domain lan
Sun Nov  2 04:44:08 2014 daemon.info dnsmasq[1552]: using nameserver 210.45.240.99#53
Sun Nov  2 04:44:08 2014 daemon.info dnsmasq[1552]: using nameserver 8.8.8.8#53
Sun Nov  2 04:44:08 2014 user.notice firewall: Reloading firewall due to ifup of wan (eth1)
Sun Nov  2 04:44:10 2014 daemon.notice netifd: Interface 'wan6' is now up
Sun Nov  2 04:44:10 2014 daemon.info hnetd[2022]: platform: interface update for br-lan detected
Sun Nov  2 04:44:10 2014 daemon.info hnetd[2022]: platform: interface update for lo detected
Sun Nov  2 04:44:10 2014 daemon.info hnetd[2022]: platform: interface update for eth1 detected
Sun Nov  2 04:44:10 2014 daemon.info hnetd[2022]: platform: interface update for eth1 detected
Sun Nov  2 04:44:11 2014 user.notice root: starting ntpclient
Sun Nov  2 04:44:11 2014 user.crit ddns-scripts[2421]: myddns_ipv4: CRITICAL ERROR - Service Configuration is disabled - EXITING
Sun Nov  2 04:44:11 2014 daemon.warn 6relayd[1567]: A default route is present but there is no public prefix on br-lan thus we don't announce a default route!
Sun Nov  2 04:44:11 2014 daemon.warn odhcpd[1379]: A default route is present but there is no public prefix on br-lan thus we don't announce a default route!
Sun Nov  2 04:44:12 2014 daemon.err miniupnpd[2508]: could not open lease file: /var/upnp.leases
Sun Nov  2 04:44:12 2014 daemon.notice miniupnpd[2508]: HTTP listening on port 5000
Sun Nov  2 04:44:12 2014 daemon.notice miniupnpd[2508]: HTTP IPv6 address given to control points : [fdc0:8c3b:8d0::1]
Sun Nov  2 04:44:12 2014 daemon.notice miniupnpd[2508]: Listening for NAT-PMP/PCP traffic on port 5351
Sun Nov  2 04:44:13 2014 user.notice firewall: Reloading firewall due to ifup of wan6 (eth1)
Sun Nov  2 04:44:13 2014 user.crit ddns-scripts[2593]: myddns_ipv6: CRITICAL ERROR - Service Configuration is disabled - EXITING
Sun Nov  2 04:44:13 2014 daemon.warn 6relayd[1567]: Termination requested by signal.
Sun Nov  2 04:44:14 2014 daemon.info dnsmasq[1552]: read /etc/hosts - 1 addresses
Sun Nov  2 04:44:14 2014 daemon.info dnsmasq[1552]: read /tmp/hosts/odhcpd - 1 addresses
Sun Nov  2 04:44:14 2014 daemon.info dnsmasq[1552]: read /tmp/hosts/6relayd - 0 addresses
Sun Nov  2 04:44:14 2014 daemon.info dnsmasq[1552]: read /tmp/hosts/dhcp - 1 addresses
Sun Nov  2 04:44:28 2014 user.info autossh[1475]: starting ssh (count 11)
Sun Nov  2 04:44:28 2014 user.info autossh[1475]: ssh child pid is 2685
Sun Nov  2 04:44:28 2014 user.info autossh[1475]: ssh exited with error status 255; restarting ssh
Sun Nov  2 04:44:42 2014 user.notice mwan3track: Interface wan (eth1) is offline
Sun Nov  2 04:44:42 2014 user.notice mwan3: ifdown interface wan (eth1)
Sun Nov  2 04:44:42 2014 user.info autossh[1475]: starting ssh (count 12)
Sun Nov  2 04:44:42 2014 user.info autossh[1475]: ssh child pid is 2795
Sun Nov  2 04:44:42 2014 user.info autossh[1475]: ssh exited with error status 255; restarting ssh
Sun Nov  2 04:44:49 2014 user.notice root: stopping ntpclient
```
观察可发现，插上网线会有mwan打开接口eth1，然后这时候可以通过认证，然后过一会没认证了，这时发现正好会出现mwan4track和mwan3 把eth1关掉了。（实际上，我在路由器shell中查看并没有关掉eth1）

mwan3是一个多拨的软件，我个人认为这个软件可能会尝试几次多播，不成功就停止运行，可能与我们校园的web认证冲突。
目前down下来的openwrt源码，默认的menuconfig中已经把mwan加到编译选项上。所以才会出现这种情况！
问题找到了，解决问题办法是删掉mwan3和mwan3track。ps观察在/usr/sbin/下，删除，重启，于是问题解决。
最终解决办法在编译openwrt的时候把mwan取消即可。
```
Network-->Routing and Redirection--->mwan3
LuCI-->Application-->luci-app-multiwam
LuCI-->Application-->luci-app-mwan3
```

#### 4.4 Openwrt安装pptp客户端

需要两个包，一般的固件中，可能都集成了。
一个是ppp-mod-pptp，另一个是luci-proto-pptp。

具体的配置可以在luci界面上配置，接口-网络-添加新接口，然后选择协议为pptp即可。
输入user,password,server即可。

另外比较重要的一部是在/etc/ppp/options和/etc/ppp/options.pptp文件中。以下为自己的配置文件。
```
root@OpenWrt:/etc/ppp# cat options
#debug
logfile /dev/null
#noipdefault
#noaccomp
#nopcomp
#nocrtscts
#lock
#maxfail 0
#lcp-echo-failure 5
#lcp-echo-interval 1

```


```
root@OpenWrt:/etc/ppp# cat options.pptp
noipdefault
noauth
nobsdcomp
nodeflate
idle 0
mppe required,no40,no56,stateless
maxfail 0
refuse-eap
refuse-pap

```

#### 4.5 HP 1020 plus打印机--->网络共享打印机

> 所需模块：kmod-usb-printer、p910nd、luci-app-p910nd

> 打印的原理即：
pc把要打印的文档通过打印机驱动封装后成为打印机可识别的数据，发送给路由器端，路由器端由p910nd监听程序进行转发，把接到的数据转发给打印机。所以路由器端是不需要驱动程序的，只要保证p910nd进程能够正常运行即可。

**具体配置（a,b,c,d）：**

a.由于惠普的低端打印机（类似于LaserJet 1010 1008 1009 1020 等）本身并不是自带firmware，我们每次打印的时候都是由PC端的驱动程序发送给打印机一个firmware。这个firmware在打印机的内存中，掉电后firmware就没了。基于打印机断电后不能够正常使用的问题，就是因为没有firmware的原因。firmware常用下载地址http://oleg.wl500g.info/hplj/ 找到自己的下载即可。也可以自己去编译，然后把编译出来的固件扔到路由器的/lib目录下。编译的过程参考http://wiki.openwrt.org/doc/howto/p910nd.server?s[]=p910nd

所以我们可以通过一个热插拔的脚本来做，每次usb接到路由器时，就把firmware这个固件推送到打印机上（所以每次插路由器的时候会听到打印机呗驱动的声音）。然后再进行打印即可。usb的热插拔采用hotplug模块，这个一般op中都是自带的了。
配置文件的位置在/etc/hotplug.d/usb/下。名字叫做20-usb_mode。可参考我的配置文件。
```
#!/bin/sh
set -e

# change this to the location where you put the .dl file:
FIRMWARE="/lib/sihp1020.dl"
DEVICE=/dev/usb/lp0
LOGFILE=/var/log/hp

if [ "$PRODUCT" = "3f0/2b17/100" -a "$ACTION" = "add" ]; then
for i in $(seq 30); do
if [ -c $DEVICE ]; then
echo "$(date) : Sending firmware to printer…" > $LOGFILE
nc 192.168.14.55 9109 < /lib/sihp1020.dl
echo "$(date) : done." ? $LOGFILE
exit
fi
sleep 1
done
fi
```
b.做完以上的步骤之后，并在/etc/rc.local下编辑如下内容，使p910nd进程自启。
```
/etc/init.d/p910nd start &

exit 0
```

c.另外在luci管理界面上，把p910nd的配置中，接口选择wan口。启用前勾上。bidirectional mode(双向模式)不要打钩。

也可参考配置文件。/etc/config/p910nd


```
config p910nd
        option device '/dev/usb/lp0'
        option enabled '1'
        option bind '192.168.14.55'
        option port '9'
```
d.在PC端，首先需要安装系统的驱动程序。
有两种方式，第一种是从百度上直接搜索驱动，然后插上打印机的usb到电脑上，安装驱动。
另一种方式不用插usb，直接在控制面板中，添加打印机，网络打印机，然后选择ip和端口名，然后windows update找到打印机响应型号
安装即可。

第一种安装完之后拔掉usb后，双击1020打印机，然后在左上角的打印下，把脱机打印取消。然后右击1020打印机，选择端口，添加端口，
把ip和端口号写好，其他的默认，确认。还有重要的一步是，把双向打印前的勾一定要取消掉，不然会出现重复打印的情况。

第二种安装完之后即可使用。注意 双向打印去掉。。

***稳定性*：**
还算可靠，在打印的时候，如果不能保证纸张充足的情况下打印，最后剩下的没有纸张了就不能够打印了，你在pc上再选择打印也不可以。
这时候需要把打印机usb从路由器上拔掉，等待3秒后重新插上，然后打印机就会把剩余的任务打印出来。


## 五 参考文献：

http://blog.csdn.net/qingfengtsing/article/details/39344327
http://blog.csdn.net/jk110333/article/category/1148871
http://openwrt.diandian.com/post/2014-09-16/40062999348
https://downloads.openwrt.org/kamikaze/docs/openwrt.html
https://dev.openwrt.org/wiki/GetSource
http://chaochaoblog.com/archives/1011
http://www.right.com.cn/forum/thread-146084-1-1.html
http://wenku.it168.com/d_000649332.shtml
http://www.right.com.cn/forum/thread-147651-1-1.html
http://www.right.com.cn/forum/thread-148069-1-1.html
http://www.right.com.cn/forum/home.php?mod=space&uid=200302&do=thread&type=thread&view=me&from=space
http://wenku.baidu.com/view/85d69c56ee06eff9aef80777.html
https://forum.openwrt.org/viewtopic.php?id=50795
https://github.com/pichuang/openvwrt