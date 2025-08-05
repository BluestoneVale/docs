# ProxmoxVE集群学习笔记

## 安装

### 准备

* BIOS设置
  * 开启VT-x虚拟化支持
  * 开启VT-o虚拟化IO影射支持
* Proxmox Virtual Environment ISO 镜像：https://enterprise.proxmox.com/iso/proxmox-ve_8.4-1.iso
* 大于 2G 的 U 盘（存放 Proxmox Virtual Environment ISO 镜像）
* 刻录工具（ 推荐 balenaEtcher ）：https://github.com/balena-io/etcher/releases/tag/v1.19.21

### 规划

* 虚拟机，名称，地址，网关
  * pve-node-1.local 192.168.1.201/24 192.168.1.1
  * pve-node-2.local 192.168.1.202/24 192.168.1.1
  * pve-node-3.local 192.168.1.203/24 192.168.1.1 
  * pve-node-4.local 192.168.1.204/24 192.168.1.1 
  * PS
    * pve-node-3模拟故障，pve-node-4模拟故障后新增 
* 系统-页签
  * 机型：Q35
* 磁盘-页签
  * scsi0 sda 系统盘
  * scsi1 sdb 数据盘
  * scsi2 sdc 数据盘-DB盘
  * scsi3 sdd 数据盘-WAL盘
  * PS
    * 数据盘：用于存储数据的磁盘，为节省成本，通常采用HDD磁盘。
    * DB盘：高速缓存，用于存储BlueStore内部产生的元数据文件，可采用普通SSD。
    * WAL盘：超高速缓存，用于存储RocksDB log文件，可以采用性能更高、延迟更低的SSD或NVMe SSD。
* CPU
  * 1 核
  * 类别：Host
* 内存
  * 12048
  * PS
    * 8GB内存使用率接近90%
* 网络
  * 默认

## 初始化

### 免密登录（本地）

```bash
ssh-copy-id pve-node-1
ssh-copy-id pve-node-2
ssh-copy-id pve-node-3
ssh-copy-id pve-node-4
```

### pvetools工具准备（本地）

下载： https://github.com/ivanhao/pvetools/releases/download/pve8/pvetools-pve8.0.3.zip，上传：/opt

```bash
scp pvetools-pve8.0.3.zip pve-node-1:/opt/
scp pvetools-pve8.0.3.zip pve-node-2:/opt/
scp pvetools-pve8.0.3.zip pve-node-3:/opt/
scp pvetools-pve8.0.3.zip pve-node-4:/opt/
```

### 软件安装

```bash
rm /etc/apt/sources.list.d/pve-enterprise.list
apt-get update
apt-get install -y vim unzip
```

### 环境设置

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

## 实验科目1：创建PVE集群

### pve-node-1 操作
数据中心-集群，创建集群：mycluster
数据中心-集群，加入信息：复制信息

### pve-node-2、pve-node-3 操作
加入集群，输入口令和选择本地地址

页面出现报错: '/etc/pve/nodes/pve-node-2/pve-ssl.pem' does not exist! (500)
重新刷新后，集群加入正常。

网络架构设计参考：https://zhuanlan.zhihu.com/p/617024637
集群使用参考：https://wiki.sqlfans.cn/it/pve-install-cluster.html
其他参考文档：https://blog.csdn.net/hanziqing0630/article/details/114262035

## 实验科目2：PVE-Ceph集群创建

* 软件源准备

```bash
cat > /etc/apt/sources.list.d/ceph.list <<EOF
# deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy/ bookworm no-subscription
EOF
cp /usr/share/perl5/PVE/CLI/pveceph.pm /usr/share/perl5/PVE/CLI/pveceph.pm_back
sed -i 's|http://download.proxmox.com|https://mirrors.ustc.edu.cn/proxmox|g' /usr/share/perl5/PVE/CLI/pveceph.pm
apt update
```

* pve-node-1 pve-node-2、pve-node-3 操作
  * 数据中心-节点-Ceph，reef (18.2)，无订阅，回答Y
  * Public Network，选择本地网段 192.168.1.201，其他默认，最后完成

* pve-node-1 操作
  * Ceph-监视器，管理员-创建，pve-node-2、pve-node-3。
  * Ceph-CephFS，元数据服务器-创建，pve-node-1、pve-node-2、pve-node-3
 
* pve-node-1 pve-node-2、pve-node-3 操作
  * Ceph-OSD，创建OSD
    * 磁盘：sdb
    * DB盘：sdc
    * WAL盘：sdd
    * 设备类型：ssd

* pve-node-1 操作
  * Ceph-CephFS，创建CephFS，名称cephfs
  * cephfs存储，CT模版，添加debian12
  * Ceph-资源池，创建Ceph_pool，名称ssd_data


其他参考文档：https://zhuanlan.zhihu.com/p/617024637

## 实验科目3：PVE-HA 高可用集群

* pve-node-1 操作 创建虚拟机
  * 创建测试CT虚拟机
    * 名称：lxc-ha
    * 192.168.1.210/24
  * 创建测试VM虚拟机
    * 名称：vm-ha
    * 192.168.1.118/24 dhcp
  
* 数据中心 操作
  * 数据中心-HA，资源-添加，选择两个虚拟机。

## 实验科目4：PVE-HA 迁移

* 不使用ceph也可以迁移，迁移的时候local存储会复制虚拟机镜像到目标机器。故障的时候不行。
* 页面操作手动迁移两个虚拟机到pve-node-3，验证
* 强制pve-node-3关机，等待自动迁移。验证
  
## 实验科目4：PVE集群、PVE-Ceph集群、新增

* pve-node-4 操作, 
  * 加入集群，输入口令和选择本地地址
  * 准备ceph软件源，安装ceph
  * Ceph-监视器，监视器-创建，pve-node-4
  * Ceph-监视器，管理员-创建，pve-node-4
  * Ceph-CephFS，元数据服务器-创建，pve-node-4
  * Ceph-OSD，创建OSD
    * 磁盘：sdb
    * DB盘：sdc
    * WAL盘：sdd
    * 设备类型：ssd
  * 手动迁移应用验证

## 实验科目5：故障节点回归

  * 默认自动隔离
  * 清空集群数据后可重新加入集群

pve-node-1 操作
```bash
pvecm delnode pve-node-3
```

pve-node-3 操作
```bash
systemctl stop pve-cluster.service
systemctl stop corosync.service
pmxcfs -l #强制设置为本地模式
cd /etc/pve/
rm corosync.conf
rm -rf /etc/corosync/
killall pmxcfs
systemctl start pve-cluster.service
rm -rf /etc/pve/nodes/*
```
