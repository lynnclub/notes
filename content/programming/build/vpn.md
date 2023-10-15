---
title: "VPN"
type: "docs"
weight: 10
draft: true
---

## IPSec

需要开放500、4500端口，运营商宽带（中国移动）默认封禁，无法使用。

[https://github.com/hwdsl2/docker-ipsec-vpn-server](https://github.com/hwdsl2/docker-ipsec-vpn-server)

```shell
# 创建配置
vi vpn.env
# 配置，IPsec/L2TP、IPsec/XAuth ("Cisco IPsec")
VPN_IPSEC_PSK=xxx
VPN_USER=xxx
VPN_PASSWORD=your_vpn_password
# 只启动IKV2，基于证书，无需密码
VPN_IKEV2_ONLY=yes
```

```shell
# 启动
docker run \
    --name ipsec-vpn-server \
    --restart=always \
    --env-file=vpn.env \
    -v ikev2-vpn-data:/etc/ipsec.d \
    -v /lib/modules:/lib/modules:ro \
    -p 500:500/udp \
    -p 4500:4500/udp \
    -d --privileged \
    hwdsl2/ipsec-vpn-server

# 查看配置
docker logs ipsec-vpn-server

# 复制文件
# 用户名、密码
docker cp ipsec-vpn-server:/etc/ipsec.d/vpn-gen.env ./
# Windows & Linux证书
docker cp ipsec-vpn-server:/etc/ipsec.d/vpnclient.p12 ./
# Android证书
docker cp ipsec-vpn-server:/etc/ipsec.d/vpnclient.sswan ./
# iOS & macOS证书
docker cp ipsec-vpn-server:/etc/ipsec.d/vpnclient.mobileconfig ./
```

[配置 IPsec/L2TP VPN客户端](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-zh.md)

需要修改注册表，执行下述reg文件后重启。

Fix_VPN_AllowL2TPWeakCrypto_Reboot_Required.reg

```text
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RasMan\Parameters]
"AllowL2TPWeakCrypto"=dword:00000001

```

Fix_VPN_Error_809_Allow_IPsec_Reboot_Required.reg

```text
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\RasMan\Parameters]
"ProhibitIpSec"=dword:00000000

```

Fix_VPN_Error_809_Windows_Vista_7_8_10_Reboot_Required.reg

```text
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\PolicyAgent]
"AssumeUDPEncapsulationContextOnSendRule"=dword:00000002

```

## OpenVPN

[https://github.com/kylemanna/docker-openvpn](https://github.com/kylemanna/docker-openvpn)

docker-compose.yml

```yaml
version: '2'
services:
  openvpn:
    cap_add:
     - NET_ADMIN
    image: kylemanna/openvpn
    container_name: openvpn
    ports:
     - "1194:1194/udp"
    restart: always
    volumes:
     - ./openvpn-data/conf:/etc/openvpn
```

```shell
# 初始化配置文件和证书
docker-compose run --rm openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
docker-compose run --rm openvpn ovpn_initpki

# 启动
docker compose up -d openvpn

# 生成客户端证书
docker compose run --rm openvpn easyrsa build-client-full $CLIENTNAME
# 导出
docker compose run --rm openvpn ovpn_getclient $CLIENTNAME > $CLIENTNAME.ovpn
# 移除客户
docker compose run --rm openvpn ovpn_revokeclient $CLIENTNAME remove
```
