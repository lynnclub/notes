---
title: "cloudflared tunnel"
type: "docs"
weight: 3
---

cloudflared提供免费的DNS、CDN服务，还提供cloudflared tunnel用于内网穿透。

```bash
# 安装客户端
cd /usr/local/bin/
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64 -o cloudflared
chmod +x cloudflared

# 登录
cloudflared tunnel login

# 创建隧道
cloudflared tunnel create <隧道名称>
# 映射本地服务
cloudflared tunnel route dns <隧道名称> <子域名>.<你的域名>
# 临时启动
cloudflared tunnel run <隧道名称>
# 开机自启
sudo systemctl daemon-reload
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

/etc/systemd/system/cloudflared.service

```text
[Unit]
Description=Cloudflare Tunnel
After=network.target

[Service]
Type=simple
User=<用户名>
ExecStart=/usr/local/bin/cloudflared tunnel run <隧道名称>
Restart=always  # 进程崩溃后自动重启
RestartSec=5

[Install]
WantedBy=multi-user.target
```
