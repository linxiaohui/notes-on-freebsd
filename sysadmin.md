# 系统管理

## 挂载ISO文件

```
mdconfig -a -t vnode -f PATH_TO_YOUR_ISO/ISOFILE.iso   # Create a loop-back device
mount -t cd9660 /dev/mdX /cdrom  # Mount the created vnode to a folder
umount /cdrom
mdconfig -d -u X   # destroy the md-device
```


## 查看网络流量
```
   systat -if 1 （1表示1s刷新屏幕一次）
   netstat 1  # Traffic 流量 peak 峰值 average 平均值
```

## 时间校准
```
tzsetup	（时区设置 CST）
ntpdate cn.pool.ntp.org	（校准）
vi /etc/rc.conf
ntpd_enable="YES"		（配置开机启动 ntpd）
ntpd_sync_on_start="YES"
/etc/rc.d/ntpd start
vi /etc/ntp.conf
server 1.cn.pool.ntp.org
server 1.asia.pool.ntp.org
server 2.asia.pool.ntp.org	（配置time server）
/etc/rc.d/ntpd restart
```

## 终端响铃
   * 关闭vim响铃
```
vi ~/.vimrc
set vb t_vb=
```
   * 关闭终端响铃(以cshell为例)
```
echo "set nobeep" >> .cshrc  
```

## 终端分辨率
在内核配制文件里加入下面的编译内核
```
     options      VESA
     options      SC_PIXEL_MODE
```
第一行选项让內核支持VESA 2，第二行让内核支持控制台图形模式。

   * 获得本机支持的显示模式
```bash
vidcontrol -i mode < `tty`
```

输出如
```bash
    mode#     flags   type    size       font      window      linear buffer
------------------------------------------------------------------------------
……
280 (0x118) 0x0000000f G 1024x768x32 D   8x16  0xa0000 64k 64k 0x80000000 3072k
……
```

   * 改变当前屏幕的分辨率
```
vidcontrol MODE_280  （改变当前屏幕的分辨率为 280  1024x768x32）
```
   * 启动时生效
修改/etc/rc.conf
```
allscreens_flags="MODE_280"
```
   * 修改字体大小与颜色
```
vidcontrol green （把console改成黑底绿字的）
vidcontrol -f ... （console下的字体大小）
```


## 分区与文件系统
   * 给文件添加或禁用系统禁删标志（目录不适用）
```
chflags sunlink file1
chflags nosunlink file1
```
   * 初始化磁盘
```
fdisk -BI ad1
```
   * 建立FreeBSD分区
```
disklabel -B -w -r ad1s1 auto
```
   * 建立逻辑分区
```
disklabel -e ad1s1
```
   * 格式化分区，创建文件系统：
```
newfs /dev/ad1s1e
```
   * 创建交换文件
     * 创建一个交换文件(swap0)
```
dd if=/dev/zero of=/mnt/swap0 bs=1024k count=64   
```
     * 赋予这个文件适当的权限
```
chmod 0600 /mnt/swap0
```
     * 在/etc/rc.conf中启用交换文件
```
swapfile="/mnt/swap0"  #set to name ofswapfile
```
     * 重启生效
```
mdconfig -a -t vnode -f /mnt/swap0 -u o && swapon /dev/md0
```

   * 设置文件的ACL
     * `getfacl filename` 查看文件的acl信息
     * `setfacl filename` 设置文件的acl信息

## 动态库路径
系统动态库路径可以在**/etc/rc.conf**里加ldconfig_path="XXXX"在系统启动时设置。  
根据`man ldconfig`也可以：
   * 在/etc/ld-elf.so.conf中增加动态库路径
   * sudo /etc/rc.d/ldconfig restart或/etc/rc.d/ldconfig forcerestart  
   * ldconfig -r  （查看库文件路径）

## 记录使用者指令
**/etc/rc.conf**
```
accounting_enable="YES"
```
系统会将使用者的历程记录在/var/account/acct*中。
`lastcomm`根据/var/account/acct输出所记录的数据。
也可以使用lastcomm -f acct1来查看前一天的记录信息。

## Linux
Linux /proc文件系统  
`kldload linprocfs`  
`mount -t linprocfs linprocfs /compat/linux/proc`  
开启Linux 二进制兼容支持（启用这一功能最简单的方法是载入 linux KLD 模块）：  
`kldload linux`  
在内核中静态链接进Linux二进制兼容模式，在内核配置文件里面加入：
```
 	options COMPAT_LINUX
```
如果希望LINIX兼容支持在系统初始过程中期待启动,则
vim /etc/rc.conf
```
linux_enable="yes"
```

echo 'linux_load="YES"' >> /boot/loader.conf



## mtod
   * 登入后显示  
   Message Of The Day(motd)   /etc/motd  
   * 登入前显示  
     * 修改 /etc/gettytab 及 /etc/issue
     * 编辑 /etc/gettytab，找到 default的地方
```
   default:\:cb:ce:ck:lc:fd#1000:im=\r\n%s/%m (%h) (%t)\r\n\r\n:sp#1200:\
   :if=/etc/issue:
```
其中的%s %m %h %t分别对应到FreeBSD amd64 hostname tty
`if=/etc/issue`表明如果没有`/etc/issue`的话就执行default。

## 常用命令
   * 查看 sysctl 具体解释
```
sysctl -d kern.maxvnodes
kern.maxvnodes: Maximum number of vnodes
```
   * sync  
让系统暂存的数据强制存回硬盘

   * 若断电后系统无法启动：
     * 启动到sing user模式
     * fsck
     * reboot
   * du -h -I /etc
     * -I 参数来省略指定目录下的子目录  
     * -h 表示使用GB、MB等易读的格式
   * 锁住控制台
lock -v 锁住console（不注销）
     * -n 永不超时
     * -p 使用系统密码作为开启终端的密匙
   * grep  
让 grep 高亮匹配出的字符串
在/etc/csh.cshrc中加入如下配置
```
setenv GREP_OPTIONS --color=auto
```
   * shutdown
     * shutdown now 切换到单用户模式
       * 切换或启动到单用户模式后，ctrl+d进入多用户模式
     * shutdown -p now
     * shutdown -hp now 关闭电源
     * shutdown -r now reboot 重新启动机器
     * shutdown -p +90 (90分钟后关机)
     * shutdown 0203122359(0203122359表示2002年3月12日23:59 yymmddhhmm)

   * 查找程序或文件
     * which 程序名
     * whereis 程序名
     * find 文件名
     * whatis xxx 要找东西但不知道它是什么
     * locate 文件名
        * locate: database too small: /var/db/locate.database的一种解决方法
        * 开机时间不够长，检查/etc/periodic/weekly/310.locate
        * 或运行/usr/libexec/locate.updatedb

   * ktrace
      * man ktrace
      * man truss
      * /usr/ports/devel/strace
         * 用于32位系统
         * 依赖/proc文件系统, `mount -t procfs proc /proc`
