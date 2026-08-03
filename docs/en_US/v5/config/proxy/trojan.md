# Trojan

:::tip
Trojan is designed to operate in correctly configured TLS connections, as it does not provide encryption on its own.
:::

## Trojan Inbound (simplified)

* Name: `trojan`
* Type: Inbound Protocol
* ID: `inbound.trojan`

### Structure

```json
{
  "users": [],
  "packetEncoding": "None"
}
```

### Fields

> `users` : [string]

A set of recognized password for this inbound.

> `packetEncoding`: \["None" | "Packet" | "Stream"\]

UDP packet encoding method. The default is `None`. (v5.4.0+)

When the value is `None`, UDP traffic is mapped separately for each destination address and port (Address and Port-Dependent Mapping).

When the value is `Packet`, each UDP packet is encoded together with its destination address while preserving packet boundaries. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1.

When the value is `Stream`, each UDP packet and its destination address are length-prefixed and framed over a byte stream. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1. This would allow UDP connection to be passed through proxy protocol that does not support packet based communication. (v5.53.0+)

## Trojan Outbound (simplified)

* Name: `trojan`
* Type: Outbound Protocol
* ID: `outbound.trojan`

### Structure

```json
{
  "address": "",
  "port": 0,
  "password": ""
}
```

### Fields

> `address`: string

The server address. Both IP and domain name is supported.

> `port`: number

The server port number.

> `password`: string

A password recognized by server.

## Trojan Inbound (complete)

* Name: `#v2ray.core.proxy.trojan.ServerConfig`

### Structure

```json
{
  "users": [],
  "packetEncoding": "None",
  "fallbacks": []
}
```

### Fields

> `users`: [[UserObject](../protocol/user.md#user)]

> `packetEncoding`: \["None" | "Packet" | "Stream"\]

The values and default are the same as for `packetEncoding` in the simplified configuration above.

> `fallbacks`: [[FallbackObject](#fallbackobject)]

## Trojan Outbound (complete)

* Name: `#v2ray.core.proxy.trojan.ClientConfig`

### Structure

```json
{
  "server": []
}
```

> `server`: [[ServerEndpoint](../protocol/server_spec.md#serverendpoint)]

## AccountObject

### Structure

```json
{
  "@type": "v2ray.core.proxy.trojan.Account",
  "password": ""
}
```

### Fields

> `@type`: "v2ray.core.proxy.trojan.Account"

> `password`: string

## FallbackObject

### Structure

```json
{
  "alpn": "",
  "path": "",
  "type": "",
  "dest": "",
  "xver": 0
}
```

### Fields

> `alpn`: string

> `path`: string

> `type`: string

The network protocol to use. Common values include "tcp", "tcp4" (IPv4 only), "tcp6" (IPv6 only).

> `dest`: string

The destination address, typically in the form of "host:port", ex: "127.0.0.1:80" .

> `xver`: uint64
