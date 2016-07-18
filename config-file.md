# 常用配置文件

## /etc/sysctl.conf
```
# Enhance shared memory X11 interface
kern.ipc.shmmax=67108864
kern.ipc.shmall=32768
# Enhance desktop responsiveness under high CPU use (200/224)
kern.sched.preempt_thresh=224
# Bump up maximum number of open files
kern.maxfiles=200000
# Disable PC Speaker
hw.syscons.bell=0
# Shared memory for Chromium
kern.ipc.shm_allow_removed=1

# Allow users to mount disks
vfs.usermount=1

```
Some knobs can only be set at boot by the loader by setting them in /boot/loader.conf. This is also where we define kernel modules to load at boot.

## /boot/loader.conf
```
# Use new graphical console driver
kern.vty=vt
# Devil worship in loader logo
loader_logo="beastie"
# Boot-time kernel tuning
kern.ipc.shmseg=1024
kern.ipc.shmmni=1024
kern.maxproc=10000
# Load MMC/SD card-reader support
mmc_load="YES"
mmcsd_load="YES"
sdhci_load="YES"
# Access ATAPI devices through the CAM subsystem
atapicam_load="YES"
# Filesystems in Userspace
fuse_load="YES"
# Intel Core thermal sensors
coretemp_load="YES"
# AMD K8, K10, K11 thermal sensors
amdtemp_load="YES"
# In-memory filesystems
tmpfs_load="YES"
# Asynchronous I/O
aio_load="YES"
# Handle Unicode on removable media
libiconv_load="YES"
libmchain_load="YES"
cd9660_iconv_load="YES"
msdosfs_iconv_load="YES"
```

## /etc/rc.conf
```
moused_enable="YES"
# powerd: hiadaptive speed while on AC power, adaptive while on battery power
powerd_enable="YES"
powerd_flags="-a hiadaptive -b adaptive"
# Enable BlueTooth
hcsecd_enable="YES"
sdpd_enable="YES"
# Synchronize system time
ntpd_enable="YES"
# Let ntpd make time jumps larger than 1000sec
ntpd_flags="-g"
Enable remote access via SSH if you plan to use it. Otherwise, there’s no need to expose your system.
/etc/rc.conf
# Remote logins
sshd_enable="YES"

hostname="MasterBSD"
# Enable DHCP for re0 and don't let dhclient block
background_dhclient="YES"
ifconfig_em0="DHCP"
ifconfig_em0_ipv6="inet6 accept_rtadv"

ifconfig_re0="inet 172.16.0.40 netmask 255.240.0.0 broadcast 172.31.255.255"
defaultrouter="172.16.0.1"

```


## /etc/fstab
```
proc	/proc	procfs	rw	0	0
fdesc	/dev/fd	fdescfs	rw,auto,late	0	0
```

##  /etc/devfs.conf
Permissions for devices existing at boot time are set in devfs.conf.
Each line defines a full device path and octal permission value.
```
# Allow all users to access optical media
perm    /dev/acd0       0666
perm    /dev/acd1       0666
perm    /dev/cd0        0666
perm    /dev/cd1        0666
# Allow all USB Devices to be mounted
perm    /dev/da0        0666
perm    /dev/da1        0666
perm    /dev/da2        0666
perm    /dev/da3        0666
perm    /dev/da4        0666
perm    /dev/da5        0666
# Misc other devices
perm    /dev/pass0      0666
perm    /dev/xpt0       0666
perm    /dev/uscanner0  0666
perm    /dev/video0     0666
perm    /dev/tuner0     0666
perm    /dev/dvb/adapter0/demux0    0666
perm    /dev/dvb/adapter0/dvr       0666
perm    /dev/dvb/adapter0/frontend0 0666
```

## /etc/devfs.rules
For devices that may be connected post-boot, we add an entry to a devfs.rules ruleset.
Rulesets must have a unique name and number, and their rules are composed of a path or quoted path glob and octal permission value

```
[devfsrules_common=7]
add path 'ad[0-9]\*'		mode 666
add path 'ada[0-9]\*'	mode 666
add path 'da[0-9]\*'		mode 666
add path 'acd[0-9]\*'	mode 666
add path 'cd[0-9]\*'		mode 666
add path 'mmcsd[0-9]\*'	mode 666
add path 'pass[0-9]\*'	mode 666
add path 'xpt[0-9]\*'	mode 666
add path 'ugen[0-9]\*'	mode 666
add path 'usbctl'		mode 666
add path 'usb/\*'		mode 666
add path 'lpt[0-9]\*'	mode 666
add path 'ulpt[0-9]\*'	mode 666
add path 'unlpt[0-9]\*'	mode 666
add path 'fd[0-9]\*'		mode 666
add path 'uscan[0-9]\*'	mode 666
add path 'video[0-9]\*'	mode 666
add path 'tuner[0-9]*'  mode 666
add path 'dvb/\*'		mode 666
add path 'cx88*' mode 0660
add path 'cx23885*' mode 0660 # CX23885-family stream configuration device
add path 'iicdev*' mode 0660
add path 'uvisor[0-9]*' mode 0660
```
Enable our new ruleset.
/etc/rc.conf
devfs_system_ruleset="devfsrules_common"


##
