# Command-line reference

V2Ray v5 uses subcommands in the following form:

```bash
v2ray <command> [options] [arguments]
```

On Windows, replace `v2ray` with `v2ray.exe` in the examples below.

For each command or subcommand, options must appear before its first positional argument. After the first positional argument or `--`, later tokens are no longer parsed as options. This page uses the `-option` spelling; the commands also accept `--option`. Commands that run another program use `--` to separate V2Ray options from that program and its arguments.

Use the built-in help to see the commands included in your build and their built-in usage text:

```bash
v2ray help
v2ray help <command>
v2ray help <command> <subcommand>
```

The standard full build lists these user-facing top-level commands:

| Command | Description |
| --- | --- |
| `run` | Run V2Ray with configuration files. |
| `test` | Validate configuration files without starting V2Ray. |
| `convert` | Merge and convert configuration files. |
| `api` | Call the management API of a running V2Ray instance. |
| `tls` | Generate certificates and run TLS diagnostics. |
| `uuid` | Generate a UUID. |
| `verify` | Verify an official V2Fly signature. |
| `version` | Print build and version information. |

`v2ray help format-loader` and `v2ray help config-merge` provide supplemental loader and merge examples. The tables below include the additional formats present in the standard full build.

## Run and test configurations

### `v2ray run`

```bash
v2ray run [-format <format>] [-r] [-c <file>]... [-d <directory>]...
```

Starts V2Ray using the selected configuration. Configuration paths must be supplied with `-c` or `-d`; positional configuration paths are not accepted.

### `v2ray test`

```bash
v2ray test [-format <format>] [-r] [-c <file>]... [-d <directory>]...
```

Loads and validates the selected configuration without starting the server. A successful validation prints `Configuration OK.`.

Both commands accept the same configuration options:

| Option | Description |
| --- | --- |
| `-c`, `-config <file>` | Load a local configuration file. May be repeated. |
| `-d`, `-confdir <directory>` | Load supported files from a local directory. May be repeated. |
| `-r` | Search every selected configuration directory recursively. The default is non-recursive. |
| `-format <format>` | Select the parser and directory suffix filter. The default is `auto`. |

### Run and test input formats

Format names for `run` and `test` are lowercase and case-sensitive. Filename-extension matching in directories is case-insensitive.

| Value | Configuration representation | Registered filename extensions | Multiple inputs |
| --- | --- | --- | --- |
| `auto` | Select a registered loader from the filename extension, or try loaders by content for an extensionless single input or standard input. | All extensions below | Only mergeable legacy text formats can be combined. |
| `json` | Legacy/v4 JSON configuration. | `.json`, `.jsonc` | Yes. |
| `toml` | TOML converted to the legacy/v4 configuration schema. | `.toml` | Yes. |
| `yaml` | YAML converted to the legacy/v4 configuration schema. | `.yml`, `.yaml` | Yes. |
| `protobuf`, `pb` | Binary protobuf `core.Config`. | `.pb` | No; exactly one input. |
| `jsonv5` | V5 JSON configuration. | `.v5.json`, `.v5.jsonc` | No; exactly one input. |
| `jsonpb` | Legacy protobuf JSON mapping. | `.pb.json`, `.pbjson` | No; exactly one input. |
| `v2jsonpb` | Protobuf v2 JSON mapping. | `.v2pb.json`, `.v2pbjson` | No; exactly one input. |

The legacy `json` and `jsonv5` loaders accept `#`, `//`, and `/* ... */` comments. An explicit `-format` forces that parser for each `-c` file regardless of its suffix. Under `auto`, a file with an unknown non-empty suffix fails instead of being detected by content.

Automatic detection and directory scanning examine only the final filename extension. A filename such as `config.v5.json` therefore looks like ordinary `.json`. Load V5 JSON reliably with an explicit file and format:

```bash
v2ray test -format jsonv5 -c config.v5.json
```

Because `jsonv5` accepts exactly one source and directory scanning cannot discover `.v5.json` names, use one explicit `-c` file: do not rely on `auto` or `-d` to select a `.v5.json` file, and do not repeat `-c`. Likewise, use an explicit format and `-c` for the compound `.pb.json` and `.v2pb.json` spellings; the single-suffix `.pbjson` and `.v2pbjson` spellings can be detected normally.

### Input order, directories, and fallbacks

- All `-c` files are loaded first in the order in which their flags appear. Directory files follow, even if `-c` and `-d` were interleaved on the command line.
- Repeated `-d` directories are expanded in flag order. Eligible files within each directory are processed in lexical path order. `-r` includes matching files in descendant directories.
- The `json`, `toml`, and `yaml` loaders can merge multiple inputs, including a mixture of those formats under `auto`. The protobuf-family and `jsonv5` loaders require exactly one input.
- If no explicit `-d` is present, an existing directory from [`v2ray.location.confdir`](../config/env.md) is appended, even when `-c` files were provided. The uppercase `V2RAY_LOCATION_CONFDIR` is a fallback alias; the dotted name takes precedence. Any explicit `-d` suppresses this environment directory.
- If an explicit `-d` was supplied but no `-c` file or eligible directory file is found, the command fails instead of entering the fallback chain.

When no source has been selected, V2Ray tries these sources in order:

1. `config.json` in the current working directory;
2. `config.json` in the directory specified by `v2ray.location.config` or its fallback alias `V2RAY_LOCATION_CONFIG`;
3. `config.json` beside the V2Ray executable when neither configuration-location variable is set;
4. standard input when no preceding file exists.

The configuration-location environment value is a directory, not a complete filename. The dotted name takes precedence over its uppercase alias. A missing explicit file is an error and does not trigger fallback. Standard input is only the final fallback, so redirected input is ignored if an earlier source is available.

All configuration arguments are local filesystem paths. The legacy `"stdin:"` special path and HTTP(S) URLs are not supported.

Examples:

```bash
# Merge two legacy configurations in this order.
v2ray run -c 00-base.json -c 10-local.yaml

# Load only YAML files from a directory and its descendants.
v2ray test -format yaml -r -d conf.d

# Read JSON from standard input only if the fallback chain reaches stdin.
v2ray test -format json < config.json
```

## Convert configurations

```bash
v2ray convert [-i <input-format>] [-o <output-format>] [-r] [file-or-directory]...
```

`convert` loads and merges local positional inputs before writing the result to standard output. It has no `-c`, `-d`, or output-file option; redirect stdout to save the result. Input and output format names are case-insensitive.

| Option | Description |
| --- | --- |
| `-i`, `-input <format>` | Select the input parser. The default is `auto`. See the input table below. |
| `-o`, `-output <format>` | Select the stdout representation. The default is `json`. See the output table below. |
| `-r` | Search positional input directories recursively. The default is non-recursive. |

### Convert input formats

| Value | Meaning | Directory extensions |
| --- | --- | --- |
| `auto` | Detect and merge legacy JSON, TOML, and YAML. | `.json`, `.jsonc`, `.toml`, `.yml`, `.yaml` |
| `json` | Parse every input as legacy/v4 JSON. | `.json`, `.jsonc` |
| `toml` | Parse every input as TOML. | `.toml` |
| `yaml` | Parse every input as YAML. | `.yml`, `.yaml` |
| `jsonv5` | Parse V5 JSON. Unlike `run` and `test`, `convert` can merge multiple V5 JSON inputs. | `.json`, `.jsonc` |

An explicit input format controls parsing regardless of an explicit file's suffix. Under `auto`, unknown suffixes fail, while extensionless inputs are tried by content. Positional files and directories may be mixed and repeated; each directory expands in place in lexical order. With no path arguments—or when directory expansion produces no eligible files—`convert` reads standard input. Nonexistent paths fail. URL inputs are not supported.

### Convert output formats

| Value | Output |
| --- | --- |
| `json` | Compact JSON serialization of the merged map. |
| `toml` | TOML serialization of the merged map. |
| `yaml` | YAML serialization of the merged map. |
| `protobuf`, `pb` | Binary protobuf configuration. |
| `jsonpb` | Legacy protobuf JSON mapping. |
| `v2jsonpb` | Protobuf v2 JSON mapping. |

The text outputs (`json`, `toml`, and `yaml`) serialize the merged data without building a V2Ray instance, and therefore do not validate the complete configuration. They also do not translate a legacy/v4 schema into the V5 schema, or vice versa. Protobuf-family outputs build and validate the configuration first.

For a protobuf-family output from TOML, leave the input format as `auto`; explicitly combining `-i toml` with `protobuf`, `jsonpb`, or `v2jsonpb` is not supported by the current conversion path.

### Merge behavior

Inputs are merged in resolved order. A later scalar replaces an earlier scalar, nested objects merge recursively, and arrays append. Array objects with the same non-empty `tag` or `_tag` normally merge; under `dns.servers`, only `_tag` requests a merge so ordinary DNS server tags may repeat. Numeric `_priority` values sort array entries in ascending order. The helper `_tag` and `_priority` properties are removed from the final result. An incoming `null` preserves an existing value, while incompatible value types cause an error. Use `v2ray help config-merge` for a longer example.

Examples:

```bash
# Convert a V5 JSON configuration to validated binary protobuf.
v2ray convert -i jsonv5 -o protobuf config.v5.json > config.pb

# Merge all eligible files in a directory and emit YAML.
v2ray convert -r -o yaml path/to/config-directory > config.yaml

# Merge legacy JSON and YAML in positional order.
v2ray convert base.json local.yaml > merged.json
```

## API commands

The `api` commands manage a running V2Ray instance over plaintext gRPC. The corresponding service must be enabled in the instance's [API configuration](../config/api.md), and traffic to the configured API tag must be routed to it.

| Commands | Required `api.services` entry |
| --- | --- |
| `log` | `LoggerService` |
| `stats` | `StatsService` |
| `bi`, `bo` | `RoutingService` |
| `adi`, `ado`, `rmi`, `rmo` | `HandlerService` |

Most API commands accept these shared options:

| Option | Description |
| --- | --- |
| `-s`, `-server <host:port>` | API server address. The default is `127.0.0.1:8080`. |
| `-t`, `-timeout <seconds>` | Timeout for the blocking connection and API operation. The default is 3 seconds; use a positive integer. |

Use nested help to inspect the commands included in a particular build:

```bash
v2ray help api stats
```

### Follow or restart logs

```bash
v2ray api log [-restart] [-s <host:port>] [-t <seconds>]
```

Without `-restart`, the command follows the remote log stream until the server closes it, an error occurs, or the user interrupts it. The follow operation ignores `-timeout`. With `-restart`, it performs one logger restart using the configured timeout and exits.

```bash
v2ray api log
v2ray api log -restart -s 127.0.0.1:8080
```

### Query statistics

```bash
v2ray api stats [-regexp] [-reset] [-runtime] [-json] \
  [-s <host:port>] [-t <seconds>] [pattern]...
```

| Option | Description |
| --- | --- |
| `-regexp` | Interpret every pattern as a Go regular expression instead of a substring. |
| `-reset` | Return each matched counter's current value and atomically reset it to zero. |
| `-runtime` | Return process uptime, memory, goroutine, allocation, and garbage-collection statistics instead of counters. |
| `-json` | Emit the protobuf response as JSON instead of the human-readable table. |

Patterns are case-sensitive and combined with OR: a counter is selected when any pattern matches. Without a pattern, all registered counters are returned. `-runtime` ignores positional patterns, `-regexp`, and `-reset`, but it can be combined with `-json`.

The plain counter table sorts counters by name, displays values in byte units, and includes a total. JSON retains the unscaled counter values using protobuf's JSON representation. Traffic counters must also be enabled through the [`stats` configuration](../config/stats.md) and corresponding [policy flags](../config/policy.md); enabling `StatsService` alone does not create them.

```bash
v2ray api stats 'inbound>>>api>>>traffic'
v2ray api stats -regexp '^outbound>>>.*>>>traffic>>>downlink$'
v2ray api stats -runtime -json
```

### Inspect or override a balancer

```bash
v2ray api bi [-json] [-s <host:port>] [-t <seconds>] <balancer-tag>
v2ray api bo [-s <host:port>] [-t <seconds>] -b <balancer-tag> <outbound-tag>
v2ray api bo [-s <host:port>] [-t <seconds>] -b <balancer-tag> -r
```

`bi` accepts exactly one balancer tag and displays its active override and the targets currently preferred by its strategy. `-json` emits the complete protobuf response as JSON.

`bo` accepts these command-specific options:

| Option | Description |
| --- | --- |
| `-b`, `-balancer <tag>` | Target balancer tag. Required. |
| `-r`, `-remove` | Remove the balancer's active override; no outbound tag is used. |

Without `-r`, the outbound tag is required and becomes the balancer's forced selection. If `-r` and an outbound tag are both present, removal takes precedence and the outbound tag is ignored. An override changes only the running process; it does not update the configuration file and is lost when V2Ray restarts.

```bash
v2ray api bi -json main-balancer
v2ray api bo -b main-balancer direct
v2ray api bo -b main-balancer -r
```

### Add or remove inbound and outbound handlers

```bash
v2ray api adi [-s <host:port>] [-t <seconds>] [-format <format>] [-r] [file-or-directory]...
v2ray api ado [-s <host:port>] [-t <seconds>] [-format <format>] [-r] [file-or-directory]...
v2ray api rmi [-s <host:port>] [-t <seconds>] [-format <format>] [-r] [file-or-directory]...
v2ray api rmo [-s <host:port>] [-t <seconds>] [-format <format>] [-r] [file-or-directory]...
v2ray api rmi [-s <host:port>] [-t <seconds>] -tags <tag>...
v2ray api rmo [-s <host:port>] [-t <seconds>] -tags <tag>...
```

`adi` and `ado` add the inbounds or outbounds found in the merged configuration. `rmi` and `rmo` either load configuration and remove the handlers named by its tags, or treat every positional argument as a literal tag when `-tags` is present.

| Option | Description |
| --- | --- |
| `-format <format>` | Input format: `auto`, `json`, `toml`, or `yaml`. The default is `auto`; values are lowercase. |
| `-r` | Expand positional directories recursively. |
| `-tags` | `rmi`/`rmo` only: interpret all positional inputs as tags instead of paths. `-format` and `-r` then have no effect. |
| `-s`, `-server <host:port>` | API server address. The default is `127.0.0.1:8080`. |
| `-t`, `-timeout <seconds>` | Timeout shared by connection setup and the entire batch. The default is 3 seconds. |

These commands use the legacy/v4 configuration loader, not `jsonv5`:

| Value | Directory extensions |
| --- | --- |
| `auto` | `.json`, `.jsonc`, `.toml`, `.yml`, `.yaml` |
| `json` | `.json`, `.jsonc` |
| `toml` | `.toml` |
| `yaml` | `.yml`, `.yaml` |

Multiple local files and expanded directories are merged in positional order. Each input is a top-level configuration containing `inbounds` or `outbounds`, rather than a bare handler object. With no positional path, configuration is read from standard input. `-r` affects only directory traversal. Configuration-based removal requires every target handler to have an explicit non-empty tag.

Handlers are processed sequentially under one timeout. If an item fails, the command stops and earlier successful changes remain active. All additions and removals affect only the running instance and are lost on restart.

```bash
v2ray api adi -format json < inbounds.json
v2ray api ado -r conf.d
v2ray api rmi -tags inbound-a inbound-b
v2ray api rmo -tags temporary-outbound
```

## TLS and utility commands

### Generate a TLS certificate

```bash
v2ray tls cert [-domain <dns-name>]... [-name <common-name>] \
  [-org <organization>] [-ca] [-expire <days>] \
  [-json=<true|false>] [-file <prefix>]
```

Generates a self-signed ECDSA P-256 certificate and private key.

| Option | Description |
| --- | --- |
| `-domain <dns-name>` | Add a non-empty DNS Subject Alternative Name. May be repeated. This command does not provide an IP-address SAN option. |
| `-name <name>` | Set the Common Name. The default is `V2Ray Inc`. |
| `-org <organization>` | Set the organization. The default is `V2Ray Inc`. |
| `-ca` | Mark the generated self-signed certificate as a CA. It does not sign another certificate. |
| `-expire <days>` | Validity as a non-negative integer number of days. The default is `90`. |
| `-json=<boolean>` | Accept `true` or `false` and print both the certificate and private key as V2Ray JSON when enabled. The default is `true`. |
| `-file <prefix>` | Write or replace `<prefix>_cert.pem` and `<prefix>_key.pem`. No files are written by default. |

Default JSON output contains `certificate` and `key` arrays whose elements are the lines of the corresponding PEM blocks. `-file` does not suppress that output; use `-json=false` when only PEM files are wanted. Conversely, `-json=false` without `-file` generates the key pair but emits nothing.

```bash
v2ray tls cert -domain example.com -domain www.example.com \
  -file server -json=false
```

### Test a TLS handshake

```bash
v2ray tls ping [-ip <literal-ip>] <domain>
```

Opens two TCP connections to port 443 and attempts TLS 1.2 with ALPN `http/1.1`:

1. without SNI and with certificate verification disabled;
2. with the supplied domain as SNI and normal system-root and hostname verification.

If `-ip` is omitted, the system resolver resolves the domain. Otherwise, `-ip` must be a literal IPv4 or unbracketed IPv6 address; hostnames are not accepted there. The domain is still used for SNI and certificate verification, which makes `-ip` useful for testing a specific server independently of DNS.

The command has no port or timeout option. It prints handshake failures as diagnostic results; a handshake failure alone does not guarantee a non-zero process exit status.

```bash
v2ray tls ping -ip 203.0.113.10 example.com
```

### Calculate a certificate-chain hash

```bash
v2ray tls certChainHash [-cert <certificate-file>]
```

Reads an ordered PEM certificate bundle and prints its certificate-chain SHA-256 hash using standard Base64. Bundle order affects the result. The default input is `cert.pem`; an explicitly empty filename is rejected. The output can be used in [`pinnedPeerCertificateChainSha256`](../v5/config/stream.md).

```bash
v2ray tls certChainHash -cert fullchain.pem
```

### Generate a UUID

```bash
v2ray uuid
```

Prints one cryptographically random 128-bit value as lowercase hexadecimal in `8-4-4-4-12` form. The generator does not set RFC UUID version or variant bits, so the result should not be described specifically as a UUIDv4.

### Verify an official signature

```bash
v2ray verify -sig <signature-file> <file>...
```

Both the signature path and at least one target file are required; `-sig` must appear before the target files. V2Ray does not infer or add a `.sig` suffix.

The command uses the embedded official V2Fly public key to compare the SHA-256 digest of every target with the signed release manifest. One manifest may cover multiple target files. A successful check prints a `+OK` result, the signed version, and the manifest match for each file. A missing match, invalid signature, or unreadable input fails the command.

```bash
v2ray verify -sig v2ray-linux-64.zip.sig v2ray-linux-64.zip
```

### Print the version

```bash
v2ray version
```

Takes no options. The first line contains the V2Ray version and codename, build label, Go runtime version, operating system, and architecture. The second line prints the project introduction.

## Engineering commands

Engineering commands are experimental and build-dependent, and the `engineering` group is intentionally omitted from the top-level command list. Use the following command to see what is available in your binary:

```bash
v2ray help engineering
```

### Generate random data

```bash
v2ray engineering generate-random-data -length <bytes>
```

`-length` is required and must be a positive integer. It is the number of random bytes before encoding, not the length of the printed string. The command reads from the operating system's cryptographic random source and prints one padded standard Base64 value followed by a newline. There is no configured upper bound, although the requested byte slice must fit in available memory.

For example, generate 32 random bytes for key material:

```bash
v2ray engineering generate-random-data -length 32
```

### Discover NAT behavior with STUN

This command is included in standard full builds except Android builds.

```bash
v2ray engineering stun-nat-type-discovery \
  -server <host:port> \
  [-server2 <host:port>] [-timeout <milliseconds>] \
  [-attempts <count>] [-socks5udp <host:port>] \
  [-detect-buggy-nat-mapping] \
  [-mapping-lifetime-max-idle <milliseconds>] \
  [-mapping-lifetime-start-idle <milliseconds>]
```

| Option | Default | Description |
| --- | --- | --- |
| `-server <host:port>` | none | Required primary STUN server. Hostnames are resolved locally, and IPv6 literals must be bracketed. The port may be numeric or a UDP service name. |
| `-server2 <host:port>` | none | Secondary server used to test whether the mapped address is stable across servers and, when enabled, during lifetime probing. |
| `-timeout <milliseconds>` | `3000` | Positive timeout for each STUN transaction group, not for the complete test run. |
| `-attempts <count>` | `3` | Positive number of parallel requests per transaction, improving resilience to UDP loss. |
| `-socks5udp <host:port>` | none | Send SOCKS5-encapsulated UDP directly to an existing UDP relay endpoint. This does not perform a TCP connection, UDP ASSOCIATE, or authentication. |
| `-detect-buggy-nat-mapping` | disabled | Send additional probes for mapping behavior that depends on packet arrival order. |
| `-mapping-lifetime-max-idle <milliseconds>` | `0` | A positive value enables the optional UDP mapping-lifetime probe and sets its maximum idle window; zero or a negative value disables it. |
| `-mapping-lifetime-start-idle <milliseconds>` | `1000` | Initial idle window for the optional lifetime probe. It has no effect while the probe is disabled. |

When lifetime probing is enabled, a non-positive start idle is replaced by 1000 ms, a start above the maximum is clamped to the maximum, and subsequent fresh-socket probes double the idle window until reaching that maximum.

For full filtering and mapping results, the primary server must support RFC 5780 `OTHER-ADDRESS` and `CHANGE-REQUEST`. Without that support, some results are reported as unknown. The output covers NAT filtering, mapping, hairpin behavior, cross-server stability, response-origin anomalies, derived address/port properties, and observed source IPs. Enabling lifetime probing also prints lower and upper estimates for the primary, alternate, and secondary destinations, plus an endpoint-independent aggregate when supported.

```bash
# Basic RFC 5780 discovery.
v2ray engineering stun-nat-type-discovery \
  -server stun.example.com:3478

# Add cross-server stability and arrival-order probes.
v2ray engineering stun-nat-type-discovery \
  -server stun.example.com:3478 \
  -server2 stun2.example.com:3478 \
  -detect-buggy-nat-mapping

# Probe mapping lifetime up to 60 seconds.
v2ray engineering stun-nat-type-discovery \
  -server stun.example.com:3478 \
  -mapping-lifetime-max-idle 60000

# Send the tests through an already-established SOCKS5 UDP relay.
v2ray engineering stun-nat-type-discovery \
  -server stun.example.com:3478 -socks5udp 127.0.0.1:1080
```

### Send a command through SOCKS5 with socks5ify

`socks5ify` starts a command inside isolated user, mount, and network namespaces. A TUN interface carries the command's TCP and UDP traffic to an embedded V2Ray instance, which sends it through an upstream SOCKS5 server.

```bash
v2ray engineering socks5ify \
  -socks <host:port|socks://[user[:password]@]host:port|socks5://[user[:password]@]host:port> \
  [options] [-- <command> [arguments...]]
```

`-socks` is required. Use `--` before a child command so its flags are not parsed as socks5ify options. If no command is supplied, socks5ify starts the shell named by `$SHELL`, falling back to `/bin/sh`.

#### Requirements

- socks5ify is available only in full Linux builds that are not built with `confonly`.
- Linux must provide user, mount, and network namespaces and `/dev/net/tun`.
- Root is not inherently required when unprivileged user namespaces are enabled.
- The upstream SOCKS5 server must support UDP when the child command needs UDP.

#### Upstream and execution options

| Option | Description |
| --- | --- |
| `-socks <address>` | Required upstream server as bare `host:port`, `socks5://...`, or the equivalent `socks://...` alias. A URL may contain credentials but no path other than an optional `/`. The port must be from 1 through 65535, and IPv6 literals must be enclosed in brackets. |
| `-socks-user <username>` | Non-empty value independently overrides a username included in the SOCKS URL. The default is empty. |
| `-socks-pass <password>` | Non-empty value independently overrides a password included in the SOCKS URL. The default is empty. |
| `-quiet` | Suppress non-error messages and access logs from the embedded V2Ray instance. Disabled by default. |
| `-keep-uid` | Run the command as the original non-root caller's UID and primary GID after namespace setup. Disabled by default and has no effect for a root caller. |

Credentials default to none. Without `-keep-uid`, the command runs as UID 0 inside the user namespace. IPv6, DNS replacement, resolver bind mounting, and additional file bind mounts are also disabled unless requested.

#### TUN options

| Option | Default | Description |
| --- | --- | --- |
| `-tun-name <name>` | `socks5ify0` | TUN interface name; it must not be empty. |
| `-mtu <bytes>` | `1500` | TUN MTU; it must be positive. |
| `-tun-ipv4-host <ip>` | `198.18.0.1` | V2Ray-side IPv4 address. |
| `-tun-ipv4-guest <ip>` | `198.18.0.2` | Namespace-side IPv4 address. |
| `-tun-ipv4-prefix <bits>` | `30` | IPv4 prefix length from 0 through 32. `-tun-ipv4-mask` is an alias. |
| `-ipv6` | disabled | Add an IPv6 address and default route. |
| `-tun-ipv6-host <ip>` | `fd00:736f:636b:35::1` | V2Ray-side IPv6 address. |
| `-tun-ipv6-guest <ip>` | `fd00:736f:636b:35::2` | Namespace-side IPv6 address. |
| `-tun-ipv6-prefix <bits>` | `126` | IPv6 prefix length from 0 through 128. `-tun-ipv6-mask` is an alias. |

Changing an IPv6 address or prefix option without also enabling `-ipv6` is an error.

#### DNS and mount options

| Option | Description |
| --- | --- |
| `-dns <server[,server...]>` | Generate an `/etc/resolv.conf` containing the listed `nameserver` values inside the mount namespace. Whitespace is trimmed and empty list items are skipped. |
| `-resolv-conf <path>` | Bind-mount an existing resolver file onto `/etc/resolv.conf` inside the mount namespace. |
| `-bind-file <source:target>` | Bind-mount a file inside the namespace. May be repeated; the source and non-directory target must already exist. |

`-dns` and `-resolv-conf` are mutually exclusive.

Examples:

```bash
# Open an interactive shell whose traffic uses the local SOCKS5 server.
v2ray engineering socks5ify -socks 127.0.0.1:1080

# Run one command through an authenticated SOCKS5 server.
v2ray engineering socks5ify \
  -socks socks5://user:password@127.0.0.1:1080 \
  -- curl https://example.com

# Preserve the caller's identity and use an isolated DNS configuration.
v2ray engineering socks5ify \
  -socks 127.0.0.1:1080 -keep-uid -dns 1.1.1.1
```

## V2Ctl

V2Ray v5 consolidated the former `v2ray` flags and separate `v2ctl` executable into the `v2ray` command tree. Use these replacements when migrating older instructions:

| Legacy command | Current command |
| --- | --- |
| `v2ray -version` | `v2ray version` |
| `v2ray -test -config config.json` | `v2ray test -c config.json` |
| `v2ray -config config.json` | `v2ray run -c config.json` |
| `v2ctl config` | `v2ray convert -o protobuf` |
| `v2ctl cert` | `v2ray tls cert` |
| `v2ctl tlsping` | `v2ray tls ping` |
| `v2ctl uuid` | `v2ray uuid` |
| `v2ctl verify` | `v2ray verify -sig <signature-file> <file>` |
| `v2ctl fetch` | No direct replacement; use an HTTP client. |

The generic `v2ctl api <Service.Method> <Request>` interface has no direct command-line replacement. Use one of the supported `v2ray api` subcommands. For example, the former logger restart call is now:

```bash
v2ray api log -restart
```

(This document is machine generated. Please report any mistakes inside.)
