# Tun
* Name: `tun`
* Type: Service
* ID: `service.tun`

Tun is an interface that accepts and forwards packet based network traffic, and converts traffic processed by inbounds to streams. (v5.9.0+)

Look at its [pull request](https://github.com/v2fly/v2ray-core/pull/2541) for working examples of how to configure it.

Supports Linux operating system on amd64 and arm64.

### Tun

> `name`: string

The name of the tun interface.

> `mtu`: number

The mtu of the tun interface. The recommanded value is 1500.

> `tag`: string

The inbound tag associated with tun generated traffic.

> `ips`: [ [IPObject](#ipobject) ]

The ip address associated with tun. You will need to add them to tun on the operating system side as well.

> `routes`: [ [RouteObject](#routeobject) ]

The routes associated with tun. You will need to add them to tun on the operating system side as well.

> `enablePromiscuousMode`: bool

Recommanded to set to true.

> `enableSpoofing`: bool

Recommanded to set to true.

> `packetEncoding`: \["None" | "Packet" | "Stream"\]

UDP packet encoding method. The default is `None`.

When the value is `None`, UDP traffic is mapped separately for each destination address and port (Address and Port-Dependent Mapping).

When the value is `Packet`, each UDP packet is encoded together with its destination address while preserving packet boundaries. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1.

When the value is `Stream`, each UDP packet and its destination address are length-prefixed and framed over a byte stream. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1. This would allow UDP connection to be passed through proxy protocol that does not support packet based communication. (v5.53.0+)

> `sniffingSettings`: [SniffingObject](../inbound.md#sniffingobject)

The sniffing settings for the tun inbound. It allows the connection to be routed based on its content and metadata.（v5.11.0+）

### IPObject

> `ip`: [ number ]

The IP address in base 10 expression.

> `prefix`: number

### RouteObject

> `ip`: [ number ]

> `prefix`: number
