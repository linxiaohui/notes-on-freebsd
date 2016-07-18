# No pkg(8) database found!

Due to an incompatibility between bsdconfig(8) and pkg(8) version 1.3,
packages included on the FreeBSD dvd installer will not be recognized by bsdconfig(8).

To install packages from the dvd1.iso installer, create the /dist target directory, and manually mount the dvd1.iso ISO:
```
mkdir -p /dist
mount -t cd9660 /dev/cd0 /dist
export REPOS_DIR=/dist/packages/repos
#OR
setenv REPOS_DIR /dist/packages/repos

pkg bootstrap
pkg install xorg-server xorg gnome2 [...]
```

# 如何从ISO安装packages
```
mount -t cd9660 /dev/cd0  /dist
mkdir -p /usr/local/etc/pkg/repos
/usr/local/etc/pkg/repos
cp /dist/PACKAGES/REPOS/FREEBSD_INSTALL_CDROM.conf   ./
```
# FreeBSD.conf

FreeBSD: {
enabled: no                     //change to yes
}
