# 使用Strongswan创建IKEv2类型的VPN

之前我们介绍了如何使用[https://safewoo.com](https://safewoo.com)一键式创建IKEv2 VPN. 
如果想亲手搭建一个 IKEv2 VPN ? 我们将带您一步步使用 Strongswan 实现。我们将从服务器端配置开始，
详细讲解每个步骤，并覆盖 Windows、macOS、iOS、Android 等主流操作系统上的客户端配置。
通过本教程，您将掌握 IKEv2 VPN 的实践技能。

我们的目标是:

1. 搭建一个支持IKEv2的VPN服务器
2. 支持基于用户名和密码的认证
3. 支持iOS, macOS, Windows的内置VPN客户端, 以及Android的Strongswan客户端 

由于本站主要介绍安全上网相关的知识,所以我们并不会介绍如何配置Linux的Strongswan连接.

**注意**: 本文将会涉及一些网络,加密,操作系统等方面的知识,不适合小白用户. 
如果你是小白用户,请使用 **安全屋** [https://safewoo.com](https://safewoo.com)一键创建IKEv2类型的VPN.

## Strongswan介绍

Strongswan是一个开源的IPsec实现，支持IKEv1和IKEv2协议。它是Linux系统上最流行的VPN解决方案之一。
官网地址：[https://www.strongswan.org/](https://www.strongswan.org/).

它实现了 IKEv1 和 IKEv2 协议，支持多种认证方法（如 RSA、PSK、EAP），并提供灵活的配置选项. 

它的核心功能包括:

- IKE 协商: 用于建立安全关联，交换密钥
- IPsec 数据包加密: 对传输的数据进行加密和认证，确保数据的机密性、完整性和真实性
- NAT 穿透: 支持 NAT 环境下的 VPN 连接
- EAP 认证: 支持多种 EAP 认证方法，如 EAP-TLS、PEAP 等, 支持用户名和密码认证

## 所需资源

- 一台 Debian 12/Ubuntu 22.04 服务器
  - 具有公网IPv4地址(可以是NAT转发)
  - 账户具有sudo权限
- 一个域名，用于配置证书

***为什么需要域名？*** 为了提升 VPN 连接的安全性，IKEv2 协议通常采用数字证书进行身份验证。如果使用自签名证书，
客户端需要手动导入并信任证书，这对于普通用户来说操作较为复杂。而使用域名申请的 SSL 证书，
则可以借助受信任的 CA 机构（如 Let's Encrypt）颁发，客户端操作系统通常会默认信任这些 CA，
从而省去繁琐的证书配置过程。

## 服务端配置

### 申请SSL证书

推荐使用[acme.sh](https://github.com/acmesh-official/acme.sh)工具申请Let's Encrypt证书，acme.sh 是一个开源的 Shell 脚本，支持自动化申请和更新
Let's Encrypt 证书。具体使用方法请参考官方文档

如果你没有域名,或者无法申请SSL证书, 
**安全屋** [https://safewoo.com](https://safewoo.com)可以一键创建IKEv2类型的VPN,并提供证书和解析.
自动续签证书,无需担心证书过期问题.



### 安装依赖软件

```bash
sudo apt update
sudo apt install -yqq charon-systemd strongswan-swanctl strongswan-pki \
    libcharon-extra-plugins libstrongswan-extra-plugins \
    openssl curl
sudo apt install -yqq iptables-persistent
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y iptables-persistent
```


### 配置系统参数

1. 配置sysctl

```bash
sudo bash -c 'cat > /etc/sysctl.d/99-strongswan.conf << EOF
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
net.ipv4.conf.all.send_redirects=0
net.ipv4.conf.default.send_redirects=0
net.ipv4.conf.all.send_redirects=0
EOF'
sudo sysctl --system
```

2. 配置iptables流量转发

大型云服务商(AWS,GCP,Azure)不会直接将公网IP地址绑定到网卡上, 小型的VPS商家可能会直接将公网IP地址绑定到网卡上.

如果你的服务器没有直接绑定公网IP地址, 你需要配置NAT转发, 以便VPN服务器可以将流量转发到公网.

```bash
NETWORK_V4=192.168.78.0/24  # VPN子网
NIF=$(ip -o -4 route show to default | awk '{print $5}') # 获取默认网卡
# ipv4 rules
iptables -t nat -A POSTROUTING -s $NETWORK_V4 -o $NIF -m policy --dir out --pol ipsec -j ACCEPT # 允许IPsec流量
iptables -t nat -A POSTROUTING -s $NETWORK_V4 -o $NIF -j MASQUERADE # NAT地址转换, 允许VPN将流量转发到公网
```

### 安装和配置Strongswan

- 假设你的域名是 `vpn.example.com`
- 已有SSL证书 `vpn.example.com.crt` 证书私钥 `vpn.example.com.key`, CA证书 `ca.crt`

将上述证书复制到下面的对应路径

- `/etc/swanctl/x509ca/ca.crt`
- `/etc/swanctl/x509/vpn.example.com.crt`
- `/etc/swanctl/private/vpn.example.com.key`

编辑配置文件

```bash 
sudo bash -c "cat > /etc/swanctl/conf.d/safewoo-eap.conf << EOF
connections {
    conn_safewoo_eap {
        pools = pool_safewoo_eap_v4
        reauth_time = 0
        send_certreq = yes 
        send_cert = always
        version = 2
        proposals = aes128-aes256-sha256-modp1024-modp2048-curve25519-ecp256 # 兼容 Windwos, macOS, iOS, Android
        local {
            auth = pubkey
            certs = ca.crt # CA证书
            id = vpn.example.com # 与上文证书名一致
        }

        remote {
            auth = eap-mschapv2
        }

        children {
            dt {
                local_ts = 0.0.0.0/0
            }
        }
    }
}

pools {
    pool_safewoo_eap_v4 {
        addrs = 192.168.78.0/24
        dns = 1.1.1.1, 8.8.8.8
    }
}

secrets {
    eap-safewoo {
        id = safewoo  # 账户名
        secret = "password" # 密码
    }
}
EOF"
```

载入配置和证书

```bash
sudo swanctl -q
```

配置即可生效

## 客户端配置

我们成功配置了了IKEv2协议的VPN, 使用EAP-MSCHAPv2认证方式, 可以通过直接输入账户名和密码连接VPN. 
主流的设备都支持这种方式. 我们在配置文件里直接指定了账户/密码 `safewoo`/`password`. 
**请替换为更加安全的账户名和密码**.

具体各系统的连接方式请参考之前的博客的链接: [连接你的VPN](如何自建VPN#连接你的VPN)



