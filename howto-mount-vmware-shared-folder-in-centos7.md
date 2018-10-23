# How to Mount VMware Shared Folder in CentOS 7

## Add Shared Folder in VMware Preferences

`shared`

## Install VMWare Tools

```sh
yum install -y open-vm-tools
```

If you use CentOS with X11, install `open-vm-tools-desktop` too:

```sh
yum install -y open-vm-tools open-vm-tools-desktop
```

## Mount Shared Folder in CentOS Guest OS

```sh
mkdir -p /data/shared
/usr/bin/vmhgfs-fuse .host:/shared /data/shared -o uid=$(id -u www),gid=$(id -g www),umask=0022,subtype=vmhgfs-fuse,allow_other
```

## Umount If Necessary:

```sh
umount /data/shared
```
