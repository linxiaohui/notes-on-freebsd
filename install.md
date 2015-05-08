# 安装配置

## 基本安装

   * 系统环境
```
FreeBSD-10.1-RELEASE-amd64-dvd1.iso @  Thinkpad X201i
```
   * 基本安装
     * 根据FreeBSD手册，制作U盘启动盘，然后安装系统；
需要注意的是FreeBSD必须安装在主分区。注意磁盘分区。
     * 安装X、gnome3
```bash
pkg install x-org
pkg install gnome3
```
     * 配置`loader.conf`
```vimrc
loader_logo="beastie"		(boot menu logo)
autoboot_delay="3"		(boot delay)
```
     * 配置`rc.conf`
```
powerd_enable="YES"
powerd_flags="-a adaptive -b adaptive -n adaptive"  # 保持温度不会过高
sendmail_enable="NONE"
#screensaver程序
blanktime="900" 启用时间为15分钟，以秒为单位
saver="logo"   图形接口（图形），daemon（文字）
```
   * 备注
     * `man loader.conf`，`man rc.conf`获取关于这两个配置文件的更多信息
     * 修改/etc/rc.conf不重启生效
```
sh /etc/rc
/etc/netstart
```

## 本地化
   * 方法一：Class Definitions
     * 修改/etc/login.conf
```
chinese:Chinese Users Account:\
	:charset=UTF-8:\
	:lang=zh_CN.UTF-8:\
	:tc=default:
```
     * 执行
```bash
cap_mkdb /etc/login.conf  (rebuilds the database file /etc/login.conf.db)
pw usermod linxh -L chinese	(修改用户语言环境)  或者
pw useradd linxh -L chinese		(添加用户并使用中文)
```
     * 执行`pw usershow linxh`检查输出
```
    linxh:*:1001:1001:chinese:0:0:linxh:/home/linxh:/bin/sh
```
     * 执行`su - linxh` `locale`检查输出
```
LANG=zh_CN.UTF-8
LC_CTYPE="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_ALL=
```

   * 方法二：User Definitions
     * 修改~/.login_conf
```
me:\
	:lang=zh_CN.UTF-8:\
	:setenv=LC_CTYPE=zh_CN.UTF-8:\		
	:setenv=LC_COLLATE=zh_CN.UTF-8:\	
	:setenv=LC_TIME=zh_CN.UTF-8:\		
	:setenv=LC_NUMERIC=zh_CN.UTF-8:\	
	:setenv=LC_MONETARY=zh_CN.UTF-8:\	
	:setenv=LC_MESSAGES=zh_CN.UTF-8:\
	:setenv=LC_ALL=zh_CN.UTF-8:\
	:charset=UTF-8:
```

## 配置X
   * 修改系统配置文件
     * 修改`/boot/loader.conf`[1]
```
kern.vty=vt
hw.vga.textmode=1
```
     * 修改` /etc/rc.conf`
```
dbus_enable="YES"
hald_enable="YES"
```
   * 配置gnome
     * 修改`/etc/fstab`
```
proc /proc procfs rw 0 0
```
     * 修改`~/.xinitrc`
```
exec gnome-session
```
   * 重启`reboot`
   
   * 配置X
     * `Xorg -configure` 将在 /root 中生成名为xorg.conf.new的配置文件
     * `Xorg -config xorg.conf.new` 测试生成的文件, 若成功则显示一个黑色的界面 
     * `Xorg -config xorg.conf.new -retro` 显示为中间有个X的灰色界面
上步的配置成功, `Ctrl+Alt+F1`将返回原配置终端, `Ctrl+C`停止
     * `cp xorg.conf.new /etc/X11/xorg.conf`
   * 启动X
     * `startx`
   * 配置gdm
     * /etc/rc.conf 
```
gdm_enable="YES"
gnome_enable="YES"
```
     * ~/.dmrc
```
Language=zh_CN.UTF-8
```
     * /usr/local/etc/gdm/locale.conf
```
LANG="zh_CN.UTF-8"
LC_CTYPE="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
```
   * 注意事项
     * 需要设置环境变量 LANG 为 UTF-8，否则在gnome中启动"终端"会失败
     * 根据
```
Because of the new Xorg, you have to use vt, 
due to the new drivers requiring KMS support (which the traditional syscons does not have),
but only if you have an Intel or ATI Radeon card, AFAIK…
Without this, Xorg completely got a black screen (and totally unresponsive) 
in my case and had to restart the system…
So let’s use this vt (I also set it to text mode, but that is not necessary)
```

## 安装中文字体及输入法
   1. 安装文泉驿中文字体
```
cd /usr/ports/x11-fonts/wqy
make install clean
```
   2. 添加Times Roman,Helvetica,Palatino等Type1字体【4】
URW字体集合 (x11-fonts/urwfonts) 包括了高质量的 标准 type1 字体。
```
cd /usr/ports/x11-fonts/urwfonts  
make install clean 
```
   3. 安装微软雅黑、宋体等等中文字体即 TrueType字体
Xorg 已经内建了对 TrueType 字体的支持。为 TrueType 字体创建一个目录，如/usr/local/lib/X11/fonts/TrueType， 然后把所有的 TrueType 字体复制到这个目录。能被 X11 使用的必须是UNIX/MS-DOS/Windows 格式的。
```
cd /usr/local/lib/X11/fonts/TrueType  
ttmkfdir -o fonts.dir (让X字体引擎知道已经安装了这些新文件)
用 ttmkfdir 来创建 fonts.dir 文件，xlsfonts显示系统中安装的字体。
```
   4. 编辑/etc/X11/xorg.conf
```
FontPath "/usr/local/lib/X11/fonts/wqy" 
FontPath "/usr/local/lib/X11/fonts/URW/"  
FontPath "/usr/local/lib/X11/fonts/TrueType"  
```
   5. 安装ibus-pinyin
     * GNOME 中 IBUS首选项设置
     * 设置区域与语言中设置输入源，设置键盘修改输入法切换的快捷键
   6. 配置文件
     * .cshrc
```
setenv XIM ibus 
setenv GTK_IM_MODULE ibus 
setenv QT_IM_MODULE xim 
setenv XMODIFIERS @im=ibus 
setenv XIM_PROGRAM ibus-daemon 
setenv XIM_ARGS "--daemonize --xim" 
```
     * .xinitrc
```
exec gnome-session
XIM=ibus
GTK_IM_MODULE=ibus
QT_IM_MODULE=xim
XMODIFIERS='@im=ibus'
XIM_PROGRAM="ibus-daemon"
XIM_ARGS="-d -x"
```
   7. 备注 :
     * `Load "freetype"`后 gconf-editor才能显示 界面 
     * 可以用`ibus-daemon -x -d -r`测试ibus是否能够启动

## 无线网络配置
无线网卡一般为PCI设备，可以使用命令pciconf –lv查看网卡信息
查看无线网络（SSID等） ifconfig wlan0 scan 
   1. /etc/wpa_supplicant.conf 
```
network={
        ssid="MyWifi"
        key_mgmt=NONE
}
```
或者
```
network={
        ssid="MyWPA"
        psk="SomeP@sswd!"
}
```
   2. /etc/rc.conf
```
wlans_iwn0="wlan0"
#ifconfig_wlan0="WPA DHCP"
ifconfig_wlan0="ssid ChinaNet bssid 54:89:98:8a:8a:40 DHCP"
/etc/rc.d/netif start
```
   3. wpa_cli 工具
   4. ports:
```
net/pcbsd-netmanager
wpa_gui
 /etc/wpa_supplicant.conf.
```
		ctrl_interface=/var/run/wpa_supplicant
		ctrl_interface_group=wheel
```
`/etc/rc.d/wpa_supplicant restart wlan0`
`wpa_gui`
   5. 常用命令
```
ifconfig wlan0 up scan
ifconfig wlan0 list scan
dhclient wlan0
man wpa_supplicant.conf 
ifconfig wlan0
```

## 声音
安装后遇到系统没有声音，但是插入耳机有声音的情况
   * /boot/loader.conf
```
 snd_hda_load="YES"
 boot_verbose="-v"  （或者：重启看到启动菜单时按空格暂停，选择 verbose 模式回车）
 cat /dev/sndstat 
```
   * 登陆后执行 dmesg | grep hda > hda.log 得到默认的硬件配置状况
```
grep hda /var/run/dmess.boot
nid：nodes with ID（设备节点 ID）
as：association（第 N 组）
seq：sequence（顺序）
在  /boot/device.hints
把扬声器和耳机放到同一组里，还需要当检测到耳机插入时，将扬声器静音，输出到耳机，根据 man snd_hda 【2】中的描述，为了实现上述功能必须把 Headphones 的 seq 设置为15。
因此得到如下配置：
hint.hdac.0.cad0.nid31.config="as=1 seq=0 device=Speaker"
hint.hdac.0.cad0.nid25.config="as=1 seq=15 device=Headphones"
照上面配置好声音通道和 pcm 设备后，因为 pcm0 设备关联到了内置的扬声器，故无需再修改hw.snd.default_unit 为 1 了，直接使用默认的 0
sysctl hw.snd
mixer  控制台调音量
sysctl dev.pcm
   * 附：
```
kldload snd_driver	      （加载所有声卡驱动自动匹配）
cat filenam > /dev/dsp	(测试声卡，filenam为任意文件产生噪音说明正常工作）
```

## NTFS与exFat分区
   1. 安装ntfs-3g
```
cd /usr/ports/sysutils/fusefs-ntfs
make install clean
```
   2. 启动时加载fuse
```
vim /etc/rc.conf
fusefs_enable="YES"
vim /boot/loader.conf
fuse_load="YES"
```
   3. 手动挂载 ntfs
```
mount -t ntfs -o rw,mountprog=/usr/local/bin/ntfs-3g,late /dev/ada0s7 /media/
或者  ntfs-3g /dev/ada0s7 /media/
[ ntfs-3g /dev/ada0s7 /media -o rw,umask=0,locale=zh_CN.utf8 ]
```
   4. 自动挂载
```
/etc/fstab
/dev/ada0s7 /media/ ntfs rw,mountprog=/usr/local/bin/ntfs-3g,late 0 0
备注：配置fstab自动挂载后，在GNOME界面可以点击图标进入相应的NTFS分区。
```
   5. 安装fuse-exfat
```
googlecode无法访问，根据make提示，搜索相关的tar包， 在【5】下载后 放入提示的目录下 
make install
```
   6. mount 光盘
```
mount_cd9660 /dev/cd0 /cdrom
```
   7. 强制卸载
```
umount -f /dev/ad0s7
```

## 启用3D加速功能
     DRI(Direct Rendering Infrastructure)是实现3D功能最重要的部分。
     可以把它理解为X和硬件间的接口。
     但它包含很多独立的模块，负责从X到硬件的各个环节，包括内核。
     它最主要的工作就是将Mesa或OpenGL的函数调用转换为硬件的指令，从而指挥硬件进行渲染等工作。
   * 测试显卡是否能实现3D功能：
    `dmesg | grep agp`  如果出现:
   agp0: <VIA 82C691 (Apollo Pro) host to PCI bridge> mem 
   0xe0000000-0xe3ffffff at device 0.0 on pci0 的字样，那么该显卡就有可能实现3D功能。
   * `kldload drm`
   如果没报错该显卡基本上就可以实现3D功能了。
   * `glxinfo`
   如果出现”Direct Rendering: YES“字样说明已经启用3D加速了。
   more /var/log/Xorg.0.log | grep "direct rendering"
 (II) I810(0): direct rendering: Enabled 

## pkg
To use binary packages:
1. Ensure your pkg(8) is up-to-date. 'pkg -v' should say at least
   1.1.4_8. If it does not, first upgrade from ports.
2. Remove any repository-specific configuration from
   /usr/local/etc/pkg.conf, such as PACKAGESITE, MIRROR_TYPE, PUBKEY.
   If this leaves your pkg.conf empty, just remove it.
3. mkdir -p /usr/local/etc/pkg/repos
4. Create the file /usr/local/etc/pkg/repos/FreeBSD.conf with:
FreeBSD: {
  url: "http://pkg.FreeBSD.org/${ABI}/latest",
  mirror_type: "srv",
  enabled: "yes"
}
Mirrors you may use instead of the global pkg.FreeBSD.org:
/etc/pkg/FreeBSD.conf
"pkg+http://pkg.tw.freebsd.org/${ABI}/latest/"
    pkg.eu.FreeBSD.org
    pkg.us-east.FreeBSD.org
    pkg.us-west.FreeBSD.org
<Official FreeBSD Binary Packages now available for pkgng>”

   * pkg info
   显示所有包信息
   * pkg info -d subversion
   显示包之间的依赖关系
   * pkg info -l subversion
   查看一个包安装了哪些文件
   * pkg which
   查看一个文件属于哪个包
   * pkg query
   * pkg help XXX
   * pkg update [-f]

## ports
   1. /etc/portsnap.conf
```
SERVERNAME=portsnap.hshh.org
```
portsnap服务器：
  portsnap.hshh.org
  portsnap2.hshh.org
  portsnap3.hshh.org (网通)
  portsnap4.hshh.org
```
     * 首次使用portsnap：
首次使用portsnap必须执行下面2步：
       * portsnap fetch
       * portsnap extract
可以合起来使用：
       * portsnap fetch extract
       * portsnap fecth是从网上获取portsnap快照的最新压缩包。
       * portsnap extract 则是把这个压缩包创立到/usr/ports。
哪怕以前已经手工安装了ports，也会重新创立一次。
       * 使用portsnap更新ports：
以后更新，只需要执行下面2步：
            * portsnap fetch
            * portsnap update
这2步可以合起来使用：
       * portsnap fetch update
       * portsnap第一次运行extract命令时，可能需要一段时间，
以后更新使用update的时候，速度就快很多了。
   2. /etc/make.conf
配置ports和src的国内源，推荐安装axel替代fetch提高下载速度，修改/etc/make.conf文件：
cd /usr/ports/ftp/axel
make install
vim /etc/make.conf
```
    FETCH_CMD=axel
    FETCH_BEFORE_ARGS= -n 10 -a
    FETCH_AFTER_ARGS=
    DISABLE_SIZE=yes
    MASTER_SITE_OVERRIDE?=\
    http://ports.hshh.org/${DIST_SUBDIR}/\
    http://ports.cn.freebsd.org/${DIST_SUBDIR}/\
    ftp://ftp.freeBSDchina.org/pub/FreeBSD/ports/distfiles/${DIST_SUBDIR}/\
    http://mirrors.ustc.edu.cn/freebsd/ports/distfiles/${DIST_SUBDIR}/\
    http://mirrors.tuna.tsinghua.edu.cn/freebsd/distfiles/${DIST_SUBDIR}/\
ftp://ftp.hk.freebsd.org/pub/FreeBSD/distfiles/${DIST_SUBDIR}/\
ftp://ftp.tw.freebsd.org/pub/FreeBSD/distfiles/${DIST_SUBDIR}/\
ftp://ftp.freebsd.org/pub/FreeBSD/distfiles/${DIST_SUBDIR}/	
MASTER_SITE_OVERRIDE?=${MASTER_SITE_BACKUP}
CPUTYPE?=core2		(根据情况调整，起到优化作用)
INSTALL=install -C	(比对已安装文件与最新文件 版本号，低于新版本才安装)
    		有新的服务器加入就直接往后面加就可以了，格式就是“源地址+/${DIST_SUBDIR}/ \”
但是不要同时存在2个“MASTER_SITE_OVERRIDE?”，否则第二个无效
		(more /usr/share/example/etc/make.conf)
```
   3. 常用命令
     * make fetch 
     只抓取tarball 
     * make fetch-recursive
     抓取安装ports所有须要的其他ports的tarball
     * make fetch-list
     列出port所需的文件
     * make clean 
     Ports里面make clean,会附带着make clean依赖的软件的
     * make -DBATCH install
     不需要用户输入任何东西
     * make -DINTERACTIVE install
     继续上一步
     * make configure 
     * make distclean
     删除不想要的distfiles
     * make distclean
     移除不是port collections所期望下载的文件   
     * make rmconfig
     清除用户配置的参数
     * make showconfig
     查看当前配置的参数
     * make config
     更改参数
     * 通过软件名查找ports：
cd /usr/ports
make search name=xxx | grep ^Path
通过关键字查找（在软件名和说明中包括这个单词的）：
cd /usr/ports
make search key=xxx | grep ^Path


## grub2引导FreeBSD
   * 进入Linux，`sudo grub2-install /dev/sda`
   * 要添加项目，诸如
```
menuentry "FreeBSD(on /dev/sda2)" {
 set root='(hd0,msdos2)'
 insmod ufs2
 chainloader +1
 boot
}
```
放入/etc/grub.d/40_custom***什么的中。
   * rm /boot/grub2/grub.cfg 删除掉原有的
   * sudo grub2-mkconfig > /boot/grub2/grub.cfg
从新生成即可。

## 用win2000的引导器启动FreeBSD
   我在笔记本上装了双系统，win2000 和freebsd，装完 freebsd后 系统自动设置如下： 
   f1 dos 
   f2 freebsd 
   * 先起动到dos下，fdisk /mbr
   * 把FreeBSD光盘上的boot\boot1复制到c:\,
   * 再编辑c:\boot.ini 加一行c:\boot1="FreeBsd"

## 安装flashplayer插件 
   * 安装 nspluginwrapper
```bash
cd /usr/ports/www/nspluginwrapper    
make    
make install  
```
   * 安装 linux-c6-flashplugin11
```bash
cd /usr/ports/www/linux-c6-flashplugin11   
make  
make install  
```
   * 执行`nspluginwrapper -v -a -i`
     * 若报错Kernel too old, 执行命令`sysctl compat.linux.osrelease=2.6.18`
     * 修改/etc/sysctl.conf, 增加`compat.linux.osrelease=2.6.18`

   * 修改/etc/fstab
```
linproc /usr/compat/linux/proc linprocfs rw 0 0   
```

   * 备注
     * 安装步骤参考FreeBSD Handbook
     * 安装nspluginwrapper会安装linux_base-c6
     * 注意安装的版本  
