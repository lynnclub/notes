---
title: "镜像"
type: "docs"
weight: 2
---

### 镜像源

南科大，北清华，遇事不决阿里云。

阿里云镜像，全国可用 [https://developer.aliyun.com/mirror/](https://developer.aliyun.com/mirror/)

清华大学镜像，北方推荐，教育网 [https://mirrors.tuna.tsinghua.edu.cn/](https://mirrors.tuna.tsinghua.edu.cn/)

中国科学技术大学镜像，南方推荐，教育网 [https://mirrors.ustc.edu.cn/](https://mirrors.ustc.edu.cn/)

注意：阿里云 ECS 用户，可以将 `https://mirrors.aliyun.com/`
替换成 `http://mirrors.cloud.aliyuncs.com/`。

### Docker

[https://www.docker.com](https://www.docker.com/)

```json
// 阿里云个人账户镜像
"registry-mirrors": [
    "https://2kqxkocb.mirror.aliyuncs.com"
]
```

### Snap

Linux 系统通用的应用格式包，Ubuntu 背后的公司 Canonical 主导，支持 Debian、Arch Linux、Fedora、Kaili Linux、openSUSE、Red Hat 等。

[https://snapcraft.io/](https://snapcraft.io/)

### Scoop

Linux 风格的第三方开源 Windows 包管理工具。

[https://scoop.sh/](https://scoop.sh/)

### Alpine

[https://www.alpinelinux.org](https://www.alpinelinux.org/)

```shell
# 阿里云
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 中国科学技术大学
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories

# 清华大学
sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
```

### Debian

稳定、性能好，缺点是包版本旧。

```shell
sed -i 's/deb.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list
sed -i 's/security.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list
sed -i 's/ftp.debian.org/mirrors.aliyun.com/g' /etc/apt/sources.list
```

### Ubuntu

```shell
sed -i 's/http:\/\/archive.ubuntu.com/http:\/\/mirrors.aliyuncs.com/g' /etc/apt/sources.list
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
[[ -s "$HOME/.gvm/scripts/gvm" ]] && source "$HOME/.gvm/scripts/gvm"
export GO111MODULE=on
export GOPROXY=https://mirrors.aliyun.com/goproxy/,direct
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
# nodejs nvm，还需要配置 NVM_DIR
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

### PHP（phpbrew）

[https://www.php.net](https://www.php.net/)

[https://github.com/phpbrew/phpbrew](https://github.com/phpbrew/phpbrew) 安装多版本报错，不推荐

PHP 多版本，homebrew 等包管理器会升级到最新版本，phpbrew 报错，最大的问题还是装扩展很麻烦，建议使用 alpine 编写 Dockerfile 构建镜像，参考实例 [Docker php](./docker_php)。

```shell
# 安装PHP
brew install php
apt install php

# 第三方库
brew tap shivammathur/php
brew install shivammathur/php/php@7.4
brew tap shivammathur/extensions
brew install shivammathur/extensions/redis@7.4

# 安装php版本管理工具
# 通用方式
curl -L -O https://github.com/phpbrew/phpbrew/releases/latest/download/phpbrew.phar
chmod +x phpbrew.phar
sudo mv phpbrew.phar /usr/local/bin/phpbrew
# MacOS
brew install phpbrew
```

### PHP Composer

[https://getcomposer.org](https://getcomposer.org/)

中科大、清华未提供

```shell
# 下载composer命令
curl -L -O https://mirrors.aliyun.com/composer/composer.phar
chmod +x composer.phar && mv composer.phar /usr/local/bin/composer

# 阿里云
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
# 取消
composer config -g --unset repos.packagist
```

### Python pip

[https://www.python.org](https://www.python.org/)

> Python 库的官方仓库 pypi 允许开发者自由上传软件包，这会导致某些攻击者利用这点构造恶意包进行供应链攻击，在用户安装包或者引入包时触发恶意行为。目前国内镜像源与官方镜像源往往并不是完全一致，国内源普遍采用的是增量更新机制，也就是官方删除的恶意包国内不会同步删除。而这一部分恶意包可能会在很长一段时间内对国内用户造成影响。

阿里云非增量更新，推荐使用。

```shell
#阿里云
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
