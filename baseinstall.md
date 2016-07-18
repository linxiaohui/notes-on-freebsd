# 安装配置

## 基本安装

   * 系统环境
```
FreeBSD-10.1-RELEASE-amd64 @  Thinkpad X201i
```
   * 安装基本系统
     * 根据FreeBSD手册，制作U盘启动盘
        * 使用`dd`或`ImageWriter`
     * 安装系统
        * **注意** FreeBSD必须安装在主分区，注意磁盘分区。
     * 使用`bsdconfig` 配置系统网络等
     * 安装X、gnome3
```bash
pkg install xorg
pkg install gnome3
```
     * 配置`loader.conf`
```vimrc
loader_logo="beastie" #boot menu logo
autoboot_delay="3"    #boot delay
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
     * 影响 Login Shell
     * 修改/etc/login.conf
```
chinese:Chinese Users Account:\
	:charset=UTF-8:\
	:lang=zh_CN.UTF-8:\
	:tc=default:
```
     * 执行
```
cap_mkdb /etc/login.conf     # rebuilds the database file /etc/login.conf.db
pw usermod linxh -L chinese	 # 修改用户语言环境
pw useradd linxh -L chinese	 # 添加用户并使用中文
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


   * 环境变量`LANG=xx_YY.ZZZZ`
      * sets the system locale to language code xx
      * country code YY
      * character encoding ZZZZ
      * `locale -a | grep UTF-8` : a list of every available UTF-8 locale

   * /etc/profile
      * non login shell
      * GDM and other login managers

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
