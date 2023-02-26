# 0. SSH 连接至路由器

> 此前现需要讲RSA密钥添加至 OpenWrt Web管理端 路径一般为 ~/.ssh/known_host
> 复制密钥到下图位置中
![在这里插入图片描述](https://img-blog.csdnimg.cn/ca0d2aa017c7403bae2d65d252e4d8e6.png#pic_center)

然后

```bash
ssh root@192.168.1.1 
```

> 成功连接

![成功连接](https://img-blog.csdnimg.cn/6703ebb9315f46b19621f7e2621516a7.png#pic_center)

# 1. OpenClash下载安装

## (1)依赖安装

```bash
#依赖安装
#iptables
opkg update
opkg install coreutils-nohup bash iptables dnsmasq-full curl ca-certificates ipset ip-full iptables-mod-tproxy iptables-mod-extra libcap libcap-bin ruby ruby-yaml kmod-tun kmod-inet-diag unzip luci-compat luci luci-base

```
> 这个步骤中缺少的安装包可以[在此查找并下载](https://op.supes.top/packages/)，手动上传包安装

## (2)客户端IPK下载 [前往下载](https://github.com/vernesong/OpenClash/releases)：

> 注意需要需要对应路由器的架构进行IPK包的选择，具体架构可以ssh到路由器输入以下指令查询

```bash
cat /proc/cpuinfo
```
如我所示的TP-Link AC1750 `cpu model`一栏即为CPU架构

![architecture](/home/liuboyang/Pictures/architecture.png)

## (3) 上传包

> 在OpenWrt Web端管理界面，顶端`系统`->`SoftWare`

> 进入后选择本地的IPK包上传并安装
>
> 安装完成后刷新页面，在顶端的`服务`就可以找到OpenClash了

## 

 # 修复：Overlay空间不足，解决办法：U盘挂载


```bash
opkg install block-mount e2fsprogs kmod-fs-ext4 kmod-usb-storage kmod-usb2 kmod-usb3

```
## 格式化U盘/SD卡，假设U盘设备节点为/dev/sda1：

```bash
mkfs.ext4 /dev/sda1 << EOF
> Y
> EOF
#mke2fs 1.44.1 (24-Mar-2018)
#Creating filesystem with 3839992 4k blocks and 960992 inodes
#Filesystem UUID: ccb794c1-4ac4-401a-a5bf-f49344043014
#Superblock backups stored on blocks: 
#	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

#Allocating group tables: done                            
#Writing inode tables: done                            
#Creating journal (16384 blocks): done
#Writing superblocks and filesystem accounting information: done  
```
## 给U盘/SD卡制作根文件系统：

```bash
mount /dev/sda1 /mnt
mkdir /tmp/root
mount -o bind / /tmp/root
cp /tmp/root/* /mnt -a
umount /tmp/root
umount /mnt
```
## 配置自动挂载并重启路由

```bash
block detect > /etc/config/fstab
uci set fstab.@mount[0].target='/overlay'
uci set fstab.@mount[0].enabled='1'
uci commit fstab
reboot
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/e767d30650774e98bc60daada9173590.png#pic_center)

> 至此完毕

