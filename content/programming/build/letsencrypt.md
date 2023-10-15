---
title: "LetsEncrypt"
type: "docs"
weight: 3
---

[https://letsencrypt.org/](https://letsencrypt.org/)

Let's Encrypt 是免费、开放和自动化的证书颁发机构，提供免费的HTTPS证书，由非盈利组织互联网安全研究小组（ISRG）运营。

推荐使用certbot自动化安装，设置定时任务定期更新。

## Certbot

[https://certbot.eff.org/](https://certbot.eff.org/)

如果使用docker容器，需要容器内能够访问nginx配置目录。

### CentOS 8

```shell
#安装snap
sudo yum install snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap

#更新snap
sudo snap install core && sudo snap refresh core

#安装certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

#获取证书
sudo certbot --nginx
#更新证书，可添加定时任务自动更新，/etc/crontab/
sudo certbot renew
```

### alpine

```shell
apk add certbot certbot-nginx
```
