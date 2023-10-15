---
title: "SSH"
type: "docs"
weight: 3
---

## 配置文件

用于 ssh、git。

~/.ssh/config

```text
# server
Host zjk
HostName www.server.com
User root
IdentityFile ~/.ssh/id_rsa_xxx

# github
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_xxx
```

## 问题

### No matching host key type found. Their offer: ssh-rsa

原因：openssh 觉得 ssh-rsa 加密方式不安全，直接从 8.8 开始默认不允许登陆。

长期方案

~/.ssh/config

```text
Host jumpserver.com
PubkeyAcceptedKeyTypes +ssh-rsa
HostKeyAlgorithms +ssh-rsa
```

临时方案

sftp -P 22 -oHostKeyAlgorithms=+ssh-rsa xxx@jumpserver.com
