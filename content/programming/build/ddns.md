---
title: "DDNS"
type: "docs"
weight: 3
---

[https://github.com/jeessy2/ddns-go](https://github.com/jeessy2/ddns-go)

建议直接安装在主机上，通过docker安装可能获取不到公网网卡。

安装到主机

```shell
brew install ddns-go
```

通过docker安装

```shell
# 挂载主机目录, 使用docker host模式。可把 /opt/ddns-go 替换为你主机任意目录, 配置文件为隐藏文件
docker run -d --name ddns-go --restart=always --net=host -v /opt/ddns-go:/root jeessy/ddns-go

# 不使用docker host模式
docker run -d --name ddns-go --restart=always -p 9876:9876 -v /opt/ddns-go:/root jeessy/ddns-go
```

控制台地址

[http://127.0.0.1:9876/](http://127.0.0.1:9876/)
