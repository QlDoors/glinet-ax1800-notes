# 1. 扩展存储空间

> ⚠️<font color=red>**Warning: 一定要使用高速U盘，否则会因为U盘读写速度跟不上导致路由卡死！**</font>

由于AX1800的存储空间只有128MB，安装 OpenClash 后空间很紧张，所以我决定使用U盘来扩展存储空间。

以下操作都是以root用户SSH登录路由器之后操作的。

## 1.1 查看路由器是否识别U盘

输入命令：

```bash
df -h
```

查看U盘挂载信息。

## 1.2 卸载“网络存储”功能

输入命令：

```bash
umount /dev/sda1
```

如果不卸载网络存储功能，后续可能会报错，导致无法正常挂载。

## 1.3 安装相关工具

输入命令：

```bash
opkg update
opkg install block-mount  kmod-usb-storage  kmod-fs-ext4 e2fsprogs kmod-fs-vfat
```

## 1.4 查看U盘分区信息

输入命令：

```bash
blkid /dev/sda1
```

查看U盘信息，确认文件系统格式是否为 `TYPE="ext4"`，如果是，则跳过“步骤 1.5”。

## 1.5 格式化U盘

输入命令：

```bash
mkfs.ext4 /dev/sda1 << EOF
> EOF
```

## 1.6 制作根文件系统

输入命令：

```bash
mount -t ext4 /dev/sda1 /mnt
mkdir /tmp/root
mount -o bind / /tmp/root
cp /tmp/root/* /mnt -a
umount /tmp/root
umount /mnt
```

## 1.7 配置自动挂载

输入命令：

```bash
mv /etc/config/fstab /etc/config/fstab.bk
block detect > /etc/config/fstab
uci set fstab.@mount[0].target='/overlay'
uci set fstab.@mount[0].enabled='1'
uci commit fstab
```

## 1.8 备份 OpenWrt 系统

在“luci”页面的“系统 -> 备份/升级”找到“生成备份”，备份系统。这里只能备份系统的配置，无法备份已安装的软件包，系统重启后，需要重新安装软件包。

## 1.9 重启路由器

输入命令：

```bash
reboot
```

路由器重启后，进入管理页面，在软件包列表可以看到内存已经扩展，或是通过SSH查看。

## 1.10 恢复系统备份

在“luci”页面的“系统 -> 备份/升级”找到“上传备份”，恢复系统。恢复重启后，重新安装需要的软件包。

由于整个根文件系统已拷贝到U盘，因此在扩容成功后，原有软件和配置都会保留，无需重新配置。

# 2. 踩坑经历

由于我刚开始的U盘比较老，读写速度很慢，执行完“步骤 1.9”后路由器的“DHCP服务”都无法正常工作，只能想办法恢复路由器。

这是电脑不管是通过有线还是Wifi都无法正常连接路由，所以当时只能抱着死马当活马医治的态度，把路由器断电、拔U盘后重启，没想到路由恢复工作了！

路由恢复正常工作后，需要恢复之前备份的挂载配置文件。

```bash
rm /etc/config/fstab
mv /etc/config/fstab.bk /etc/config/fstab
```

然后，在某宝花巨资45元购买了一个读取速度为279MB/s的高速U盘，重新操作“步骤 1”，顺利完成了路由器的存储扩展。

# 引用

https://docs.gl-inet.cn/router/4/features/memory_mount/