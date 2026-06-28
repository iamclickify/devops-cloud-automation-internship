# Secure Dockerfile Best Practices

## Overview

Secure Dockerfiles are the foundation of container security. By implementing best practices during image creation, you minimize attack surface, reduce vulnerabilities, and ensure applications are built on a secure foundation.

---

## Table of Contents

1. [Minimal Base Images](#minimal-base-images)
2. [Avoiding Unnecessary Packages](#avoiding-unnecessary-packages)
3. [Least Privilege: Non-Root Users](#least-privilege-non-root-users)
4. [Secrets Management](#secrets-management)
5. [Layer Caching Security](#layer-caching-security)
6. [Regular Updates](#regular-updates)
7. [Best Practices Summary](#best-practices-summary)

---

## Minimal Base Images

### What are Minimal Base Images?

**Definition**: Stripped-down operating system distributions or containerization-specific images containing only essential packages and libraries.

### Common Minimal Base Images

| Image | Size | Use Case | Features |
|-------|------|----------|----------|
| **Alpine Linux** | <5MB | General purpose | Security-focused, musl libc |
| **Distroless** | Varies | Language-specific | No package manager, no shell |
| **Scratch** | 0MB | Static binaries | Completely empty image |
| **Slim variants** | ~100MB | Language-specific | Reduced from full images |

### Why Minimal Matters

✅ **Reduced attack surface**: Fewer packages = fewer vulnerabilities  
✅ **Smaller image size**: Faster pulls, quicker deployments  
✅ **Improved performance**: Less overhead  
✅ **Easier auditing**: Fewer components to track  
✅ **Faster patching**: Quicker to rebuild and redeploy  

### Implementation

**Alpine Linux Example**:
```dockerfile
FROM alpine:3.19  # Specific version, not 'latest'
RUN apk add --no-cache python3
```

**Distroless Example** (Go application):
```dockerfile
# Stage 1: Build
FROM golang:1.22-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: Runtime (distroless)
FROM gcr.io/distroless/static-debian11
COPY --from=builder /app/myapp /myapp
ENTRYPOINT ["/myapp"]
```

**Scratch Example** (static binary):
```dockerfile
FROM scratch
COPY myapp /myapp
ENTRYPOINT ["/myapp"]
```

### Best Practices

✅ Pin to specific version (e.g., `alpine:3.19`)  
✅ Avoid `latest` tag (non-reproducible)  
✅ Test with chosen base image (musl vs glibc)  
✅ Understand base image internals  

---

## Avoiding Unnecessary Packages

### What are Unnecessary Components?

- **Development tools**: gcc, make, git, build-essential
- **Package managers**: apt, yum, apk (if not needed post-deployment)
- **Shells & utilities**: bash, curl, wget, vim
- **Unused libraries**: Dependencies not in runtime use
- **Background services**: Daemons not part of core app

### Why Remove Them?

✅ Minimize attack surface  
✅ Reduce vulnerability exposure  
✅ Improve image integrity  
✅ Enhance performance  
✅ Reduce disk footprint  

### Multi-Stage Builds (Best Approach)

```dockerfile
# Stage 1: Builder with all dev tools
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Runtime with only essentials
FROM node:20-alpine

WORKDIR /app

# Copy ONLY built artifacts and runtime deps
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

CMD ["node", "dist/index.js"]
```

### Cleanup Best Practices

**Alpine cleanup**:
```dockerfile
RUN apk update && \
    apk add --no-cache some-package && \
    rm -rf /var/cache/apk/*
```

**Debian/Ubuntu cleanup**:
```dockerfile
RUN apt-get update && \
    apt-get install -y some-package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Python example**:
```dockerfile
# Stage 1: Build
FROM python:3.11-slim AS builder
RUN pip install --upgrade pip setuptools wheel
WORKDIR /app
COPY requirements.txt .
RUN pip wheel --no-cache-dir --wheel-dir /wheels -r requirements.txt

# Stage 2: Runtime
FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /wheels /wheels
COPY requirements.txt .
RUN pip install --no-cache-dir --no-index --find-links=/wheels -r requirements.txt && \
    rm -rf /wheels && \
    rm -rf /root/.cache/pip
```

### Common Mistakes

❌ Forgetting to clean package manager caches  
❌ Installing git/VCS tools in production images  
❌ Not leveraging multi-stage builds  
❌ Installing full JDK when JRE suffices  

---

## Least Privilege: Non-Root Users

### Why Non-Root is Critical

**Default behavior**: Process runs as root (UID 0) = unrestricted container access

**If compromised**:
- ❌ Attacker gains root privileges
- ❌ Can modify system files
- ❌ Can install malicious software
- ❌ Can escape to host (via kernel exploits)
- ❌ Can interfere with other processes

**Running non-root**:
- ✅ Limits blast radius
- ✅ Prevents accidental damage
- ✅ Mitigates kernel exploits
- ✅ Aligns with best practices

### Implementation: Create Non-Root User

```dockerfile
FROM alpine:3.19

# Arguments for user/group (best practice)
ARG USER=appuser
ARG GROUP=appgroup
ARG UID=1001
ARG GID=1001

# Create group and user
RUN addgroup -g ${GID} ${GROUP} && \
    adduser -u ${UID} -G ${GROUP} -s /bin/false ${USER}

WORKDIR /app

# Copy files and set ownership
COPY --chown=${USER}:${GROUP} . /app

# Switch to non-root user
USER ${USER}

CMD ["your_application"]
```

### Kubernetes Security Context

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  containers:
  - name: my-app-container
    image: your-image:latest
    securityContext:
      runAsUser: 1001              # Non-root UID
      runAsGroup: 1001             # Non-root GID
      runAsNonRoot: true           # Enforce non-root
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true # Read-only FS
      capabilities:
        drop:
        - ALL                       # Drop all capabilities
        add:
        - NET_BIND_SERVICE         # Add only if needed
```

### Important Considerations

**File Permissions**: Ensure app can write to needed directories
```dockerfile
RUN mkdir -p /app/logs && \
    chown -R ${USER}:${GROUP} /app/logs
```

**Port Binding**: Non-root cannot bind to ports < 1024
- Solution: Run on port >1024 (e.g., 8080)
- Use Docker mapping: `-p 80:8080`

**Home Directory**: Some apps need home dir
```dockerfile
RUN useradd -u ${UID} -G ${GROUP} -m -s /bin/false ${USER}
```

---

## Secrets Management

### What Are Secrets?

- API keys
- Database credentials
- Private keys (SSH, TLS)
- OAuth/JWT tokens
- Sensitive configuration

### ❌ NEVER DO THIS

```dockerfile
# EXTREMELY INSECURE!
ENV API_KEY="my-secret-key-12345"
ENV DB_PASSWORD="superSecret123"
```

**Why**:
- Visible in image layers
- Extractable from registry
- In version control history
- Cannot be easily rotated
- Cannot be revoked independently

### ✅ Best Approaches

#### 1. Runtime Environment Variables

```bash
docker run -d \
  -e DB_PASSWORD='my-secret' \
  -e API_KEY='key123' \
  myapp-image
```

**Note**: Can be inspected via `docker inspect`

#### 2. Docker Secrets (Swarm/K8s)

```yaml
version: '3.8'

services:
  myapp:
    image: myapp-image
    secrets:
      - db_password
      - api_key
    environment:
      DB_PASSWORD_FILE: /run/secrets/db_password
      API_KEY_FILE: /run/secrets/api_key

secrets:
  db_password:
    file: ./db_password.txt
  api_key:
    file: ./api_key.txt
```

Application reads from `/run/secrets/`:
```python
with open('/run/secrets/db_password', 'r') as f:
    password = f.read().strip()
```

#### 3. External Secret Management

**HashiCorp Vault**:
```python
import hvac

client = hvac.Client(url='http://vault:8200', token='mytoken')
secret = client.secrets.kv.read_secret_version(path='my-app/db')
password = secret['data']['data']['password']
```

**Cloud Providers**:
- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager

#### 4. Multi-Stage Secret Handling

```dockerfile
# Stage 1: Decrypt and use secrets
FROM ubuntu:latest AS decrypter
RUN apt-get update && apt-get install -y sops

WORKDIR /app
COPY secrets.yaml.enc ./
RUN sops --decrypt secrets.yaml.enc > secrets.yaml
# Use secrets.yaml here for building...

# Stage 2: Runtime without secrets
FROM ubuntu:latest
COPY --from=decrypter /app/built-artifact /app/
# Secrets are NOT copied to final image
CMD ["run_app"]
```

### Best Practices

✅ Never hardcode secrets  
✅ Use Docker secrets for Swarm/K8s  
✅ Use external vault for complex scenarios  
✅ Inject at runtime only  
✅ Don't copy secrets to final layer  
✅ Rotate secrets regularly  

---

## Layer Caching Security

### Understanding Docker Layer Caching

**How it works**: Docker reuses layers if instruction and context haven't changed

**Problem**: Stale/vulnerable components persist in cache

### Security Implications

❌ Vulnerable packages in cached layers  
❌ Sensitive data accidentally included  
❌ Security updates bypassed  
❌ Cache poisoning risks  

### Management Strategies

#### 1. Strategic Instruction Ordering

```dockerfile
FROM alpine:latest

# Stable dependencies (front)
RUN apk update && apk add --no-cache python3

# Less stable (middle)
COPY requirements.txt /app/
RUN pip install -r requirements.txt

# Most volatile (back)
COPY . /app/
CMD ["python", "app.py"]
```

**Benefit**: Code changes don't invalidate dependency cache

#### 2. Cache Busting (When Needed)

```dockerfile
# Force rebuild of this layer
ARG CACHEBUST=1
RUN apk update && apk add --no-cache some-package
```

Build with:
```bash
docker build --build-arg CACHEBUST=$(date +%s) -t myimage .
```

**Warning**: Invalidates cache for all subsequent layers

#### 3. Cleanup Sensitive Data

```dockerfile
# Encrypt, use, decrypt ALL in one RUN
RUN apt-get install -y sops && \
    sops --decrypt secrets.enc > secrets.yaml && \
    # Use secrets here... && \
    rm secrets.yaml && \
    apt-get remove -y sops && \
    rm -rf /var/lib/apt/lists/*
```

#### 4. Prune Build Cache

```bash
# Remove all dangling cache
docker builder prune -a
```

#### 5. Pin Specific Versions

```dockerfile
# Good: Specific version
FROM alpine:3.19
FROM python:3.11.5-slim

# Avoid: Latest tag
FROM ubuntu:latest
```

---

## Regular Updates

### Why Updates Matter

**Vulnerabilities discovered daily in**:
- Base OS packages
- Language runtimes
- Application dependencies
- Third-party software

### Implementation

#### 1. Automated Scheduled Rebuilds

```yaml
name: Weekly Rebuild and Scan

on:
  schedule:
    - cron: '0 3 * * 0'  # Every Sunday at 3 AM UTC
  workflow_dispatch

jobs:
  rebuild:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: myregistry/app:latest-${{ github.run_number }}
      
      - uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myregistry/app:latest-${{ github.run_number }}'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
```

#### 2. Dependency Management

```bash
# Check for vulnerabilities
npm audit
pip audit
cargo audit

# Update dependencies
npm update
pip list --outdated
cargo update
```

#### 3. Version Pinning

```dockerfile
# Pin everything
FROM alpine:3.19
RUN apk add --no-cache python3=3.11.7-r0
```

#### 4. Vulnerability Scanning Integration

```bash
# Scan after build
trivy image --severity HIGH,CRITICAL myapp:latest

# In CI/CD (fail on high)
trivy image --exit-code 1 --severity HIGH,CRITICAL myapp:latest
```

### Best Practices

✅ Rebuild images weekly  
✅ Scan automatically  
✅ Pin all versions  
✅ Test before deploying  
✅ Monitor security feeds  
✅ Automate updates  

---

## Best Practices Summary

### Dockerfile Checklist

#### Build Time
- [ ] Minimal base image (Alpine, Distroless)
- [ ] Pinned version (no `latest`)
- [ ] Multi-stage builds
- [ ] Non-root user created
- [ ] No hardcoded secrets
- [ ] Clean up package caches
- [ ] Remove build tools from final layer

#### Runtime
- [ ] USER instruction set
- [ ] securityContext configured
- [ ] Read-only root filesystem capable
- [ ] Secrets injected at runtime
- [ ] Proper file permissions
- [ ] Health checks (if applicable)

#### Security
- [ ] No CAP_SYS_ADMIN
- [ ] Drop all capabilities by default
- [ ] No privileged mode
- [ ] Limited volume mounts
- [ ] Proper logging (no secrets)

### Example Secure Dockerfile

```dockerfile
# Minimal base image with pinned version
FROM python:3.11.5-slim

# Build arguments for user
ARG USER=appuser
ARG GROUP=appgroup
ARG UID=1001
ARG GID=1001

# Create non-root user
RUN groupadd -g ${GID} ${GROUP} && \
    useradd -u ${UID} -G ${GROUP} -m -s /bin/false ${USER}

WORKDIR /app

# Copy and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt && \
    rm -rf /root/.cache/pip

# Copy application code with proper ownership
COPY --chown=${USER}:${GROUP} . /app

# Switch to non-root user
USER ${USER}

# Health check
HEALTHCHECK --interval=30s --timeout=3s CMD python -c "import socket; socket.create_connection(('localhost', 5000), timeout=1)"

# Expose port
EXPOSE 5000

# Run application (no shell)
CMD ["python", "-u", "app.py"]
```

### Security Scanning Tools

| Tool | Purpose |
|------|---------|
| **Trivy** | Image vulnerability scanning |
| **Hadolint** | Dockerfile linting |
| **Snyk** | Dependency vulnerability scanning |
| **Clair** | Image layer scanning |
| **GitGuardian** | Secret scanning |

```

## Quick Reference: Dockerfile Commands

```dockerfile
FROM image:tag              # Base image
RUN command                 # Execute command
COPY source dest           # Copy files
ADD source dest            # Copy files (with URL support)
WORKDIR path               # Set working directory
ENV KEY=value              # Environment variable
ARG KEY=default            # Build argument
USER username              # Set user
EXPOSE port                # Document port
HEALTHCHECK                # Health check
ENTRYPOINT ["cmd"]         # Main executable
CMD ["arg"]                # Default arguments
VOLUME ["/path"]           # Mount point
LABEL key=value            # Metadata

```
