# Pentora Security Scanner - Official Plugin Repository

Official security scanning plugins for [Pentora](https://github.com/pentora-ai/pentora), a modular network security scanner built in Go.

## 🔌 Plugin Overview

This repository contains **19 security plugins** across 5 categories, providing comprehensive vulnerability detection and security compliance checks.

### Plugin Categories

| Category | Plugins | Description |
|----------|---------|-------------|
| **SSH** | 5 | SSH protocol vulnerabilities, weak algorithms, default credentials |
| **HTTP/Web** | 4 | Missing security headers, version disclosure, weak SSL/TLS |
| **TLS/SSL** | 4 | Protocol weaknesses, weak ciphers, certificate issues |
| **Database** | 3 | Default credentials, authentication bypasses |
| **Network** | 3 | Open services (FTP, Telnet), SNMP weaknesses |

## 📦 Installation

Plugins are automatically downloaded using the Pentora CLI:

```bash
# Download all plugins
pentora plugin update

# Download specific category
pentora plugin update --category ssh

# Preview what would be downloaded
pentora plugin update --dry-run
```

Plugins are cached locally in `~/.pentora/plugins/cache/`

## 🚀 Usage

### List Available Plugins

```bash
# List all installed plugins
pentora plugin list

# List by category
pentora plugin list --category tls

# Show detailed information
pentora plugin info "TLS Weak Protocol Version"
```

### Using Plugins in Scans

```bash
# Run scan with vulnerability detection
pentora scan --targets 192.168.1.0/24 --vuln

# Scan with specific categories
pentora scan --targets example.com --vuln --category ssh,http
```

### Verify Plugin Integrity

```bash
# Verify all cached plugins
pentora plugin verify

# Clean invalid plugins
pentora plugin clean
```

## 📋 Plugin List

### SSH Plugins (5)

- **OpenSSH CVE-2024-6387 (regreSSHion)** - Critical RCE vulnerability in OpenSSH
- **SSH Weak Encryption Cipher** - Detects weak/deprecated encryption algorithms (3DES, CBC mode)
- **SSH Weak Key Exchange Algorithm** - Identifies weak KEX algorithms (DH group1-sha1)
- **SSH Weak MAC Algorithm** - Finds weak message authentication codes (MD5, 96-bit)
- **SSH Default Credentials** - Checks for IoT/embedded devices with default credentials

### HTTP/Web Plugins (4)

- **HTTP Missing Security Headers** - Detects missing security headers (CSP, HSTS, X-Frame-Options)
- **HTTP Server Version Disclosure** - Identifies version information leakage
- **HTTP Default Installation Pages** - Finds default web server pages (Apache, nginx, IIS)
- **HTTP Weak SSL/TLS Configuration** - Checks for weak HTTPS configurations

### TLS/SSL Plugins (4)

- **TLS Weak Protocol Version** - Detects SSLv2, SSLv3, TLS 1.0, TLS 1.1
- **TLS Weak Cipher Suite** - Identifies weak ciphers (RC4, DES, export ciphers)
- **TLS Expired Certificate** - Checks for expired SSL/TLS certificates
- **TLS Self-Signed Certificate** - Detects self-signed certificates

### Database Plugins (3)

- **MySQL Default Credentials** - Tests for MySQL default credentials
- **PostgreSQL Default Credentials** - Tests for PostgreSQL default credentials
- **Redis No Authentication** - Detects Redis instances without authentication

### Network Service Plugins (3)

- **Open FTP Service** - Detects open FTP services with anonymous access
- **Open Telnet Service** - Identifies insecure Telnet services
- **Weak SNMP Community String** - Tests for default SNMP community strings

## 🔧 Plugin Format

Plugins are written in YAML with a declarative security check format:

```yaml
name: "TLS Weak Protocol Version"
version: "1.0.0"
type: evaluation
author: "pentora-security"

metadata:
  severity: critical
  tags: [tls, ssl, protocol]
  cve: "CVE-2014-3566"  # POODLE attack
  references:
    - "https://nvd.nist.gov/vuln/detail/CVE-2014-3566"

triggers:
  - data_key: "tls.version"
    condition: exists

match:
  logic: OR
  rules:
    - field: "tls.version"
      operator: "equals"
      value: "SSLv2"
    - field: "tls.version"
      operator: "equals"
      value: "SSLv3"
    - field: "tls.version"
      operator: "equals"
      value: "TLSv1.0"

output:
  vulnerability: true
  severity: critical
  message: "Server supports weak SSL/TLS protocol versions"
  remediation: "Disable SSLv2, SSLv3, TLS 1.0, and TLS 1.1. Only support TLS 1.2 and TLS 1.3"
```

### Plugin Fields

- **name**: Human-readable plugin name
- **version**: Semantic version (e.g., "1.0.0")
- **type**: Plugin type (`evaluation` for security checks)
- **author**: Plugin author/maintainer
- **metadata**: Severity, CVE references, tags, descriptions
- **triggers**: Data context keys that activate the plugin
- **match**: Matching rules using operators (equals, contains, matches, version_lt, etc.)
- **output**: Vulnerability message, severity, and remediation guidance

## 📊 Plugin Manifest

The repository includes an auto-generated `manifest.yaml` with:

- Plugin metadata (name, version, author, description)
- Download URLs (GitHub raw content)
- SHA-256 checksums for integrity verification
- Category index for efficient filtering

Example manifest entry:

```yaml
plugins:
  - name: "TLS Weak Protocol Version"
    version: "1.0.0"
    description: "Detects weak SSL/TLS protocol versions"
    author: "pentora-security"
    categories: [tls]
    url: "https://raw.githubusercontent.com/pentora-ai/pentora-plugins/main/tls/tls-weak-protocol.yaml"
    checksum: "sha256:7fd2b8273c04cf68810c214d3416303b9c163ede61fe9e4180711e3c1aec4e65"
    size: 1337
```

## 🤝 Contributing

We welcome contributions! To add a new plugin:

1. **Fork this repository**
2. **Create a new YAML plugin** in the appropriate category directory
3. **Follow the plugin format** (see examples above)
4. **Test your plugin** locally with Pentora
5. **Submit a pull request** with:
   - Plugin YAML file
   - Description of what it detects
   - Test cases or examples

### Plugin Guidelines

- ✅ Use descriptive names
- ✅ Include CVE/CWE references when applicable
- ✅ Provide clear remediation guidance
- ✅ Test against real-world targets
- ✅ Use appropriate severity levels (critical, high, medium, low, info)
- ✅ Follow semantic versioning

### Regenerating Manifest

After adding/modifying plugins, regenerate the manifest:

```bash
# Clone pentora repository
git clone https://github.com/pentora-ai/pentora.git

# Run manifest generator
go run pentora/cmd/tools/manifest-generator/main.go \
  -dir ./pentora-plugins \
  -output ./pentora-plugins/manifest.yaml \
  -base-url https://raw.githubusercontent.com/pentora-ai/pentora-plugins/main

# Commit and push
git add manifest.yaml
git commit -m "chore: regenerate manifest"
git push
```

GitHub Actions automatically regenerates the manifest on every push.

## 🔒 Security

### Reporting Vulnerabilities

If you discover a security issue in a plugin or the plugin system:

1. **DO NOT** open a public issue
2. Email security@pentora.ai with details
3. Include steps to reproduce and impact assessment

### Plugin Verification

All plugins are:
- ✅ Reviewed by maintainers before merge
- ✅ SHA-256 checksum verified on download
- ✅ Validated against Pentora's plugin schema
- ✅ Tested in CI/CD pipeline

## 📜 License

This repository is licensed under the Apache License 2.0 - see [LICENSE](LICENSE) file for details.

Individual plugins may have additional attribution requirements listed in their YAML files.

## 🔗 Links

- **Pentora Main Repository**: https://github.com/pentora-ai/pentora
- **Documentation**: https://docs.pentora.ai
- **Issue Tracker**: https://github.com/pentora-ai/pentora/issues
- **Discussions**: https://github.com/pentora-ai/pentora/discussions

## 📈 Statistics

- **Total Plugins**: 19
- **Categories**: 5
- **Total Lines**: ~1,200 YAML
- **Checksum Coverage**: 100%
- **Auto-generated Manifest**: Yes

---

**Built with ❤️ by the Pentora Security Team**
