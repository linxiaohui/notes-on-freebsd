# 桌面环境

## 配置X
   * 修改系统配置文件
     * 修改`/boot/loader.conf`
```
kern.vty=vt
hw.vga.textmode=1
```
     * 修改`/etc/rc.conf`
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
     * `Xorg -configure` 将在当前目录中生成名为xorg.conf.new的配置文件
     * `Xorg -config xorg.conf.new` 测试生成的文件, 若成功则显示一个黑色的界面 
     * `Xorg -config xorg.conf.new -retro` 显示为中间有个X的灰色界面上步的配置成功
     * `Ctrl+Alt+F1`将返回原配置终端, `Ctrl+C`停止
     * `cp xorg.conf.new /etc/X11/xorg.conf`
   * 启动X
     * `startx`
   * 配置gdm
     * **/etc/rc.conf**
```
gdm_enable="YES"
gnome_enable="YES"
```
     * **~/.dmrc**
```
Language=zh_CN.UTF-8
```
     * **/usr/local/etc/gdm/locale.conf**
```
LANG="zh_CN.UTF-8"
LC_CTYPE="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
```
   * 注意事项
     * 需要设置环境变量 LANG 为 UTF-8，否则在gnome中启动"终端"会失败
     * 新的Xorg中使用Intel or ATI Radeon显卡，必须在loader.conf中配置`kern.vty=`**vt**
        * 否则Xorg -config时`Ctrl+Alt+F1`将无法返回

## 安装中文字体及输入法
   1. 安装文泉驿中文字体
```
cd /usr/ports/x11-fonts/wqy
make install clean
```
   2. 添加Times Roman,Helvetica,Palatino等Type1字体
```
cd /usr/ports/x11-fonts/urwfonts  
make install clean 
```
   3. 安装微软雅黑、宋体等TrueType字体   
将要安装的TrueType 字体复制到/usr/local/lib/X11/fonts/TrueType，执行
```
cd /usr/local/lib/X11/fonts/TrueType  
ttmkfdir -o fonts.dir (让X字体引擎知道已经安装了这些新文件)
xlsfonts (显示系统中安装的字体)
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

## 声音
安装后遇到系统没有声音，但是插入耳机有声音的情况
   * /boot/loader.conf
```
snd_hda_load="YES"
boot_verbose="-v"  （或者：重启看到启动菜单时按空格暂停，选择 verbose 模式回车）
cat /dev/sndstat 
```
   * 重启
   * 执行 dmesg | grep hda > hda.log 得到默认的硬件配置状况
```
grep hda /var/run/dmess.boot
nid：nodes with ID（设备节点 ID）
as：association（第 N 组）
seq：sequence（顺序）
```
在**/boot/device.hints**中把扬声器和耳机放到同一组里，  
还需要当检测到耳机插入时，将扬声器静音，输出到耳机，  
根据 man snd_hda 中的描述，为了实现上述功能必须把 Headphones 的 seq 设置为15。
因此得到如下配置
```
hint.hdac.0.cad0.nid31.config="as=1 seq=0 device=Speaker"
hint.hdac.0.cad0.nid25.config="as=1 seq=15 device=Headphones"
```
照上面配置好声音通道和 pcm 设备后，因为 pcm0 设备关联到了内置的扬声器   
故无需再修改hw.snd.default_unit 为 1 了，直接使用默认的 0   
```
sysctl hw.snd
mixer  控制台调音量
sysctl dev.pcm
```
   * 附：
```
kldload snd_driver      (加载所有声卡驱动自动匹配)
cat filenam > /dev/dsp  (测试声卡，filenam为任意文件产生噪音说明正常工作)
```

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
