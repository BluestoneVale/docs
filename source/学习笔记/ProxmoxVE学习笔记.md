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
放家里自建各类服务，选择这个考虑带外管理（vPro 需要插HDMI欺骗器）。

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
