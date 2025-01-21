
import Image from 'next/image'

# 如何自建VPN 

- Author: Safewoo
- Date: 2024-12-04

## VPN协议

大家在进行安全上网的时候都会使用VPN。
VPN全称是Virtual Private Network，中文翻译为虚拟专用网络。它是一种在设备之间建立安全通道的方法。
实际上，很多代理代理协议，代理客户端也被称为VPN。例如 Shadowsocks、V2Ray, Clash 等。
本文专注于虚拟专用网络（VPN）。

如果你想突破防火墙，访问互联网，或者担心隐私泄露，不想在学校或咖啡店的公共网络上暴露流量，你有几种选择。
下载已经上架的商业VPN，或者如果你有相应的技术，可以自己搭建VPN。

在选择VPN时，许多用户优先考虑其突破封锁的能力。毕竟使用VPN的主要目的是自由访问互联网，躲避审查，保护隐私。
但是易于使用也是一个非常重要的因素。Windows 和 Android 用户通常有很多选择，但是 iOS App Store 
对 VPN 应用有更严格的规定。在中国大陆，是无法从 App Store 下载流行的VPN和代理工具。

购买商业VPN应用看似是最简单的选择，但是这种选择充满了潜在的风险。许多廉价甚至免费的VPN实际上是
政府或黑产设立的蜜罐。不小心使用这些产品无异于在他们面前裸奔。

自己搭建VPN是许多人选择的方式，因为这样可以更好地控制自己的隐私和安全。本文将探讨用于自建VPN的最常见的VPN协议。


## 常见的VPN协议

我们看看有哪些常见的VPN协议。

#### PPTP

- **不推荐**, 较早的一种协议，安全性较差，它的连接甚至没有加密，现在被防火墙秒封。

#### L2TP/IPSec

- 安全性较好的一种协议，
- 大多数设备都支持。 例如 Windows, macOS, iOS, Android等系统的内置VPN客户端都可以直接设置。
- 由于它不支持Mobike，当你在Wi-Fi和移动网络之间切换时，可能会导致连接中断。

#### OpenVPN

- 安全性高，特征明显，秒封。
- 大多数系统并不内置OpenVPN客户端，需要安装第独立客户端。

#### IKEv2

- 安全性高，速度快。
- 大多数设备都支持，例如 Windows, macOS, iOS, Android等系统的内置VPN客户端都可以直接设置。
- 由于它支持Mobike，当你在Wi-Fi和移动网络之间切换时，连接不会中断。

#### WireGuard

- 较新的协议，安全性高，速度快。
- 由于它是一个新的协议，不是所有设备都支持。需要安装第三方客户端。


根据比较，IKEv2在安全性和速度之间取得了很好的平衡，使其成为自建VPN比较好的选择。它有广泛设备兼容性和无缝的移动
网络切换。虽然IKEv2是一个强大的协议，但是设置IKEv2 VPN非常麻烦。需要深入了解网络协议和配置，并且需要分别配置
不同客户端（Windows, macOS, iOS, Android）需要花费大量时间。

## 使用Safewoo.com快速搭建VPN 

### 准备工作

你只需要一个VPS（Virtual Private Server）来搭建自己的VPN。你可以从云服务商购买VPS，例如AWS, Google Cloud, 
Banwagon, Vultr等。这里以Google Cloud为例。

假设你有一个VPS，IP地址为`xx.xx.xx.xx`, 用户名为`sam`，已配置通过SSH密钥认证。

### 一键搭建VPN

访问 [safewoo.com](https://safewoo.com) , 按照以下步骤操作：

1. 登录Safewoo.com，可以直接使用Google或者GitHub账号登录。
2. VPN Type选择 `IKEv2 - EAP (Username and password)`。它比较容易使用。
3. 输入VPS的账户，地址。 选择 `SSH Private Key` 选项, 贴入密钥内容，如果密钥设置了口令，需要输入口令。
4. 点击 `Create VPN` 等待设置完成。

设置完成后，查看下方显示的安装指南。你会得到 ***域名***, ***VPN账户*** , 和 ***VPN密码***。你可以使用这个账户在任何支持IKEv2 VPN的设备上连接你的VPN服务器。

### 连接你的VPN

#### 使用iOS连接你的VPN 

1. 打开设置->通用->VPN & 设备管理->添加VPN配置
2. 选择***IKEv2***作为VPN类型
3. 输入以下信息：
   - 描述：VPN的名称，例如，My VPN
   - 服务器：之前提供的 ***域名***。
   - 远程ID：之前提供的 ***域名***。
   - 本地ID：留空
   - 用户认证：选择***用户名***
   - 用户名：之前提供的 ***VPN账户***。
   - 密码：之前提供的 ***VPN密码***。
   <!-- <Image src="/blog-res/how-to-build-your-own-vpn/add-vpn-ios.png" width={293} height={663} alt="Add iOS VPN"/> -->
   <img src="assets/add-vpn-ios.png" width="300" alt="Add iOS VPN" />
4. 点击**保存**，然后点击**连接**。

#### 使用macOS连接你的VPN

1. 打开系统偏好**设置**->**网络**
2. 点击左下角的加号，**选择VPN**->**IKEv2**
3. 输入以下信息：
   - 显示名称：VPN的名称，例如，My VPN
   - 服务器地址：之前提供的 ***域名***。
   - 远程ID：之前提供的 ***域名***。
   - 本地ID：留空
   - 用户认证：选择**用户名**
   - 用户名：之前提供的 ***VPN账户***。
   - 密码：之前提供的 ***VPN密码***。
   <!-- <Image src="/blog-res/how-to-build-your-own-vpn/add-vpn-macos.png" width={621} height={417} alt="Add macOS VPN"/> -->
   <img src="assets/add-vpn-macos.png" width="480" alt="Add macOS VPN" />
4. 点击**连接**。

#### 使用Windows连接你的VPN

*注意* 以下步骤适用于Windows 11。其他版本的Windows可能有所不同。
1. 打开Windows设置应用。
2. 点击**网络和Internet**->**VPN**->**添加VPN**
3. 输入以下信息：
   - VPN提供商：**Windows（内置）**
   - 连接名称：VPN的名称，例如，My VPN
   - 服务器名称或地址：之前提供的 ***域名***。
   - VPN类型：**IKEv2**
   - 登录信息类型：选择**用户名和密码**
   - 用户名：之前提供的 ***VPN账户***。
   - 密码：之前提供的 ***VPN密码***。
   <!-- <Image src="/blog-res/how-to-build-your-own-vpn/add-vpn-win.png" className="bordered" width={292} height={592} alt="Add Windows VPN" /> -->
   <img src="assets/add-vpn-win.png" width="300" alt="Add Windows VPN" />
5. 点击**保存**，然后点击**连接**。

#### 使用Android连接你的VPN

##### 使用手机内置VPN客户端

*注意* 由于不同Android设备的设置可能有所不同，请参考手机用户手册以获取详细说明。

1. 在手机设置中找到VPN设置。
2. 添加一个新的VPN连接。
3. 选择**IKEv2/IPSec EAP_MSCHAPV2**或**IKEv2 EAP**或**IKEv2 Username/Password**作为VPN类型。
4. 输入你的 ***域名***, ***用户名***, 和 ***密码***。
5. 保存配置并连接VPN。

##### 使用StrongSwan应用

如果你的Android设备不支持IKEv2 EAP，你可以从Google Play商店安装StrongSwan应用。

1. 从Google Play商店安装StrongSwan应用。
2. 打开应用，点击**添加VPN配置**。
3. 输入以下信息：
   - 服务器：之前提供的 ***域名***。
   - VPN类型：选择**IKEv2 EAP (用户名/密码)**
   - 用户名：之前提供的 ***用户名***。
   - 密码：之前提供的 ***密码***。
   - CA证书：点击**自动选择**
   <!-- <Image src="/assets/how-to-build-your-own-vpn/add-vpn-android.jpg" width={270} height={600} alt="Add Android VPN"/> -->
   <img src="assets/add-vpn-android.jpg" width="300" alt="Add Android VPN" />
4. 保存配置并连接VPN。
