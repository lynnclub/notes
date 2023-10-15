---
title: "Docker"
type: "docs"
weight: 3
---

## Docker Desktop

Linux 可以只安装 Docker Engine，在 Windows 10 和 macOS 操作系统上必须安装包含图形界面的 Docker Desktop。

### Windows 10+

[https://docs.docker.com/desktop/install/windows-install/](https://docs.docker.com/desktop/install/windows-install/)

Windows 可以使用 WSL 虚拟机作为 docker 底层运行环境。

### Mac

[https://docs.docker.com/desktop/install/mac-install/](https://docs.docker.com/desktop/install/mac-install/)

```shell
brew install –cask docker
```

### Linux

不需要图形界面可以不装，仅需 Docker Engine。

[https://docs.docker.com/desktop/install/linux-install/](https://docs.docker.com/desktop/install/linux-install/)

## Docker Engine

Docker Desktop 主要是图形界面，Windows 和 Mac 系统必须安装 Desktop，Linux 系统可以只安装 Docker Engine。

### CentOS

```shell
# docker仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

systemctl enable docker.service
systemctl start docker.service
```

### Debian

```shell
# 添加官方GPG密钥
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 设置仓库
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 常用容器

```shell
docker run --restart=always --name=postgres --env=POSTGRES_PASSWORD=19950821 --volume=D:\run\postgresql:/var/lib/postgresql/data -p 5432:5432 -d postgres:latest

docker run --restart=always --name=redis --volume=D:\run\redis:/data -p 6379:6379 -d redis:latest

docker network create somenetwork
docker run --restart=always --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -d elasticsearch:tag
```
