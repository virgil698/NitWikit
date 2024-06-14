---
title: 如何抵御网络攻击
sidebar_position: 3
---

# 如何抵御网络攻击

随着你的 Minecraft 服务器人数和宣传越来越多，你的服务器越有可能收到其他“友商”或者某些不怀好意的玩家攻击。

别害怕，大多数网络攻击没有那么致命，可能只是会引起玩家高ping掉线、后台操作卡顿等。

## 分类

我们在这里提到几种常见的 Minecraft 服务器容易遭受的攻击类型。

### 应用层（高级称呼是 L7 ）

在服务器上运行并绑定了指定地址和端口的应用程序可以在这一层接受连接，

应用层的攻击往往意味着针对某个应用程序发起的攻击，应用可以接受到该恶意的连接，

通常是利用应用中未考虑的意外情况来导致应用占用更多的计算机资源来尝试处理遇到的情况，

也可以通过大量请求使带宽不堪重负，使服务器难以接受新连接。

#### 假人攻击

简单来说假人攻击一般是通过模拟客户端协议进入服务器发送进入服务器的数据包，伪造有玩家接入服务器造成的攻击。

这些假人大多数是不动的、名字高度相似或者非常随机的，随着假人的快速加入和退出游戏，

将导致服务器需要加载更多的数据并发送大量的数据包等导致服务器卡顿。

#### MOTD (状态请求) 攻击

简单来说就是向服务器请求状态 (也就是 Ping) 服务器，每次 Ping 服务器时，服务器将返回一个 MOTD ，由于 MOTD 中包含图片和文字信息，

大量请求会导致快速消耗服务器带宽，而使服务器难以接受新的连接。

MC 后端服务器一般是不会对 Ping 进行过滤和记录的，这会导致 Ping 的过程后台不会记录 log ，难以察觉。

对于 Velocity / BungeeCord 等反向代理客户端，默认 Ping 服务器的行为是会被记录的，类似于：

```
[/127.0.0.1:61647] <-> InitialHandler has pinged
```

:::info

可以通过调整设置 `log_pings` (BungeeCord) 或 `show-ping-requests` (Velocity) 来停止反向代理在控制台打印状态请求日志。

:::

#### 其他插件

如果你使用了 Plan 、 Dynmap 等插件，这些插件会在某个端口开启网站。请注意这些端口如果被不怀好意的人知道，

则可能会导致这些 HTTP 端口被针对或遭受攻击。

#### Minecraft 漏洞攻击

通过利用 MC 游戏本身的漏洞，向服务器发送(可能是大量的)不符合正常逻辑的数据包，会造成服务器卡顿甚至**崩溃**

### 网络层

网络层攻击是 DDoS 攻击的一种形式，它针对于网络基础架构进行攻击。最常见的网络层攻击是IP地址欺骗，

攻击者可以伪造IP地址并向目标服务器发送大量数据包，以消耗目标服务器的网络带宽和系统资源。

Minecraft JAVA 服务端采用的是 TCP 作为通信协议，所以说你就会遭受到例如 TCP Flood( TCP 洪水攻击)

防御这种类型的攻击唯一办法就是增大宽带，没有什么别的好办法。

## 解决方案

### 使用 Velocity / BungeeCord

不要试图使用单独使用任何后端服务器（如 Spigot / Paper / Purpur 等）抵御大规模应用层攻击，

后端服务器处理连接的速度较慢，这将会导致消耗比代理更多的计算机资源，一旦攻击规模过大，

这会导致后端服务器卡顿甚至崩溃，这些消耗如果在 Velocity / BungeeCord 等代理服务器上，

而反向代理服务器被设计为允许接受大量连接，且反向代理自带单个 IP 多次重新连接的配置：

```
connection_throttle: 4000
connection_throttle_limit: 3
```

这意味着，在 4000ms 内最多能连接 3 次服务器，如果超过则服务器将拒绝登入请求。

### 在代理端安装反假人插件

你可以在代理端安装机器人过滤插件，同样的，由于代理端相较后端服务器在面对大量连接时更加高效，请务必在**代理端**安装插件

以下是推荐的反机器人插件列表

| 名称                                                 | 介绍                             | 支持平台                 | 缺点                            |
|----------------------------------------------------|--------------------------------|----------------------|-------------------------------|
| [Sonar](https://github.com/jonesdevelopment/sonar) | 轻量级反机器人，皆在检测和移除机器人，而不影响任何真正的玩家 | Velocity, BungeeCord | 暂时没有？                         |
| [LimboFilter](https://github.com/jonesdevelopment/sonar) | 强大的过滤机器人方案 | Velocity | 笨重且配置复杂，且仅在必要的时候提供更新。 (缺少维护)  |
| [nAntiBot](https://en.docs.nickuc.com/v/nantibot) | 一个高效反机器人插件 | Spigot, Velocity, BungeeCord | 依赖云服务，无法在服务器网络不好的情况下使用该插件。    |
| [EpicGuard](https://github.com/4drian3d/EpicGuard) | 基于事件的反机器人和反VPN插件 | Waterfall (停止维护), Paper, Velocity | 容易绕过(但没那么烦人)，且只支持特定的Paper服务端。 |

:::warning

该列表目前仅列出了免费的反机器人插件，实际情况可能需要使用者自行决定。使用插件直接对抗超大规模的 MOTD 攻击等是不太现实的。

如果正在遭受这种攻击，最合理的办法是提升服务器带宽或使用专门针对于此类攻击的代理 ([点这里](#使用第三方Minecraft代理))。

:::

### 付费防御核心

如果您非常有钱，您可以打开服务端[核心选择](/docs/process/cross-server/server-core-choose)，选择那些付费的服务端核心，NullCordX 是一个较好的选择

但在没有想好的情况下 **不建议为反机器人付费**

### 网络层攻击防御

#### 将服务器托管到高防机房/购买高防 VPS

对于大多数 MC 服务器，托管到拥有 150G 防御是足够的，性价比较高，一般托管一个月大概 800 RMB / 月，速率为 50 Mbps

最多只建议选择到 300G 防御（再多就摆烂吧这是想让你倒闭的），如果是 VPS 建议询问 VPS 提供商咨询防御能力。

#### 套 CloudFlare

最稳定的办法还是白嫖 CF 的免费套餐，无限防御流量，唯一不太好的地方就是免费用户是没有办法使用国内节点的，所以国内访问会较慢

#### 使用第三方Minecraft代理

例如 TCPShield 和 Infinity-Filter

包含专门针对于缓解 Minecraft 攻击的均衡负载代理，且能够有效隐藏服务器 IP 地址。

缺点是目前似乎还没有任何一家这样的代理拥有国内服务器(延迟高)，且需要花费一点时间设置。

且这些代理的免费套餐都具有一定的限制(例如限制玩家数或流量)，直到升级套餐。

除非遭受了无法缓解的大型攻击，否则使用前请三思。

#### 狂套 Frp

这个方法比较缺德，我们只需要疯狂 Frp ，一个 Frp 被打死了，我们就换另一个 Frp ，通知玩家重新连接就可以，但是缺点就是比较**缺德**，而且可能面临被清退！

#### 更换 IP

打服务器是需要 IP 地址的。大不了我疯狂弹IP地址，你一打死我就换，再装备个 DDNS ，换 IP 还能顺便更新 DNS 记录

:::danger

如果您使用的是腾讯云之类的大厂 VPS ，永远不要尝试硬扛 DDOS ，待会被黑洞一年别怪我

:::