# Socks

标准 Socks 协议实现。

:::warning
如果你将 Socks5 代理在不安全的网络环境中分享给其他人使用，建议搭配防火墙使用。

Rfc: [SOCKS 5 的认证在使用 UDP 时可被绕过](https://github.com/v2fly/v2fly-github-io/issues/104)
:::

## Socks 入站

inbound.socks

> `address` : string

SOCKS5 通过 UDP ASSOCIATE 命令建立 UDP 会话。服务端在对客户端发来的该命令的回复中，指定客户端发包的目标地址。默认值为空。

若此项为空，对于通过本地回环 IPv4/IPv6 连接的客户端，回复对应的回环 IPv4/IPv6 地址；对于非本机的客户端，回复当前入站的监听地址。

你可以通过配置此项使 V2Ray 固定回复你配置的地址。如果你不知道此项的作用，留空即可。

> `udpEnabled`: true | false

是否开启 UDP 协议的支持。默认值为 `false`。

> `packetEncoding`: \["None" | "Packet" | "Stream"\]

UDP 包编码方式，默认值为 `None`。

当值为 `None` 时，UDP 流量会按目标地址和端口分别映射（Address and Port-Dependent Mapping）。

当值为 `Packet` 时，每个 UDP 包会与其目标地址一起编码，同时保留数据包边界。兼容的出站可将其还原为端点独立映射（Endpoint Independent Mapping）的 UDP 连接；这种 UDP 行为也称为 Full Cone 或 NAT1。

当值为 `Stream` 时，每个 UDP 包及其目标地址会通过长度前缀在字节流中分帧。兼容的出站可将其还原为端点独立映射（Endpoint Independent Mapping）的 UDP 连接；这种 UDP 行为也称为 Full Cone 或 NAT1。这使 UDP 连接可以通过不支持基于数据包通信的代理协议传输。（v5.53.0+）

## Socks 出站

outbound.socks

> `address`: string

服务器地址。

:::tip
仅支持连接到 Socks 5 服务器。
:::

> `port`: number

服务器端口。

如果出现 `Unknown socks version: 72` 说明目标socks5服务器实现有误
https://github.com/v2fly/v2ray-core/pull/2933
