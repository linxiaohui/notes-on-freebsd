# 系统管理

## 用户与组
  * toor用户  
   设置备用管理员账号toor（root的逆序）的密码，
在root不能登陆的时候（如shell改错了）就可以用它登陆进行维护。

  * wheel用户组  
FreeBSD要求su到root的用户必须为wheel组中的成员。
可以使用bsdconfig修改,也可以直接修改/etc/group文件(man group).

  * 利用pw管理用户信息
    * 添加组
`pw groupadd group1`
    * 添加用户
`echo <password> |pw useradd -n username -g group -m -s /bin/csh -h 0`
      * -n username  /指定用户名称
      * -g groupname  /指定组名称
      * -m  /自动创建用户目录
      * -s /指定用户shell
      * -h 0  /可以在创建用户的时候直接创建改用户的密码
    * 删除用户
`pw userdel -n username -r`   
      * -r 同时删除home目录相关资料
    * 查看组成员
`pw groupshow wheel`
    * 添加组成员
`pw groupmod wheel -M user1`
      * -M  设置用户为这个组的唯一组员
      * -m  添加用户到该组


  * 利用命令交互式的添加用户
    * 添加用户 adduser
    * 删除用户 rmuser
    * 用于修改用户数据库信息的工具 chpass
    * 修改用户口令的工具 passwd
    * 修改用户帐号的工具 pw   


  * 使用id命令显示用户信息  
`id root`  
 uid=0(root) gid=0(wheel) groups=0(wheel), 5(operator)

  * 变更Shell
`chsh -s /bin/tcsh`   
注意**输入的shell名称一定要存在于/etc/shells中**

  * 单用户需要输入密码
“Insecure Console (Single-user mode)
当FreeBSD以单用户模式(single-user mode)启动后，不输入密码即可进入命令提示符。
如设置单用户模式启动后需输入root密码登陆，
修改**/etc/ttys**文件，将
```
console none 		unknown off secure
```
中 **secure**  修改为 **insecure**
```
console none 		unknown off insecure
```

  * 丢失root密码
    * 单用户模式不需要输入密码  
启动到单用户模式
```/sbin/mount –a  （挂载/etc/fstab里所有列出的文件系统）
passwd
```
    * 用户登录也需要密码
      * 使用安装盘（CD/U盘）引导，选择修复模式
      * mount硬盘上的文件系统`mount /dev/ada0s2a /mnt`
      * 修改 ttys  
vi /mnt/etc/ttys  更改`console none unknow off insecure`为**secure**
      * 重新启动到单用户模式后`/sbin/mount –a; passwd`


  * chsh错了的解决办法   
系统启动时进入"单用户"模式:
把/etc/passwd和/etc/master.passwd里root的shell都改成/bin/csh  
然后执行pwd_mkdb
```
mount -u
mount -a
ee /etc/passwd
ee /etc/master.passwd
pwd_mkdb /etc/master.passwd
```
因为login的时候读取的是/etc/pwd.db和/etc/spwd.db  
所以更改了/etc/passwd和/etc/master.passwd之后需要执行`pwd_mkdb`  

vipw修改master.passwd时, vipw会先将master.passwd以预设的编辑器打开,
修改保存后, 它会自动更新db文件.

  * 用户帐户锁定
    * 用vipw将其shell修改成/sbin/nologin
    * pw
      * pw lock userid
      * pw unlock userid （解锁）

