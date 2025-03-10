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
Host myserver
HostName www.server.com
User lynn
IdentityFile ~/.ssh/id_rsa_xxx

# by proxy
Host server02
HostName 172.16.12.12
User lynn
ProxyJump myjump

# github
Host github.com
HostName github.com
User git
UseKeychain yes
AddKeysToAgent yes
IdentityFile ~/.ssh/id_rsa_xxx
```

```shell
# Mac添加密钥到ssh-agent并存入钥匙串（避免重复输入密码）
ssh-add --apple-use-keychain ~/.ssh/id_rsa_xxx
```

## 问题

### No matching host key type found. Their offer: ssh-rsa

原因：openssh 觉得 ssh-rsa 加密方式不安全，直接从 8.8 开始默认不允许登陆。

长期方案

~/.ssh/config

```text
Host myjump
Port 2022
HostName jumpserver.com
HostKeyAlgorithms +ssh-rsa
PubkeyAcceptedKeyTypes +ssh-rsa
User test
```

临时方案

sftp -P 2022 -oHostKeyAlgorithms=+ssh-rsa xxx@jumpserver.com
