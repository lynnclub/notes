---
title: "镜像"
type: "docs"
weight: 2
---

### 国内镜像源

南科大，北清华，遇事不决阿里云。

清华大学镜像，北方推荐，教育网 [https://mirrors.tuna.tsinghua.edu.cn/](https://mirrors.tuna.tsinghua.edu.cn/)

中国科学技术大学镜像，南方推荐，教育网 [https://mirrors.ustc.edu.cn/](https://mirrors.ustc.edu.cn/)

阿里云镜像，全国可用 [https://developer.aliyun.com/mirror/](https://developer.aliyun.com/mirror/)

注意：阿里云 ECS 用户，可以使用内网地址 `http://mirrors.cloud.aliyuncs.com/`。阿里云国内构建节点位于北京。

此外还有：  
[https://mirrors.163.com/](https://mirrors.163.com/)  
[https://mirrors.tencent.com/](https://mirrors.tencent.com/)  
[https://mirrors.huaweicloud.com/](https://mirrors.huaweicloud.com/)  
[https://mirrors.nju.edu.cn](https://mirrors.nju.edu.cn)  

### GitHub

获取全球IP绑定host，建议用日韩的IP  
[https://www.itdog.cn/ping/github.com](https://www.itdog.cn/ping/github.com)  
[https://www.itdog.cn/ping/raw.githubusercontent.com](https://www.itdog.cn/ping/raw.githubusercontent.com)

```
20.200.245.247 github.com
185.199.109.133 raw.githubusercontent.com
```

或者使用 [https://gitclone.com/](https://gitclone.com/)

```shell
#方法一（替换URL）
git clone https://gitclone.com/github.com/containerd/containerd.git

#方法二（设置git参数）
git config --global url."https://gitclone.com/".insteadOf https://
```

### Docker

[https://www.docker.com](https://www.docker.com/)

```shell
# 修改源
sudo vi /etc/docker/daemon.json
# 重启
sudo systemctl restart docker
# 查看配置
docker info
```

DaoCloud与轩辕镜像源
```json
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io",
    "https://docker.xuanyuan.me"
  ]
}
```

### Containerd

k8s默认使用containerd作为容器。

[https://containerd.io/downloads/](https://containerd.io/downloads/)

Centos
```shell
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y containerd.io
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl enable --now containerd

#重启
sudo systemctl restart containerd
```

vi /etc/containerd/config.toml  
用于k8s需要开启cri  
kubeadm初始化时会将kubelet配置为systemd，containerd需要开启
```yaml
#disabled_plugins = ["cri"]
SystemdCgroup = true
```

/etc/containerd/certs.d/registry.k8s.io/host.toml

```toml
server = "https://registry.k8s.io"

[host."https://k8s.m.daocloud.io"]
  capabilities = ["pull", "resolve"]
```

/etc/containerd/certs.d/docker.io/host.toml

```toml
server = "https://docker.io"

[host."https://docker.m.daocloud.io"]
  capabilities = ["pull", "resolve"]

[host."https://docker.xuanyuan.me"]
  capabilities = ["pull", "resolve"]
```

### Kubernetes

[https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

[https://developer.aliyun.com/mirror/kubernetes](https://developer.aliyun.com/mirror/kubernetes)

Debian/Ubuntu安装k8s，注意更换版本。kubesphere v4.1.3建议v1.21至v1.30。

```shell
apt update && apt install -y apt-transport-https

VERSION="v1.30"
curl -fsSL https://mirrors.aliyun.com/kubernetes-new/core/stable/$VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://mirrors.aliyun.com/kubernetes-new/core/stable/$VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list

apt update
apt install -y kubelet kubeadm kubectl
systemctl enable kubelet --now
```

CentOS/RHEL/Fedora

```shell
cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.30/rpm/repodata/repomd.xml.key
EOF

setenforce 0
yum install -y iproute-tc kubernetes-cni kubelet kubeadm kubectl
systemctl enable kubelet --now
```

### Helm

Helm 是 Kubernetes 的包管理器。

```shell
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Snap

[https://snapcraft.io/](https://snapcraft.io/)

Snap 是由 Canonical（Ubuntu 母公司） 开发的 Linux 应用打包与分发格式，旨在提供跨发行版的应用安装方式，适用于桌面和服务器端应用。Snap 应用使用 Snapd 运行时管理，并从 Snap Store 获取软件，Ubuntu及其衍生版本默认使用。

由于 Canonical 的强力推广，Snap Store 商业软件支持更广泛，尤其是服务器和 IoT 领域，但是较为封闭。开源社区推动的 Flatpak 越来越受欢迎，其去中心化模式（允许第三方存储库）更符合 Linux 的自由精神。

### Flatpak

[https://flatpak.org/](https://flatpak.org/)  
[https://flathub.org/](https://flathub.org/)

Flatpak 是一种用于 Linux 的软件包管理和应用沙盒化技术，旨在提供跨发行版的应用运行环境，使开发者可以一次打包应用，并在不同的 Linux 发行版上运行，而无需针对每个发行版单独打包。Fedora、EndeavourOS、Linux Mint、elementary OS默认预装。

```shell
# Debian/Ubuntu
sudo apt install flatpak
# Arch Linux
sudo pacman -S flatpak
# openSUSE
sudo zypper install flatpak
```

### AppImage

[https://appimage.org/](https://appimage.org/)

AppImage 是一种便携式 Linux 应用打包格式，不依赖于特定的发行版或包管理器，类似于 Windows 的 .exe 或 macOS 的 .dmg，用户只需下载 AppImage 文件即可运行软件，无需安装，无官方应用商店。

### Scoop

Windows 第三方包管理工具。

[https://scoop.sh/](https://scoop.sh/)

标准安装（安装到当前用户目录）

```shell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

# 安装系统级软件（需要管理员权限）
sudo scoop install -g <app>
# 更新所有全局包
sudo scoop update * -g
```

高级安装

```shell
# advanced installation
irm get.scoop.sh -outfile 'install.ps1'
# custom directory
.\install.ps1 -ScoopDir 'D:\ProgramData\Scoop' -ScoopGlobalDir 'D:\GlobalScoopApps' -NoProxy
```

```shell
# 更换scoop国内源
scoop config SCOOP_REPO "https://gitee.com/scoop-installer/scoop"
# 拉取新库地址
scoop update

scoop bucket list

# 更换南京大学源
scoop bucket rm main
scoop bucket add main https://mirrors.nju.edu.cn/git/scoop-main
scoop bucket rm extras
scoop bucket add extras https://mirrors.nju.edu.cn/git/scoop-extras
```

### Alpine

[https://www.alpinelinux.org](https://www.alpinelinux.org/)

```shell
# 阿里云
sudo sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 中国科学技术大学
sudo sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
# 清华大学
sudo sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
```

### Debian

稳定、性能好，缺点是包版本旧。

```shell
sudo sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list
sudo sed -i 's/ftp.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list
```

### Ubuntu

```shell
sudo sed -i 's/http:\/\/archive.ubuntu.com/http:\/\/mirrors.aliyuncs.com/g' /etc/apt/sources.list
```

### HomeBrew

[https://brew.sh/zh-cn/](https://brew.sh/zh-cn/)

阿里云镜像限速 200k，不推荐

```shell
# 安装brew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# homebrew-中国科学技术大学
export PATH="/usr/local/sbin:$PATH"
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles"
export HOMEBREW_API_DOMAIN="https://mirrors.ustc.edu.cn/homebrew-bottles/api"

# homebrew-清华大学
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"
export HOMEBREW_API_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles/api"
export HOMEBREW_PIP_INDEX_URL="https://pypi.tuna.tsinghua.edu.cn/simple"
```

### Golang (gvm)

[https://go.dev](https://go.dev/)

[https://github.com/moovweb/gvm](https://github.com/moovweb/gvm)

中科大、清华未提供

```shell
# 安装gvm
sudo apt install bison
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

# golang-阿里云
go env -w GOPROXY="https://mirrors.aliyun.com/goproxy/,direct"

# 或者编辑.bashrc/.zshrc
[[ -s "$HOME/.gvm/scripts/gvm" ]] && source "$HOME/.gvm/scripts/gvm"
export GO111MODULE=on
export GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
# 或者七牛云
export GOPROXY=https://goproxy.cn,direct
```

私有仓库

```shell
go env -w GOPRIVATE="codeup.aliyun.com"
go env -w GONOPROXY="codeup.aliyun.com"

# .netrc
machine codeup.aliyun.com login xxx password xxx
```

### NodeJS npm (nvm)

[https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/)

[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

阿里巴巴广泛应用 nodejs，镜像稳定可靠，推荐优先使用。清华未提供。

```shell
# Mac安装nodejs nvm，还需要配置 NVM_DIR
brew install nvm

# 阿里云
npm set registry http://registry.npmmirror.com/

# 或者手动编辑
vi ~/.npmrc
# 阿里云
registry=http://registry.npmmirror.com/
# 中国科学技术大学（仅为https://registry.npmjs.org/的反向代理）
registry=https://npmreg.proxy.ustclug.org/

# 查看生效镜像
npm get registry
```

### PHP

[https://www.php.net](https://www.php.net/)

homebrew 等包管理器会升级到最新版本，建议使用 alpine 编写 Dockerfile 构建镜像，参考实例 [Docker php](./docker_php)。

```shell
# 安装PHP
brew install php
apt install php

# brew第三方库
brew tap shivammathur/php
brew install shivammathur/php/php@7.4
brew tap shivammathur/extensions
brew install shivammathur/extensions/redis@7.4

# debian/ubuntu第三方库
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install php7.4
sudo apt install php7.4-redis

# 安装php版本管理工具
# 通用方式
# curl -L https://github.com/phpbrew/phpbrew/releases/latest/download/phpbrew.phar -O phpbrew.phar && chmod +x phpbrew.phar
# sudo mv phpbrew.phar /usr/local/bin/phpbrew
# MacOS
# brew install phpbrew
```

[https://github.com/phpbrew/phpbrew](https://github.com/phpbrew/phpbrew) 安装多版本容易报错，不推荐使用。

### PHP Composer

[https://getcomposer.org](https://getcomposer.org/)

中科大、清华未提供

```shell
# 下载composer命令
sudo wget https://getcomposer.org/composer-stable.phar -O /usr/local/bin/composer && sudo chmod +x /usr/local/bin/composer
# 或者
sudo curl -sS https://mirrors.aliyun.com/composer/composer.phar -o /usr/local/bin/composer && sudo chmod +x /usr/local/bin/composer

# 阿里云
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
# 取消
composer config -g --unset repos.packagist
```

### Python

[https://www.python.org](https://www.python.org/)

推荐使用pyenv控制版本。

```shell
# Linux/Unix
curl -fsSL https://pyenv.run | bash
# Mac
brew install pyenv

# 安装、卸载
pyenv install --list
pyenv install 3.12
pyenv uninstall 3.10.13
# 全局默认版本
pyenv global 3.7.17
# 当前会话版本
pyenv shell 3.12.10
# 设置项目目录局部版本（在目录下创建 .python-version）
pyenv local 3.12.10
```

### Python pip

[https://pypi.org/](https://pypi.org/)

> Python 库的官方仓库 pypi 允许开发者自由上传软件包，这会导致某些攻击者利用这点构造恶意包进行供应链攻击，在用户安装包或者引入包时触发恶意行为。目前国内镜像源与官方镜像源往往并不是完全一致，国内源普遍采用的是增量更新机制，也就是官方删除的恶意包国内不会同步删除。而这一部分恶意包可能会在很长一段时间内对国内用户造成影响。

阿里云非增量更新，推荐使用。

```shell
# 阿里云
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/

# 临时使用
# 中国科学技术大学
pip install -i https://mirrors.ustc.edu.cn/pypi/web/simple package
# 清华大学
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple package

# 或者手动编辑
vi ~/.pip/pip.conf

[global]
index-url = https://mirrors.aliyun.com/pypi/simple/

[install]
trusted-host=mirrors.aliyun.com
```

### Java maven

中科大、清华未提供

```shell
vi ~/.m2/settings.xml

<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

### Ruby rubygems (rvm)

```shell
# 查找默认源
gem sources -l

# 阿里云
gem sources -a https://mirrors.aliyun.com/rubygems/ --remove https://rubygems.org/

# 中国科学技术大学
gem sources --add https://mirrors.ustc.edu.cn/rubygems/ --remove https://rubygems.org/

# 清华大学
gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
```

### Rust

[https://www.rust-lang.org/zh-CN/](https://www.rust-lang.org/zh-CN/)

阿里云未提供

```shell
vi ~/.cargo/config

# 中国科学技术大学
[source.crates-io]
replace-with = 'ustc'
[source.ustc]
registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"

# 清华大学
[source.crates-io]
replace-with = 'tuna'
[source.tuna]
registry = "sparse+https://mirrors.tuna.tsinghua.edu.cn/crates.io-index/"

# 更新
cargo +nightly -Z sparse-registry update
```
