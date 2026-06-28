# Image Scanning with Trivy

## Overview

Trivy is an open-source vulnerability scanner that identifies security risks in container images, file systems, and Kubernetes configurations. It enables early detection of vulnerabilities through automated scanning, supporting "shift-left security" practices.

---

## Table of Contents

1. [What is Trivy](#what-is-trivy)
2. [Installation](#installation)
3. [Scanning Container Images](#scanning-container-images)
4. [Analyzing Results](#analyzing-results)
5. [Scanning Dependencies](#scanning-dependencies)
6. [Scanning Kubernetes Configs](#scanning-kubernetes-configs)
7. [CI/CD Integration](#cicd-integration)
8. [Mitigation Strategies](#mitigation-strategies)

---

## What is Trivy

### Definition

Open-source vulnerability scanner that detects security issues in:
- Container images (OS packages)
- Application dependencies (npm, pip, Maven, etc.)
- Kubernetes manifests
- Secrets in code
- Infrastructure-as-Code (Terraform, etc.)

### Key Features

✅ **Comprehensive scanning** - OS packages + dependencies  
✅ **Easy to use** - Simple CLI, single binary  
✅ **Fast and accurate** - Optimized for speed  
✅ **Extensive database** - Continuously updated  
✅ **Multiple output formats** - Text, JSON, JUnit XML  
✅ **Misconfiguration detection** - K8s, Dockerfile, Terraform  
✅ **Secret detection** - Hardcoded credentials  

### Why Trivy Matters

| Benefit | Impact |
|---------|--------|
| **Proactive detection** | Identify risks before production |
| **Compliance** | Meet regulatory requirements |
| **Reduced attack surface** | Fix vulnerabilities early |
| **Developer workflow** | Shift-left security |
| **Automation** | CI/CD integration |

---

## Installation

### Prerequisites

- Linux, macOS, or Windows
- Docker Desktop (for scanning images)
- Internet access

### Method 1: Homebrew (macOS/Linux)

```bash
brew install trivy
trivy --version
```

### Method 2: Binary Download (All OS)

```bash
# Download from GitHub releases
# https://github.com/aquasecurity/trivy/releases

# Extract and move to PATH
tar zxvf trivy_0.XX.X_Linux-64bit.tar.gz
sudo mv trivy /usr/local/bin/

# Verify
trivy --version
```

### Method 3: Docker Container

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image nginx:latest
```

### Configuration

**Update vulnerability database**:
```bash
trivy --download-db-only
```

**Set custom cache directory**:
```bash
export TRIVY_CACHE_DIR=/path/to/cache
```

**Ignore vulnerabilities** (.trivyignore file):
```
# .trivyignore in project root
CVE-2023-1234
CVE-2023-5678
```

---

## Scanning Container Images

### Basic Image Scan

```bash
# Scan an image
trivy image nginx:latest

# Scan with severity filter
trivy image --severity HIGH,CRITICAL nginx:latest

# JSON output
trivy image --format json nginx:latest > results.json

# Save to file
trivy image nginx:latest > scan-results.txt
```

### Understanding Output

**Scan header**:
```
nginx:latest (debian 11)

Total: 150 (UNKNOWN: 0, LOW: 50, MEDIUM: 50, HIGH: 30, CRITICAL: 20)
```

**Detailed findings**:
- **Library**: Package name
- **Vulnerability ID**: CVE identifier
- **Severity**: CRITICAL, HIGH, MEDIUM, LOW
- **Installed Version**: Current version
- **Fixed Version**: Version that patches vulnerability

### Example Output

```
┌─────────────┬───────────────────┬──────────────┬─────────────┐
│   Library   │ Vulnerability ID  │   Severity   │   Version   │
├─────────────┼───────────────────┼──────────────┼─────────────┤
│   openssl   │ CVE-2023-1234     │ CRITICAL     │ 1.1.1w-1    │
│   curl      │ CVE-2023-5678     │ HIGH         │ 7.74.0-1.3  │
└─────────────┴───────────────────┴──────────────┴─────────────┘
```

### Output Formats

```bash
# Human-readable table (default)
trivy image nginx:latest

# JSON
trivy image --format json nginx:latest

# SARIF (for GitHub)
trivy image --format sarif nginx:latest

# Cyclone DX (SBOM)
trivy image --format cyclonedx nginx:latest
```

---

## Analyzing Results

### Prioritizing Vulnerabilities

**Focus on CRITICAL and HIGH**:
- Highest risk to systems
- Easier to exploit
- Most impact if compromised

### Key Information to Extract

| Field | Purpose |
|-------|---------|
| **CVE ID** | Research on NVD (nvd.nist.gov) |
| **Affected Package** | What's vulnerable |
| **Severity** | Priority level |
| **Installed vs Fixed** | Update gap |
| **References** | Additional info |

### Analysis Example

**Scenario**: Critical OpenSSL vulnerability

```
Package: openssl
CVE: CVE-2023-1234
Severity: CRITICAL
Installed: 1.1.1w-1
Fixed: 1.1.1w-2
```

**Impact**: OpenSSL is cryptographic library → TLS/SSL affected → data encryption/authentication at risk

**Action**: Update base image or rebuild with `apt-get upgrade openssl`

### Interpreting Severity

| Level | Action | Timeline |
|-------|--------|----------|
| **CRITICAL** | Immediate | Hours |
| **HIGH** | Urgent | Days |
| **MEDIUM** | Schedule | Weeks |
| **LOW** | Monitor | As time permits |

---

## Scanning Dependencies

### Why Scan Dependencies?

- Single vulnerable dependency = entire app at risk
- Attackers actively target known library vulnerabilities
- Dependencies: npm, pip, Maven, Gradle, Go, Bundler, Composer, NuGet

### npm Dependencies

**How Trivy detects npm packages**:
- Inspects `package.json` and `package-lock.json`
- Cross-references against vulnerability database
- Reports npm package vulnerabilities

**Scanning Node.js image**:
```bash
trivy image your-node-app:latest
```

**Example npm vulnerability**:
```
┌──────────────┬────────────────┬──────────┬──────────────┐
│   Package    │ Vulnerability  │ Severity │   Version    │
├──────────────┼────────────────┼──────────┼──────────────┤
│   lodash     │ CVE-2023-XXXX  │ HIGH     │ 4.17.20      │
│   axios      │ CVE-2023-YYYY  │ MEDIUM   │ 0.21.1       │
└──────────────┴────────────────┴──────────┴──────────────┘
```

**Remediation**:
```json
// Update package.json
{
  "dependencies": {
    "lodash": "^4.17.21",  // Updated
    "axios": "^0.21.4"     // Updated
  }
}
```

Then:
```bash
npm install
docker build -t your-node-app:latest .
trivy image your-node-app:latest  # Verify fix
```

### pip Dependencies

**How Trivy detects pip packages**:
- Inspects `requirements.txt`, `Pipfile`, `pyproject.toml`
- Checks installed Python packages
- Reports pip package vulnerabilities

**Scanning Python image**:
```bash
trivy image your-python-app:latest
```

**Example pip vulnerability**:
```
┌──────────────┬────────────────┬──────────┬──────────────┐
│   Package    │ Vulnerability  │ Severity │   Version    │
├──────────────┼────────────────┼──────────┼──────────────┤
│   requests   │ CVE-2023-XXXX  │ HIGH     │ 2.25.1       │
│   numpy      │ CVE-2023-YYYY  │ MEDIUM   │ 1.20.1       │
└──────────────┴────────────────┴──────────┴──────────────┘
```

**Remediation**:
```bash
# Update requirements.txt
requests==2.26.0
numpy==1.21.0

# Rebuild
docker build -t your-python-app:latest .
trivy image your-python-app:latest  # Verify
```

### Best Practices for Dependencies

✅ Use lock files (package-lock.json, requirements.txt)  
✅ Scan regularly in CI/CD  
✅ Keep dependencies updated  
✅ Minimize unnecessary dependencies  
✅ Use specific versions  

---

## Scanning Kubernetes Configs

### Why Scan K8s Manifests?

Misconfigurations lead to:
- ❌ Exposed secrets
- ❌ Excessive privileges
- ❌ Network vulnerabilities
- ❌ Resource exhaustion
- ❌ Compliance violations

### Scanning YAML Files

```bash
# Single file
trivy config deployment.yaml

# Directory
trivy config k8s/

# All YAML files
trivy config .
```

### Common Misconfigurations

| Misconfiguration | Issue | Fix |
|-----------------|-------|-----|
| `allowPrivilegeEscalation: true` | Process can gain more privileges | Set to `false` |
| `runAsNonRoot: false` | Container runs as root | Set to `true` |
| No resource limits | Pod exhausts resources | Set `requests` and `limits` |
| `privileged: true` | Unrestricted container access | Remove or set to `false` |
| No security context | Missing security controls | Add security context |

### Example: Vulnerable Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: nginx:latest
        # ❌ Issues detected by Trivy:
        # - allowPrivilegeEscalation should be false
        # - runAsNonRoot should be true
        # - No resource limits
        # - readOnlyRootFilesystem not set
```

### Secure Deployment (Fixed)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: app
        image: nginx:latest
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          capabilities:
            drop:
            - ALL
          readOnlyRootFilesystem: true
```

### Scan Output

```
deployment.yaml (Deployment/my-app)

High    [High] allowPrivilegeEscalation should be false
Medium  [Medium] Resource limits not set
High    [High] runAsNonRoot should be true
```

---

## CI/CD Integration

### Why Integrate Trivy?

✅ **Early detection** - Catch issues before production  
✅ **Consistency** - Same scanning rules everywhere  
✅ **Automation** - No manual scanning needed  
✅ **Compliance** - Prove security checks happened  
✅ **Efficiency** - Faster feedback loop  

### GitHub Actions Workflow

```yaml
name: Security Scan with Trivy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: docker/setup-buildx-action@v3
      
      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: myapp:latest
          load: true  # Load into Docker daemon
      
      - name: Scan with Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'HIGH,CRITICAL'
          exit-code: '1'  # Fail if vulns found
      
      - name: Upload SARIF report
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

### Workflow Parameters

| Parameter | Purpose | Example |
|-----------|---------|---------|
| `image-ref` | Image to scan | `myapp:latest` |
| `format` | Output format | `sarif`, `json`, `table` |
| `severity` | Filter by severity | `HIGH,CRITICAL` |
| `exit-code` | Fail on findings | `1` |
| `ignore-unfixed` | Ignore no-fix vulns | `true` |

### Failing Builds

```yaml
- name: Scan with Trivy
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:latest'
    severity: 'CRITICAL'
    exit-code: '1'  # Fail workflow on CRITICAL
```

### Advanced Integration

```yaml
- name: Scan K8s manifests
  run: trivy config k8s/ --severity HIGH,CRITICAL --exit-code 1

- name: Generate SBOM
  run: trivy image --format cyclonedx myapp:latest > sbom.json

- name: Check secrets
  run: trivy image --scanners secret myapp:latest
```

---

## Mitigation Strategies

### 1. Secure Base Images

✅ Use minimal images (Alpine, Distroless)  
✅ Choose trusted sources (official repositories)  
✅ Regularly update base images  
✅ Scan base images before use  

### 2. Secure Dockerfile Practices

✅ Run as non-root user  
✅ Minimize attack surface  
✅ Use specific version tags  
✅ Multi-stage builds  
✅ Clean up package caches  

### 3. Dependency Management

✅ Regular scanning in CI  
✅ Automated updates (Dependabot)  
✅ Follow vulnerability feeds  
✅ Generate SBOMs  

### 4. Runtime Security

✅ Least privilege (RBAC, capabilities)  
✅ Network policies  
✅ Secrets management  
✅ Runtime threat detection  
✅ Logging and auditing  

### 5. Image Signing

✅ Sign approved images (Cosign, Notary)  
✅ Verify signatures at deployment  
✅ Admission controllers  

### 6. Policy Enforcement

✅ Security gates in CI/CD  
✅ Admission controllers (OPA, Kyverno)  
✅ Regular audits  
✅ Document exceptions  

---

## Quick Reference

### Common Commands

```bash
# Scan image
trivy image nginx:latest

# High/Critical only
trivy image --severity HIGH,CRITICAL nginx:latest

# JSON output
trivy image --format json nginx:latest > results.json

# Scan directory
trivy config k8s/

# Scan for secrets
trivy image --scanners secret myapp:latest

# Generate SBOM
trivy image --format cyclonedx myapp:latest

# Fail on findings
trivy image --exit-code 1 nginx:latest
```

### Priority Matrix

| Severity | Response | Timeline |
|----------|----------|----------|
| CRITICAL | Immediate remediation | < 24 hours |
| HIGH | Urgent patch | 2-3 days |
| MEDIUM | Schedule update | 1-2 weeks |
| LOW | Track for updates | 1 month+ |

### Remediation Checklist

- [ ] Identify affected package/library
- [ ] Determine fixed version
- [ ] Update Dockerfile/requirements
- [ ] Rebuild image
- [ ] Run Trivy scan again
- [ ] Verify vulnerability gone
- [ ] Deploy or merge
- [ ] Monitor production

---

## Best Practices

✅ Scan early, scan often  
✅ Prioritize CRITICAL/HIGH first  
✅ Pin all image/dependency versions  
✅ Automate scans in CI/CD  
✅ Keep Trivy and database updated  
✅ Use `.trivyignore` sparingly  
✅ Document all exemptions  
✅ Combine with other tools (defense-in-depth)  
✅ Scan Kubernetes configs  
✅ Generate and maintain SBOMs  

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Image not found" | Ensure Docker running, pull image first |
| Slow scan | First run downloads DB, subsequent faster |
| No vulns found | Image may be secure, try older image |
| False positives | Use `.trivyignore` after verification |
| Database outdated | Run `trivy --download-db-only` |

---

## Summary

**Trivy** provides comprehensive vulnerability scanning for:
- Container images (OS + dependencies)
- Kubernetes configurations
- Secrets in code
- Infrastructure-as-Code

**Integration with CI/CD** enables:
- Early detection (shift-left)
- Automated enforcement
- Consistent scanning
- Compliance verification

**Multi-layered approach**:
- Scan images
- Scan dependencies
- Scan configs
- Enforce policies
- Monitor runtime

---
