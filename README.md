# SNI Proxy

A high-performance SNI (Server Name Indication) proxy with DNS server capabilities, written in Go. This proxy supports multiple protocols including DNS over UDP/TCP/TLS/QUIC, HTTP/HTTPS proxying, and includes advanced ACL (Access Control List) features.

## Features

- **Multi-Protocol DNS Support**
  - DNS over UDP (Port 53)
  - DNS over TCP (Port 53)
  - DNS over TLS - DoT (Port 853)
  - DNS over QUIC - DoQ (Port 8853)
  - DNS over HTTPS - DoH (Configurable SNI)

- **HTTP/HTTPS Proxy**
  - HTTP proxy with configurable ports
  - HTTPS SNI-based proxying
  - TLS termination support

- **Access Control Lists (ACL)**
  - GeoIP-based filtering
  - Domain-based filtering
  - CIDR/IP-based filtering
  - FQDN override rules

- **Monitoring**
  - Prometheus metrics endpoint
  - Detailed logging with configurable levels

- **Upstream Support**
  - SOCKS5 proxy support for upstream traffic
  - Configurable upstream DNS servers
  - DNS over SOCKS5

## Quick Start with Docker

### Build Docker Image

```bash
docker build -t sniproxy:latest .
```

### Run with Docker

```bash
docker run -d \
  --name sniproxy \
  --restart unless-stopped \
  -p 53:53/udp \
  -p 53:53/tcp \
  -p 853:853/tcp \
  -p 8853:8853/udp \
  -p 8080:80/tcp \
  -p 443:443/tcp \
  -p 2030:2030/tcp \
  -v /root/sniproxy/cmd/sniproxy/config.defaults.yaml:/etc/sniproxy/config.yaml:ro \
  -v /root/sniproxy/domains.csv:/root/sniproxy/domains.csv:ro \
  -v /etc/letsencrypt:/etc/letsencrypt:ro \
  sniproxy:latest
```

### Port Mappings

| Host Port | Container Port | Protocol | Description |
|-----------|---------------|----------|-------------|
| 53 | 53 | UDP | DNS over UDP |
| 53 | 53 | TCP | DNS over TCP |
| 853 | 853 | TCP | DNS over TLS (DoT) |
| 8853 | 8853 | UDP | DNS over QUIC (DoQ) |
| 8080 | 80 | TCP | HTTP Proxy |
| 443 | 443 | TCP | HTTPS Proxy |
| 2030 | 2030 | TCP | Prometheus Metrics |

### Volume Mounts

- **Configuration**: `/root/sniproxy/cmd/sniproxy/config.defaults.yaml` → `/etc/sniproxy/config.yaml`
- **Domain List**: `/root/sniproxy/domains.csv` → `/root/sniproxy/domains.csv`
- **TLS Certificates**: `/etc/letsencrypt` → `/etc/letsencrypt` (for DoT, DoQ, DoH, and HTTPS)

## Building from Source

### Prerequisites

- Go 1.21 or later
- Root/sudo access (for binding to privileged ports)

### Build

```bash
go build -o sniproxy ./cmd/sniproxy
```

### Run

```bash
sudo ./sniproxy
```

## Configuration

The application uses a YAML configuration file. By default, it looks for `config.defaults.yaml` in the same directory as the binary.

### Environment Variables

You can override any configuration parameter using environment variables with the `SNIPROXY_` prefix, followed by the full tree of the parameter separated by double underscores (`__`).

**Example:**
```bash
# Override DNS bind address
SNIPROXY_GENERAL__BIND_DNS_OVER_UDP=0.0.0.0:5555

# Override upstream DNS
SNIPROXY_GENERAL__UPSTREAM_DNS=udp://8.8.8.8:53
```

### Key Configuration Options

#### General Settings

```yaml
general:
  upstream_dns: udp://45.90.28.89:53
  bind_dns_over_udp: "0.0.0.0:53"
  bind_dns_over_tcp: "0.0.0.0:53"
  bind_dns_over_tls: "0.0.0.0:853"
  bind_dns_over_quic: "0.0.0.0:8853"
  bind_http: "0.0.0.0:80"
  bind_https: "0.0.0.0:443"
  bind_prometheus: "0.0.0.0:2030"
  tls_cert: /etc/letsencrypt/live/test.imzami.com/fullchain.pem
  tls_key: /etc/letsencrypt/live/test.imzami.com/privkey.pem
  public_ipv4: 103.159.37.151
  public_ipv6: 2400:7920:0:3:24b:c2ff:fe5f:183e
  log_level: info
```

#### ACL Configuration

```yaml
acl:
  domain:
    enabled: true
    path: /opt/sniproxy/domains.csv
    refresh_interval: 1h0m0s
  
  geoip:
    enabled: false
    blocked: []
    allowed: []
    path: ""
    refresh_interval: 24h0m0s
  
  cidr:
    enabled: false
    path: ""
    refresh_interval: 1h0m0s
```

### Domain List Format

The `domains.csv` file contains domains that are allowed to be proxied. Format:

```csv
domain.example.com.,suffix
subdomain.example.com.,exact
```

- **suffix**: Matches the domain and all its subdomains
- **exact**: Matches only the exact domain

## Monitoring

### Prometheus Metrics

Access Prometheus metrics at: `http://<server-ip>:2030/metrics`

### Logging

Logs are output to stdout with configurable levels:
- `debug`
- `info`
- `warn`
- `error`

To disable colored output (for file logging):
```bash
NO_COLOR=true ./sniproxy
```

## DNS over HTTPS (DoH)

DoH is available when the override ACL is enabled with a `doh_sni` configured:

```yaml
acl:
  override:
    enabled: true
    doh_sni: "test.imzami.com"
```

Access DoH at: `https://test.imzami.com/dns-query`

## Security Considerations

- The proxy requires TLS certificates for DoT, DoQ, DoH, and HTTPS termination
- Use Let's Encrypt or your own certificates
- Ensure proper firewall rules are in place
- Domain filtering is enabled by default to control which domains can be proxied
- Consider enabling GeoIP filtering for additional security

## License

See [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit pull requests or open issues.

## Support

For issues, questions, or contributions, please visit the project repository.
