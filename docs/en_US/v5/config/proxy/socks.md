# Socks
Socks Protocol can be used to exchange proxied traffic with other applications.


## Socks Inbound
* Name: `socks`
* Type: Inbound Protocol
* ID: `inbound.socks`

> `address` : string

The server address for the purpose of UDP communication.

> `udpEnabled`: true | false

Is UDP support enabled.

> `packetEncoding`: \["None" | "Packet" | "Stream"\]

UDP packet encoding method. The default is `None`.

When the value is `None`, UDP traffic is mapped separately for each destination address and port (Address and Port-Dependent Mapping).

When the value is `Packet`, each UDP packet is encoded together with its destination address while preserving packet boundaries. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1.

When the value is `Stream`, each UDP packet and its destination address are length-prefixed and framed over a byte stream. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1. This would allow UDP connection to be passed through proxy protocol that does not support packet based communication. (v5.53.0+)

## Socks Outbound
* Name: `socks`
* Type: Outbound Protocol
* ID: `outbound.socks`

> `address`: string

The server address.

> `port`: number

The server port.
