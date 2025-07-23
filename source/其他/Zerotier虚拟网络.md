# Zerotier虚拟网络

* 官网：https://www.zerotier.com/download/
* 优点：
  * 支持内网穿透
  * 各种平台支持
* 缺点：
  * 国内建议租一个有公网IP的虚拟机，部署节点
  * 不支持域名

**Linux客户端**

```bash
curl -s https://install.zerotier.com | bash

# 如果有自建节点配置，以下操作
mkdir -p /var/lib/zerotier-one/moons.d/
/bin/cp 自建节点000000cd4c90ec5c.moon /var/lib/zerotier-one/moons.d/
/bin/cp 自建节点planet /var/lib/zerotier-one/
systemctl restart zerotier-one.service

# 常用命令
zerotier-cli join XXX
zerotier-cli leave XXX
zerotier-cli listmoons
zerotier-cli listpeers
zerotier-cli listnetworks
```

**自建节点**

待补充
