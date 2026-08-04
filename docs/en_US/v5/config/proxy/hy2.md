# Hysteria2

## Hysteria2 Inbound

inbound.hysteria2

> `packetEncoding`: \["None" | "Packet" | "Stream"\]

UDP packet encoding method. The default is `None`.

When the value is `None`, UDP traffic is mapped separately for each destination address and port (Address and Port-Dependent Mapping).

When the value is `Packet`, each UDP packet is encoded together with its destination address while preserving packet boundaries. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1.

When the value is `Stream`, each UDP packet and its destination address are length-prefixed and framed over a byte stream. A compatible outbound can restore the packets as an Endpoint Independent Mapping UDP connection. This UDP behavior is also known as Full Cone or NAT1. This would allow UDP connection to be passed through proxy protocol that does not support packet based communication. (v5.53.0+)

## Hysteria2 Outbound

outbound.hysteria2

```json
{
  "server": [
    {
      "address": "127.0.0.1",
      "port": 1234
    }
  ]
}
```

> `address`: string

The server address. Both IP addresses and domain names are supported.

> `port`: number

The server port number.

## Hysteria2 Compatibility

To use Hysteria2 in a way that is fully compatible with the [official implementation](https://hysteria.network/), also set the transport layer to Hysteria2 and configure a password.

:::tip

- When configuring TLS, you can use `allowInsecure` or the system root certificate store. Self-signed certificates and `PinnedPeerCertificateChainSha256` are not currently supported.

- If TLS is not configured, `allowInsecure` is used by default.

:::

## Best Practices

- UDP is not needed

If you do not need to proxy UDP, `vmess + hysteria2`, `trojan + hysteria2`, and `hysteria2 + hysteria2` (proxy layer + transport layer) have the same effect. `trojan + hysteria2` may provide better performance.

- UDP is needed (transparent proxying, SOCKS5, etc.)

Combining `vmess + hysteria2` or `trojan + hysteria2` results in UDP over stream. Refer to TUIC's `quic` UDP mode.

They have the same effect when proxying TCP.

- There is no need to use gRPC, HTTP/2, smux, or other multiplexing mechanisms

QUIC resolves the issues introduced by multiplexing and provides multiplexing itself, so no additional mechanism is needed.

(This document is machine generated. Please report any mistakes inside.)
