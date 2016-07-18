# 系统配置


## 无线网络配置
无线网卡一般为PCI设备，可以使用命令`pciconf –lv`查看网卡信息

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
```
**/etc/wpa_supplicant.conf**
```
   ctrl_interface=/var/run/wpa_supplicant
   ctrl_interface_group=wheel
```
`/etc/rc.d/wpa_supplicant restart wlan0`   
`wpa_gui`
   5. 常用命令
```
ifconfig wlan0 scan  查看无线网络（SSID等）
ifconfig wlan0 up scan  
ifconfig wlan0 list scan
dhclient wlan0
man wpa_supplicant.conf
ifconfig wlan0
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
googlecode无法访问，根据make提示，搜索相关的tar包，下载后 放入提示的目录下
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
   * 测试显卡是否能实现3D功能：
    `dmesg | grep agp`  如果出现:
   agp0: <VIA 82C691 (Apollo Pro) host to PCI bridge> mem
   0xe0000000-0xe3ffffff at device 0.0 on pci0 的字样，那么该显卡就有可能实现3D功能。
   * `kldload drm`
   如果没报错该显卡基本上就可以实现3D功能了。
   * `glxinfo`
   如果出现"Direct Rendering: YES"字样说明已经启用3D加速了。
   more /var/log/Xorg.0.log | grep "direct rendering"
 (II) I810(0): direct rendering: Enabled

## pkg
FreeBSD 10.1使用`pkg`替代了`pkg_add`等命令。   
`man pkg`确定pkg使用的配置文件

/usr/local/etc/pkg/repos/FreeBSD.conf
```
FreeBSD: {
  url: "http://pkg.FreeBSD.org/${ABI}/latest",
  mirror_type: "srv",
  enabled: "yes"
}
```
可选的镜像站点
```
pkg.tw.freebsd.org
pkg.eu.FreeBSD.org
pkg.us-east.FreeBSD.org
pkg.us-west.FreeBSD.org
```

   * pkg **info**
   显示所有包信息
   * pkg **info -d** subversion
   显示包之间的依赖关系
   * pkg **info -l** subversion
   查看一个包安装了哪些文件
   * pkg **which**
   查看一个文件属于哪个包
   * pkg **query**
   * pkg **help** XXX
   * pkg **update [-f]**

## ports
### portsnap
**/etc/portsnap.conf**
```
SERVERNAME=portsnap.hshh.org
```
portsnap服务器：
```
  portsnap.hshh.org
  portsnap2.hshh.org
  portsnap3.hshh.org (网通)
  portsnap4.hshh.org
```
首次使用portsnap必须执行下面2步：
   * portsnap fetch     获取portsnap快照的最新压缩包
   * portsnap extract   将压缩包创立到/usr/ports
   * 备注
      * 可以合起来使用`portsnap fetch extract`
      * 会重新创建/usr/ports

使用portsnap更新ports：
   * portsnap fetch
   * portsnap update
   * 备注
      * 可以合起来使用`portsnap fetch update`

### make.conf
修改/etc/make.conf文件, 配置ports和src源. 可以安装axel替代fetch提高下载速度。
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
#有新的服务器加入就直接往后面加就可以了
#格式就是"源地址+/${DIST_SUBDIR}/ \"
#但是不要同时存在2个"MASTER_SITE_OVERRIDE?"
#否则第二个无效(more /usr/share/example/etc/make.conf)
```

### 常用命令
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
   * 通过软件名查找ports
```
cd /usr/ports
make search name=xxx | grep ^Path
#通过关键字查找（在软件名和说明中包括这个单词的）：
cd /usr/ports
make search key=xxx | grep ^Path
```

## grub2引导FreeBSD
   * 进入Linux，`sudo grub2-install /dev/sda`
   * /etc/grub.d/40_custom*中添加FreeBSD项目
```
menuentry "FreeBSD(on /dev/sda2)" {
 set root='(hd0,msdos2)'
 insmod ufs2
 chainloader +1
 boot
}
```
   * rm /boot/grub2/grub.cfg 删除掉原配置文件
   * sudo grub2-mkconfig > /boot/grub2/grub.cfg 新生成

## 用Windows的引导器启动FreeBSD
   * Windows下 `fdisk /mbr`
   * 把FreeBSD光盘上的boot\boot1复制到c:\
   * 编辑c:\boot.ini 加一行c:\boot1="FreeBsd"


## 安装VMWare Tools
 * 安装`compat6x-amd64`
 * 在VMWare中选择安装VMWare Tools后, FreeBSD中需要`mount -t cd9660 /dev/cd0 /dist`后, 从`/dist`中复制VMWare安装包
 * 注意Perl的路径
