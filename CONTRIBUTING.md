# Contributing to Pentora Plugins

Thank you for your interest in contributing to the Pentora plugin ecosystem! This guide will help you create high-quality security plugins.

## Table of Contents

- [Getting Started](#getting-started)
- [Plugin Architecture](#plugin-architecture)
- [Writing a Plugin](#writing-a-plugin)
- [Testing Guidelines](#testing-guidelines)
- [Submission Process](#submission-process)
- [Best Practices](#best-practices)

## Getting Started

### Prerequisites

- **Pentora installed**: Get it from [pentora-ai/pentora](https://github.com/pentora-ai/pentora)
- **Go 1.24+**: For running the manifest generator
- **Basic YAML knowledge**: Plugins are written in YAML
- **Security knowledge**: Understanding of vulnerabilities you're detecting

### Development Environment

1. **Fork the repository**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/pentora-plugins.git
   cd pentora-plugins
   ```

2. **Install Pentora**:
   ```bash
   cd ../
   git clone https://github.com/pentora-ai/pentora.git
   cd pentora
   make binary
   ```

3. **Test environment setup**:
   ```bash
   # Symlink your local plugins for testing
   mkdir -p ~/.local/share/pentora/plugins
   ln -s $(pwd)/../pentora-plugins ~/.local/share/pentora/plugins/dev
   ```

## Plugin Architecture

### Data-Centric Trigger System

Pentora uses a **data-centric** architecture where plugins trigger based on **parsed data availability**, NOT on port numbers.

```yaml
# ✅ CORRECT - Data-centric trigger
triggers:
  - data_key: ssh.banner
    condition: exists
    value: true

# ❌ WRONG - Port-based trigger (deprecated)
triggers:
  - port: 22
    service: ssh
```

**Why data-centric?**
- **Stronger signal**: Requires successful protocol parsing (not just port scanning)
- **More accurate**: Reduces false positives from misidentified services
- **Service-agnostic**: Works for non-standard ports (e.g., SSH on port 2222)

### Available Data Keys

Common data keys produced by Pentora's parsing modules:

| Data Key | Description | Example Value |
|----------|-------------|---------------|
| `ssh.banner` | SSH server banner | `"SSH-2.0-OpenSSH_8.9"` |
| `ssh.version` | SSH version string | `"OpenSSH_8.9"` |
| `ssh.kex_algorithms` | Key exchange algorithms | `["diffie-hellman-group14-sha256"]` |
| `ssh.encryption_algorithms` | Encryption ciphers | `["aes128-ctr", "aes256-gcm"]` |
| `ssh.mac_algorithms` | MAC algorithms | `["hmac-sha2-256"]` |
| `http.server` | HTTP Server header | `"nginx/1.18.0"` |
| `http.headers` | HTTP response headers | `{"Server": "nginx/1.18.0"}` |
| `http.body` | HTTP response body | `"<html>..."` |
| `tls.version` | TLS protocol version | `"TLSv1.2"` |
| `tls.cipher_suites` | TLS cipher suites | `["TLS_AES_128_GCM_SHA256"]` |
| `tls.certificate.not_after` | Certificate expiration | `"2025-12-31T23:59:59Z"` |
| `tls.certificate.issuer` | Certificate issuer | `"CN=localhost"` |
| `mysql.version` | MySQL version | `"8.0.32"` |
| `mysql.banner` | MySQL banner | `"MySQL"` |
| `postgres.version` | PostgreSQL version | `"14.5"` |
| `redis.banner` | Redis banner | `"Redis"` |
| `ftp.banner` | FTP banner | `"220 ProFTPD Server"` |
| `telnet.banner` | Telnet banner | `"Telnet Server"` |
| `snmp.version` | SNMP version | `"SNMPv2c"` |

## Writing a Plugin

### 1. Choose a Category

Place your plugin in the appropriate directory:

- `ssh/` - SSH protocol security
- `http/` - HTTP/HTTPS web security
- `tls/` - TLS/SSL security
- `database/` - Database security
- `network/` - Network service security

### 2. Plugin Template

```yaml
name: "Your Plugin Name"
version: "1.0.0"
type: evaluation
author: "your-github-username"

metadata:
  severity: critical  # critical, high, medium, low, info
  tags: [category, vulnerability-type, protocol]
  cve: "CVE-YYYY-XXXXX"  # If applicable
  cwe: "CWE-XXX"  # If applicable
  cvss_score: 7.5  # If applicable
  cvss_vector: "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H"
  description: "Brief description of what this plugin detects"
  impact: "What could happen if this vulnerability is exploited"
  references:
    - "https://nvd.nist.gov/vuln/detail/CVE-YYYY-XXXXX"
    - "https://www.example.com/security-advisory"

# Trigger when specific data is available
triggers:
  - data_key: "protocol.field"
    condition: exists
    value: true

# Match vulnerable conditions
match:
  logic: OR  # OR, AND
  rules:
    - field: "protocol.field"
      operator: "equals"  # See operators below
      value: "vulnerable-value"
      description: "Why this is vulnerable"

# Output on successful match
output:
  vulnerability: true
  severity: critical
  message: "Clear description of the detected vulnerability"
  remediation: |
    Step-by-step remediation:
    1. Action to fix
    2. Verification step
    3. Additional hardening
  reference: "https://link-to-security-advisory"
```

### 3. Match Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `equals` | Exact string match | `value: "SSLv3"` |
| `contains` | Substring match | `value: "OpenSSH"` |
| `matches` | Regex match | `value: "^OpenSSH_[67]\\."` |
| `startswith` | String prefix | `value: "OpenSSH_"` |
| `endswith` | String suffix | `value: "_Ubuntu"` |
| `in` | Value in list | `value: ["SSLv2", "SSLv3"]` |
| `version_lt` | Version less than | `value: "1.2.3"` |
| `version_lte` | Version ≤ | `value: "1.2.3"` |
| `version_gt` | Version greater than | `value: "2.0.0"` |
| `version_gte` | Version ≥ | `value: "2.0.0"` |
| `version_eq` | Version equals | `value: "1.2.3"` |
| `gt` | Numeric greater than | `value: 100` |
| `lt` | Numeric less than | `value: 50` |
| `gte` | Numeric ≥ | `value: 100` |
| `lte` | Numeric ≤ | `value: 50` |
| `exists` | Field exists | `value: true` |

### 4. Example: SSH Weak Cipher Plugin

```yaml
name: "SSH Weak Encryption Cipher"
version: "1.0.0"
type: evaluation
author: "pentora-security"

metadata:
  severity: high
  tags: [ssh, crypto, weak-crypto, cipher]
  references:
    - "https://www.ssh.com/academy/ssh/encryption"

triggers:
  - data_key: ssh.encryption_algorithms
    condition: exists
    value: true

match:
  logic: OR
  rules:
    - field: ssh.encryption_algorithms
      operator: contains
      value: "3des-cbc"
      description: "3DES cipher (deprecated)"

    - field: ssh.encryption_algorithms
      operator: contains
      value: "aes128-cbc"
      description: "AES CBC mode (vulnerable to padding oracle)"

    - field: ssh.encryption_algorithms
      operator: contains
      value: "aes256-cbc"
      description: "AES CBC mode (vulnerable to padding oracle)"

output:
  vulnerability: true
  severity: high
  message: "SSH server supports weak encryption ciphers (3DES, CBC mode)"
  remediation: |
    Update SSH server configuration to disable weak ciphers:

    1. Edit /etc/ssh/sshd_config:
       Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

    2. Restart SSH service:
       sudo systemctl restart sshd

    3. Verify with:
       ssh -vv user@host 2>&1 | grep "cipher"
  reference: "https://www.ssh.com/academy/ssh/encryption"
```

## Testing Guidelines

### 1. Local Testing

Test your plugin against real targets:

```bash
# Place plugin in local directory
mkdir -p ~/.local/share/pentora/plugins/dev/ssh
cp ssh/my-new-plugin.yaml ~/.local/share/pentora/plugins/dev/ssh/

# Run scan with your plugin
pentora scan --targets test.example.com --vuln --debug

# Check plugin loading in logs
pentora plugin list | grep "My New Plugin"
```

### 2. Test Cases

Create test cases in your PR description:

```markdown
## Test Cases

### Vulnerable Target
- Target: `test.vulnerable-ssh.com:22`
- Expected: Plugin triggers with severity HIGH
- Actual: ✅ Detected vulnerable cipher: 3des-cbc

### Non-Vulnerable Target
- Target: `test.secure-ssh.com:22`
- Expected: No vulnerability detected
- Actual: ✅ No detection (only modern ciphers)

### False Positive Check
- Target: `192.168.1.1:22`
- Expected: No false positives
- Actual: ✅ Plugin did not trigger on secure config
```

### 3. Validation

Pentora automatically validates plugins during loading:

```bash
# Validate plugin syntax
pentora plugin verify
```

Common validation errors:
- Missing required fields (name, version, type, author)
- Invalid trigger format (using `port` instead of `data_key`)
- Missing `output.vulnerability` field
- Invalid severity level
- Empty match rules

## Submission Process

### 1. Prepare Your Submission

```bash
# Create feature branch
git checkout -b add-plugin-name

# Add your plugin
git add ssh/your-plugin.yaml

# Commit with descriptive message
git commit -m "feat(ssh): add detection for SSH vulnerability XYZ

- Detects weak SSH cipher suites (3DES, CBC mode)
- Triggers on ssh.encryption_algorithms data key
- Includes remediation steps and CVE references

Test cases:
- Vulnerable: OpenSSH 7.4 (3des-cbc enabled)
- Non-vulnerable: OpenSSH 8.9 (only modern ciphers)"

# Push to your fork
git push origin add-plugin-name
```

### 2. Create Pull Request

Include in your PR:

- **Description**: What vulnerability this detects
- **Trigger conditions**: When the plugin activates
- **Test results**: Real-world testing against vulnerable/non-vulnerable targets
- **References**: CVE numbers, security advisories, documentation
- **Example output**: Screenshot or text output of detection

### 3. Review Process

Your PR will be reviewed for:

1. **Technical correctness**: Does it detect the vulnerability accurately?
2. **Code quality**: Follows plugin format and best practices
3. **Testing**: Adequate test coverage and validation
4. **Documentation**: Clear descriptions and remediation steps
5. **Security**: No malicious code or dangerous patterns

## Best Practices

### ✅ DO

- Use **data_key triggers** (not port/service)
- Include **CVE/CWE references** when applicable
- Provide **clear remediation steps**
- Use **appropriate severity levels**
- Test against **real-world targets**
- Include **description** fields in match rules
- Use **semantic versioning** (1.0.0, 1.0.1, 1.1.0)
- Add **meaningful tags** for categorization
- Write **clear, actionable messages**

### ❌ DON'T

- Use **port-based triggers** (`port: 22`)
- Create **duplicate plugins** (check existing first)
- Use **overly broad regex** (causes false positives)
- Skip **testing** (untested plugins will be rejected)
- Plagiarize **other security tools** (write original detection logic)
- Include **hardcoded credentials** or secrets
- Create **noisy low-value checks** (use `severity: info` sparingly)

### Severity Guidelines

| Severity | Use When |
|----------|----------|
| `critical` | Remote code execution, authentication bypass, data breach |
| `high` | Privilege escalation, weak crypto, exposed sensitive data |
| `medium` | Information disclosure, missing headers, outdated software |
| `low` | Minor misconfigurations, best practice violations |
| `info` | Informational findings, version detection |

### Trigger Condition Examples

```yaml
# Single data key existence
triggers:
  - data_key: ssh.banner
    condition: exists
    value: true

# Multiple data keys (OR logic)
triggers:
  - data_key: ssh.version
    condition: exists
  - data_key: ssh.banner
    condition: exists

# Specific value requirement
triggers:
  - data_key: http.status_code
    condition: equals
    value: 200
```

## Manifest Generation

After your plugin is merged, the manifest is **automatically regenerated** by GitHub Actions.

Manual regeneration (if needed):

```bash
cd pentora-plugins

# Run standalone manifest generator
cd tools/manifest-generator
go run . \
  -dir ../.. \
  -output ../../manifest.yaml \
  -base-url https://plugins.pentora.ai

# Commit manifest
git add ../../manifest.yaml
git commit -m "chore: regenerate manifest"
git push
```

## Questions?

- **GitHub Issues**: https://github.com/pentora-ai/pentora/issues
- **Discussions**: https://github.com/pentora-ai/pentora/discussions
- **Email**: security@pentora.ai

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.

---

**Thank you for making Pentora better!** 🚀
