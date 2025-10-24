# Docker容器学习笔记

## 操作系统

ProxmoxVE环境中有两个方式，虚拟机和CT容器，如果没有特殊要求，推荐CT容器。

**CT容器方式**

debian-12-standard_12.12-1_amd64.tar.zst

**虚拟机方式**

* ISO: debian-12.12.0-amd64-netinst.iso
* 语言: 英文
* 国家: 中国
* 分区: 默认
* 软件源: utsc
* 软件包: 桌面环境如果不需要，不要勾选。SSH 这个必备
* 其他默认

### 时区

```bash
ln -fs /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
#dpkg-reconfigure tzdata
# Asia/Shanghai'
```

### 语言
```bash
sed -i '/en_US.UTF-8/s/^#//g' /etc/locale.gen
sed -i '/zh_CN.UTF-8/s/^#//g' /etc/locale.gen
locale-gen
update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8
#dpkg-reconfigure locales
#  C.UTF-8... done
#  en_US.UTF-8... done
#  zh_CN.UTF-8... done
```

## 软件

### SSH服务

```bash
sed -i '/PermitRootLogin/ a PermitRootLogin yes' /etc/ssh/sshd_config
systemctl restart ssh
mkdir -p ~/.ssh/
chmod 700 ~/.ssh/
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
cat >> ~/.ssh/authorized_keys <<EOF
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDUsW1WyjMEZ9KGfA6C3SKavL6fYvqUY/yvki6kvuxrDEmlQEe/0y4MdzK6BZb98QICQylKH/5jJaWIu7lllLLopCNhWYcDG1p9EvRUircqO4RmG1QXJC06JXwux9EQiLShDRBsqu1HOLnohbdxHUM5Dte0aUbL6gO6UWhGYjwrN0xtglHmZ+HCey0G8G1TmwUjGZorJQs7u0xY3HMCMoxGjq8TXvN/FJJW5uzeES8VUliK4mXhmHXo+c1XzNXuKTnj8jsFKfiZuyYrnRGyvXir5t3WPSTQy1lmXbJ2frBRNAw3+gxIkV4r7WO2k++x5wAehDYoFNNcCEoy4JxoKB70BjJxCpekrDu/uu1s4uGUOR/FvJ5EoeU25KptIemL9mE4/Vw78vDfNwx1F4X0TBK2Cfq6wAiWhCk1ilcT880cc+nCBJbG8JW6/ZE9atvlg7ym7X1fGDhaqrXf9niGb5kP73GHGxYD59IY2amxlxnMn8t9giajV10jF6AzqOLwP6s= root@hlvps
EOF
```

### 软件源

```bash
# 软件源配置，如果想改，参考这个
cat > /etc/apt/sources.list <<EOF
## 默认禁用源码镜像以提高速度，如需启用请自行取消注释
deb http://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb http://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb http://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
## 安全更新软件源
deb http://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
EOF
mkdir -p /etc/apt/keyrings
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.ustc.edu.cn/docker-ce/linux/debian bookworm stable" | tee /etc/apt/sources.list.d/docker.list
apt update
apt install -y curl vim git neofetch iproute2 htop iftop iotop sysstat command-not-found
update-command-not-found
```

### Docker

```bash
# 如果指定版本，可以先查看哪些版本
# apt list -a docker-ce
# apt-cache madison docker-ce
# apt install -y docker-ce=5:28.1.1-1~debian.12~bookworm
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
mkdir -p /opt/container
cd /opt/container
# 如果想要独立命令，参考如下命令
# wget https://github.com/docker/compose/releases/download/v2.36.2/docker-compose-linux-x86_64
# mv docker-compose-linux-x86_64 /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
# 如果国内需要镜像
sudo tee /etc/docker/daemon.json <<EOF
{
    "registry-mirrors": [
        "https://docker.mirrors.sjtug.sjtu.edu.cn",
        "https://9bc7vgcd.mirror.aliyuncs.com",
        "https://docker.m.daocloud.io",
        "https://docker.nju.edu.cn",
        "https://dockerproxy.com"
    ]
}
EOF

systemctl restart docker.service
```