# Zerotier虚拟网络学习笔记

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

```yaml
version: '3.8'
services:
  zerotier-app:
    image: keynetworks/ztncui:1.2.16
    container_name: zerotier-app
    hostname: zerotier-app
    network_mode: bridge
    restart: always
    environment:
      - NODE_ENV=production
      - HTTPS_HOST=0.0.0.0
      - HTTPS_PORT=3443
      - MYDOMAIN=ztncui.blues.com
      - MYADDR=
      - HTTP_PORT=3000
      - HTTP_ALL_INTERFACES=yes
      - ZTNCUI_PASSWD=
      - TZ=Asia/Shanghai
    ports:
      - '3000:3000'
      - '3443:3443'
      - '9993:9993'
      - '9993:9993/udp'
      - '3180:3180'
    volumes:
      - './zerotier-one:/var/lib/zerotier-one'
      - './ztncui/etc:/opt/key-networks/ztncui/etc'
    logging:
      driver: "json-file"
      options:
        max-size: "1g"
```