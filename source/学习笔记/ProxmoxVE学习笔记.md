# ProxmoxVE学习笔记

## 硬件

**Centerm C92**

|属性|内容|
|:-:|:-:|
|型号|Centerm C92|
|处理器|Intel Celeron J1900 (4) @ 1.999GHz|
|内存|DDR3 8 GB|
|硬盘|mSATA SSD 256GB|
|系统|Proxmox VE 8.1.4 x86_64|

2024年二手380元左右购入，目的体验此类小主机，功耗低稳定。

**中柏 N100 Pro**

|属性|内容|
|:-:|:-:|
|型号|中柏 n100pro|
|处理器|Intel N100 (4) @ 3.400GHz|
|内存|DDR4 16 GB|
|硬盘|SATA SSD 800GB|
|硬盘|Nvme SSD 512GB|
|系统|Proxmox VE 8.1.4 x86_64|

2024年1200元左右购入，后扩容硬盘，放公司提供存储服务。

**中柏 N305**

|属性|内容|
|:-:|:-:|
|型号|中柏 n305|
|处理器|Intel N305 (4) @ 3.800GHz|
|内存|DDR4 32 GB|
|硬盘|SATA SSD 800GB|
|系统|Proxmox VE 8.1.4 x86_64|

2024年1300元左右购入准系统，另配内存和硬盘，配合MacMiniM4使用。

**HP 800G4 SFF**

|属性|内容|
|:-:|:-:|
|型号|HP 800G4 SFF|
|处理器|Intel i5-8600T (6) @ 3.700GHz|
|内存|DDR4 64 GB|
|硬盘|Nvme ZHITAI TiPlus5000 1TB|
|系统|Proxmox VE 8.1.4 x86_64|

2024年二手330元左右购入准系统，内存550元左右，硬盘440元左右，处理器300元左右。
放家里部署NAS，选择这个考虑带外管理（vPro 需要插HDMI欺骗器）。

## 软件

### 安装
* BIOS设置
  * 开启VT-x虚拟化支持
  * 开启VT-o虚拟化IO影射支持
* PVE 安装包 8.1.4 ，内核 6.5.11-8 。
* 大于 2G 的 U 盘（存放 Proxmox Virtual Environment ISO 镜像）
* 刻录工具（ 推荐 balenaEtcher ）
https://github.com/balena-io/etcher/releases/tag/v1.19.21
* Proxmox Virtual Environment ISO 镜像
https://enterprise.proxmox.com/iso/proxmox-ve_8.1-2.iso
 
### pvetools工具准备（本地）

* 下载：https://github.com/ivanhao/pvetools/releases/download/pve8/pvetools-pve8.0.3.zip
* 上传：/opt


### 软件安装
```bash
rm /etc/apt/sources.list.d/pve-enterprise.list
apt-get update
apt-get install -y vim unzip
```

### 切换软件源
```bash
cd /opt
unzip pvetools-pve8.0.3.zip
cd pvetools-pve8.0.3
./pvetools.sh 
```
关闭企业源，切换国内源ustc.edu.cn，关闭订阅

### local-lvm 删除

```bash
lvremove pve/data
lvextend -l +100%FREE -r pve/root
resize2fs /dev/mapper/pve-root
# 网页登陆，数据中心-存储-删除local-lvm
# 编辑local，内容里添加 磁盘映像和容器，保存
```

### VT-D 直通
```bash
# vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream pci=realloc=off"

# 更新
update-grub

# vim /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd

# 更新
update-initramfs -u -k all

# 重启
reboot

# 检查是否开启
dmesg | grep -e DMAR -e IOMMU
``` 

### UPS 配置

硬件是施耐德 APC BK650M2-CH 的 UPS（不间断电源）。
配合 apcupsd 使用，满足普通家庭用户需求。

**UPS物理连接主机**
将 UPS 电源线连接市电并开机，把服务器关机，服务器主机电源线插到 UPS 的那些供电插口上。把随 UPS 一起附赠的信号线网线头查到 UPS 的信号输出口，USB 头插到主机后面的 USB 上。

**检查信号线路**
使用命令 lsusb 确认服务器上已经识别到 UPS了。

```bash
root@hp800g4sff:~# lsusb 
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 010: ID 051d:0002 American Power Conversion Uninterruptible Power Supply
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

**安装 Apcupsd 服务**

```bash
# 安装软件
apt install apcupsd
# 备份配置
cp /etc/apcupsd/apcupsd.conf /etc/apcupsd/apcupsd.conf.bak
```

**配置 Apcupsd 服务**
编辑配置文件 /etc/apcupsd/apcupsd.conf

```bash
# 连接线材：USB
UPSCABLE usb
# 连接方式：USB
UPSTYPE usb
# 串口信息，这个需要注释或者配置为置空
# DEVICE /dev/ttyS0
# 停电后，电池开始供电多少秒后，开始关闭系统。
TIMEOUT 60
# 电池供电多少秒后，关闭 UPS。这里设置为 0 禁用，担心主机没有完全关机，ups 就主动断电了
KILLDELAY 0
# 停电后使用电池供电时，电池电量剩余小于等于 30% 时，执行关闭系统操作
BATTERYLEVEL 30
# 停电后使用电池供电时，电池电量供电剩余时间小于 10 分钟时，执行关闭系统操作
MINUTES 3
# 从检测到电源故障到 apcupsd 对事件做出反应的秒数
ONBATTERYDELAY 6
```

**调整 Apcupsd 服务**
```bash
systemctl restart apcupsd
systemctl status apcupsd
systemctl enable apcupsd
```

**查看 Apcupsd 服务状态**
```bash
root@hp800g4sff:~# apcaccess
APC      : 001,035,0846
DATE     : 2025-07-24 15:16:33 +0800  
HOSTNAME : hp800g4sff
VERSION  : 3.14.14 (31 May 2016) debian
UPSNAME  : hp800g4sff
CABLE    : USB Cable
DRIVER   : USB UPS Driver
UPSMODE  : Stand Alone
STARTTIME: 2025-07-24 11:54:56 +0800  
MODEL    : Back-UPS BK650M2-CH 
STATUS   : ONLINE 
LINEV    : 222.0 Volts
LOADPCT  : 12.0 Percent
BCHARGE  : 100.0 Percent
TIMELEFT : 60.3 Minutes
MBATTCHG : 30 Percent
MINTIMEL : 3 Minutes
MAXTIME  : 60 Seconds
SENSE    : Low
LOTRANS  : 160.0 Volts
HITRANS  : 278.0 Volts
ALARMDEL : 30 Seconds
BATTV    : 13.5 Volts
LASTXFER : No transfers since turnon
NUMXFERS : 0
TONBATT  : 0 Seconds
CUMONBATT: 0 Seconds
XOFFBATT : N/A
SELFTEST : OK
STATFLAG : 0x05000008
BATTDATE : 2001-01-01
NOMINV   : 220 Volts
NOMBATTV : 12.0 Volts
NOMPOWER : 390 Watts
FIRMWARE : 0000 00 -292804G 
END APC  : 2025-07-24 15:17:10 +0800  
```
注意看状态 STATUS   : ONLINE，说明当前市电正常，负载百分比：LOADPCT。
参考资料：https://beltxman.com/4560.html

