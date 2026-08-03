# Shadowsocks

[Shadowsocks](https://shadowsocks.org) Protocol，mostly compatible with other implementations。

## Shadowsocks Inbound
* Name: `shadowsocks`
* Type: Inbound Protocol
* ID: `inbound.shadowsocks`

> `method` : string

Encryption method，one of [supported encryption methods](#supported encryption methods) .

> `password`: string

A recognized password for this inbound.
Shadowsocks does not mandate the length of the password, but it would be easy to crack a short password,
thus a password of 16 characters or more is recommended.

> `networks`: "tcp" | "udp" | "tcp,udp"

Enabled network type. 
For example, when `"tcp"` is specified, this inbound will only accept TCP traffic.
This value is `"tcp"` by default.

> `packetEncoding`: \["None" | "Packet" | "Stream"\]

UDP packet encoding method. The default is `None`.

When the value is `None`, UDP traffic is mapped separately for each destination address and port (Address and Port-Dependent Mapping).

When the value is `Packet`, each UDP packet is encoded together with its destination address while preserving packet boundaries. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1.

When the value is `Stream`, each UDP packet and its destination address are length-prefixed and framed over a byte stream. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1. This would allow UDP connection to be passed through proxy protocol that does not support packet based communication. (v5.53.0+)

## Shadowsocks Outbound

* Name: `shadowsocks`
* Type: Outbound Protocol
* ID: `outbound.shadowsocks`

> `address`: string

The server address. Both IP and domain name is supported.

> `port`: number

The server port number.

> `method` : string

Encryption method，one of [supported encryption methods](#supported encryption methods) .

> `password`: string

A password recognized by server.

## Supported Encryption Methods

* `"AES_256_GCM"`
* `"AES_128_GCM"`
* `"CHACHA20_POLY1305"`
* `"NONE"`

::: warning
In "NONE" unencrypted and unauthenticated mode, the server will not try to validate the password.

This is typically used when authentication is already completed by the transport layer, like enabling TLS encryption and WebSocket transport with a long and unpredictable path.
:::
