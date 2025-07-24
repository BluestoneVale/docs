# MacOS学习笔记

整理学习笔记持续更新。

## 硬件

**兼容机**

大概1995年开始接触计算机，从80386开始，然后80486、80586。
直到1997年左右家里买的第一台计算机(联想台式，1万元左右、MMX奔腾166Mhz、内存16M、硬盘2.1G)。
在这之后都是自己组装机，这些年算下来也有5台左右。
很是感慨科技更新速度之不可思议。

**Macbook Pro 2010**

|属性|内容|
|:-:|:-:|
|模具型号|A1278|
|处理器|2.66G GHz 双核|
|内存|8 GB|
|硬盘1|320G HDD|
|硬盘2|120G SSD|
|系统|Mac OS X Yosemite 10.10|

2010年1万元购入13寸笔记本，由黑转白，后升级内存和硬盘，已经失踪。

**Macmini 2012**

|属性|内容|
|:-:|:-:|
|模具型号|A1347|
|处理器|2.3 GHz 四核Intel Core i7|
|内存|16 GB 1600 MHz DDR3|
|图形卡|Intel HD Graphics 4000 1536 MB|
|硬盘1|Samsung SSD 840 EVO 250GB|
|硬盘2|Samsung SSD 860 EVO 1TB|
|系统|macOS Catalina 10.15.7|

2015年二手4600元左右购入，后升级内存和硬盘，截止2025年仍然在用，放公司轻量办公用。

**Macbook Pro 2015**

|属性|内容|
|:-:|:-:|
|模具型号|A1502|
|处理器|2.7GHz 双核 Intel Core i5 处理器|
|内存|8GB 1866MHz LPDDR3|
|图形卡|Intel Iris Graphics 6100|
|硬盘1|西部数据 SSD SN550 1TB|
|系统|macOS Catalina 10.15.7|

2018年从公司离职折价购入，后升级硬盘，目前基本闲置。

**Macmini 2018**

|属性|内容|
|:-:|:-:|
|模具型号|A1993|
|处理器|3.2 GHz 六核Intel Core i7|
|内存|64 GB 2667 MHz DDR4|
|图形卡|Intel UHD Graphics 630 1536 MB|
|硬盘1|APPLE SSD AP2048M 2TB|
|系统|macOS Sequoia 15.4|

2022年2万元左右购入，考虑需要x86环境选择最后一代使用Intel处理器的Macmini，截止2025年仍然在用，放家里用。

**Macmini 2024**

|属性|内容|
|:-:|:-:|
|模具型号|A3238|
|处理器|Apple M4 芯片 10 核中央处理器|
|内存|16GB 统一内存|
|图形卡|10 核图形处理器|
|硬盘1|256GB 固态硬盘|
|系统|macOS Sequoia 15.4|

2025年3200元左右购入，相对2018那台，直观感受速度快温度低。

Intel MacOS Catalina 10.15.7 虚拟化应用推荐免费的VMware Fusion和Docker Desktop。
Apple Silicon MacOS Sequoia 15.4 虚拟化应用推荐UTM和Orbstack，x86虚拟体验不好。

>新方案MacOS和ProxmoxVE组合
缺点：网络远程或携带小主机，前者在移动办受公网络不稳定影响，后者小主机供电和携带增加负荷也需要妥协。
优点：本地硬件配置选丐版压低成本，ProxmoxVE服务器的选配丰富，应用及数据安全都有提升。


## 软件

**操作系统**

桌面操作系统从最开始MSDOS-6.22开始、然后Windows系列(95/98/me/xp/7/10)、UbuntuLTS系列(6/8)、黑苹果(10)、白苹果(10/11/12/13/14/15)。
几经尝试最终逐步过度苹果系统 MacOS，极少数必须使用Windows应用的时候，本地或PVE环境启动虚拟机Windows(10/11)解决。

从接触MacOS开始，桌面环境就基本固定不变了，以下使用笔记。

### 基本工具 Xcode

需要确保系统中安装了 bash、git 和 curl，对于 macOS 用户需额外要求安装 Command Line Tools (CLT) for Xcode。

```bash
xcode-select --install
```

### 软件管理 Brew

需要安装第三方软件，可以使用brew，源码或二进制包都支持。

* 官网：https://brew.sh/zh-cn/
* 清华大学镜像：https://mirrors.tuna.tsinghua.edu.cn/help/homebrew/
* 其他镜像：https://cloud.tencent.com/developer/article/2486985

**卸载**

自动识别芯片架构（Intel/Apple Silicon）
删除 /usr/local（Intel）或 /opt/homebrew（Apple Silicon）下的所有文件
清理缓存目录 ~/Library/Caches/Homebrew

```bash
# 方法1，官网
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/uninstall.sh)"

# 方法2，国内镜像
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/HomebrewUninstall.sh)"

# 方法3，脚本失效场景
# Apple Silicon 芯片专用
sudo rm -rf /opt/homebrew
rm -rf ~/Library/Caches/Homebrew
# Intel 芯片专用
sudo rm -rf /usr/local/Cellar /usr/local/Homebrew
rm -rf /usr/local/var/homebrew
```

**安装**
```bash
# 方法1，官网
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 方法2，清华大学镜像
cd $HOME/WorkSpace
# 从镜像下载安装脚本并安装 Homebrew / Linuxbrew
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"
export HOMEBREW_INSTALL_FROM_API=1
git clone --depth=1 https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/install.git brew-install
/bin/bash brew-install/install.sh
rm -rf brew-install

# 变量加入初始化环境
echo >> $HOME/.zprofile
echo 'eval "$(/usr/local/bin/brew shellenv)"' >> $HOME/.zprofile
eval "$(/usr/local/bin/brew shellenv)"
echo '# Set non-default Git remotes for Homebrew/brew and Homebrew/homebrew-core.' >> $HOME/.zprofile
echo 'export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"' >> $HOME/.zprofile
echo 'export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"' >> $HOME/.zprofile
echo 'export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"' >> $HOME/.zprofile
echo 'export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"' >> $HOME/.zprofile

# 检查配置
brew config
# 安装软件
brew install ipcalc
```

brew tap 命令用于在 Homebrew 中添加第三方仓库，以便安装更多的软件包。默认情况下，Homebrew 的仓库源来自 GitHub，但也可以是其他任何位置和协议的仓库
```bash
# 添加
brew tap 
# 删除
brew untap
```

**更新**
```bash
brew update
```

### 终端 Terminal/iTerm2

命令行工具

* 安装字体：Menlo for Powerline，可以解决zsh提示符问号
* 下载：https://github.com/BluestoneVale/docs/blob/main/fonts/Menlo%20for%20Powerline.ttf  
* 系统自带 Terminal
  * 菜单：终端-偏好设置-描述文件
    * HomeBrew 设置默认
  * 字体选择 Menlo for Powerline 字号选择 18
  * 优点：系统自带
  * 缺点：功能少，不支持lrzsz
* 免费 iTerm2
  * 下载：https://iterm2.com/
  * 优点：支持lrzsz
  * 缺点：相对前者资源消耗多些
  

### 终端 Zoc

命令行工具

* 官网：https://www.emtec.com/download.html#zocfiles
* 优点：
  * 支持会话管理
  * 功能非常丰富
  * 试用30天后，忽略警告提示可以继续使用
* 缺点：
  * 界面不支持中文
  * 中文输入有问题


### SHELL Oh-My-Zsh

增强SHELL，可以适当操作效率。

* 官网：https://ohmyz.sh/
* 清华大学镜像：https://mirrors.tuna.tsinghua.edu.cn/help/ohmyzsh.git/

**卸载**
```bash 
uninstall_oh_my_zsh
```

**安装**
```bash
# 方法1，官网
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 方法2，清华大学镜像
cd $HOME/WorkSpace
git clone https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
cd ohmyzsh/tools
REMOTE=https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git sh install.sh

# 切换已有 ohmyzsh 至镜像源
git -C $ZSH remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
git -C $ZSH pull
```

**更新**
```bash
# 方法1
~/.oh-my-zsh/tools/upgrade.sh 

# 方法2
cd ~/.oh-my-zsh
git pull

# 方法3
upgrade_oh_my_zsh
```

**主题**
```bash
# 将主题修改为 ZSH_THEME="agnoster"
vim ~/.zshrc
# 立即加载
source .zshrc
```

**插件**

git (默认安装)
插件功能：展示当前分支

zsh-syntax-highlighting
插件功能：这个包为shell zsh提供语法突出显示。它允许高亮显示在zsh提示符下输入到交互式终端的命令。这有助于在运行命令之前检查它们，特别是在捕获语法错误方面。例：在你输入某个命令时，如果该命令不存在，那么它显示为红色；否则，它会变成绿色。

zsh-autosuggestion
插件功能：输入命令时可提示自动补全（灰色部分），按tab键（→ ）即可补全。

```bash
# 克隆项目到本地 $ZSH_CUSTOM/plugins 路径下，默认是 ~/.oh-my-zsh/custom/plugins
# 官网 zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
# 镜像 zsh-autosuggestions
git clone https://gitee.com/phpxxo/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
# 官网 zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
# 镜像 zsh-syntax-highlighting
git clone https://gitee.com/huo15-study/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 将插件修改为 plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
vim ~/.zshrc
# 立即加载
source .zshrc
```

### 代理

建议在香港或其他国家租用一个虚拟机，可以在闲鱼上找到，100左右每年的就够，建议独立资源和专属IP。
虚拟机初始密码设置一个复杂点的，有条件的再部署一个 fail2ban 服务
生成密钥对命令：ssh-keygen
上传公钥命令：ssh-copy-id vps 

**SSH本地配置**

文件 ~/.ssh/config，建议把这个文件通过私有git仓库管理，在多个桌面环境切换时，保持习惯不变。

```text
# 虚拟机
Host vps
    HostName <虚拟机专属IP>
    User root
    Port 22
    ServerAliveInterval 60
    TCPKeepAlive yes
    ForwardAgent yes
    GatewayPorts yes
    StrictHostKeyChecking no

# 本地开Socks5代理，地址端口 127.0.0.1:7070
Host vps-7070
    HostName <虚拟机专属IP>
    User root
    Port 22
    ServerAliveInterval 60
    TCPKeepAlive yes
    ForwardAgent yes
    GatewayPorts yes
    StrictHostKeyChecking no
    DynamicForward 127.0.0.1:7070

# 本地开隧道，访问本地 127.0.0.1:7080 转发到虚拟机 127.0.0.1:7080
# 可以在虚拟机上部署 3proxy（http_proxy https_proxy） 服务，限制仅 127.0.0.1 可以访问
Host vps-7080
    HostName <虚拟机专属IP>
    User root
    Port 22
    ServerAliveInterval 60
    TCPKeepAlive yes
    ForwardAgent yes
    GatewayPorts yes
    StrictHostKeyChecking no
    LocalForward 127.0.0.1:7080 127.0.0.1:7080

# 拉取 github.com 代码走 vps 跳板
Host github.com
    HostName github.com
    User git
    Port 22
    StrictHostKeyChecking no
    ProxyCommand ssh -q -W %h:%p vps

```

**Shell代理配置**

```bash
# 设置代理
export http_proxy="http://127.0.0.1:7080"
export https_proxy="http://127.0.0.1:7080"
# 适应面小
export socks5_proxy="socks5://127.0.0.1:7070"
# 撤销代理
unset http_proxy
unset https_proxy
unset socks5_proxy
```

**Git代理配置**

如果想git拉取代码全部走代理，可以有以下这些选择

```bash
# 设置 HTTP HTTPS 全局代理 HTTP
git config --global http.proxy http://127.0.0.1:7080
git config --global https.proxy http://127.0.0.1:7080
# 设置 HTTP HTTPS 全局代理 SOCKS5 很多应用不支持
git config --global http.proxy socks5://127.0.0.1:7070
git config --global https.proxy socks5://127.0.0.1:7070
# 取消 HTTP HTTPS 全局代理
git config --global --unset http.proxy
git config --global --unset https.proxy
# 查看全局代理
git config --global --get http.proxy
git config --global --get https.proxy
# 仅 https://github.com 走代理
git config --global http.https://github.com.proxy http://127.0.0.1:7080
git config --global https.https://github.com.proxy http://127.0.0.1:7080
# 或
git config --global http.https://github.com.proxy socks5://127.0.0.1:7070
git config --global https.https://github.com.proxy socks5://127.0.0.1:7070
```

**浏览器代理配置**

  * Firefox 应用
  * 下载：https://www.firefox.com/zh-CN/ 
  * 菜单：Firefox-偏好设置-常规：网络设置
  * 手动配置代理：
    * SOCKS 主机：127.0.0.1
    * 端口：7070
    * 勾选：使用 SOCKS v5 时代理 DNS 查询
