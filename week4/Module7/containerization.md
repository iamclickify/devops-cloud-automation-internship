# Container Security Fundamentals

## Overview

Container security is a critical practice spanning the entire container lifecycle. It involves protecting container images, registries, runtime environments, and orchestration platforms from threats while maintaining operational efficiency and compliance.

---

## Table of Contents

1. [Threat Landscape](#threat-landscape)
2. [Shared Responsibility Model](#shared-responsibility-model)
3. [Principle of Least Privilege](#principle-of-least-privilege)
4. [Image Provenance & Trust](#image-provenance--trust)
5. [Container Registry Security](#container-registry-security)
6. [Best Practices](#best-practices)

---

## Threat Landscape

### Common Container Security Vulnerabilities

#### 1. Misconfigurations

**Definition**: Settings that deviate from security best practices, creating unintended security gaps.

**Common Misconfigurations**:
- Insecure Docker daemon settings
- Overly permissive container capabilities
- Misconfigured Kubernetes RBAC
- Exposed ports to public internet
- No resource limits (CPU/memory)
- Running containers with `--privileged` flag
- Unencrypted etcd (Kubernetes state database)

**Consequences**:
- ❌ Unauthorized access to sensitive data
- ❌ Privilege escalation
- ❌ Denial of service (DoS)
- ❌ Lateral movement in network
- ❌ Data breaches

**Mitigation**:
✅ Regular configuration audits  
✅ Automated scanning (Trivy, Aqua Security)  
✅ Infrastructure as Code (IaC) security  
✅ Principle of least privilege  
✅ Network segmentation  
✅ Resource limits enforcement  
✅ Security contexts (Kubernetes)  

**Tools**: CIS Benchmarks, Trivy, Aqua Security, Prisma Cloud

---

#### 2. Vulnerable Dependencies

**Definition**: Software components (libraries, packages, base OS) with known security flaws.

**Sources**:
- Outdated base OS images
- Third-party libraries (npm, PyPI, Maven)
- Application code vulnerabilities
- Transitive dependencies

**Examples**:
- Log4Shell (CVE-2021-44228) in Apache Log4j
- Outdated Alpine Linux packages
- Old Node.js dependencies

**Consequences**:
- ❌ Easy exploitation via public exploits
- ❌ System compromise
- ❌ Data theft
- ❌ Malware injection
- ❌ Reputational damage

**Mitigation**:
✅ Image scanning tools (Trivy, Clair, Snyk)  
✅ CI/CD pipeline integration  
✅ Regular base image updates  
✅ Dependency management tools  
✅ Software Bill of Materials (SBOM)  
✅ Vulnerability tracking  

**Tools**: Trivy, Clair, Anchore, Snyk

---

#### 3. Exposed Secrets

**Definition**: Sensitive credentials in insecure locations (hardcoded, logs, configs).

**Common Exposures**:
- Hardcoded API keys in Dockerfile
- Credentials in environment variables
- Secrets in plain-text config files
- Secrets committed to Git repositories
- Secrets in container logs

**Consequences**:
- ❌ Unauthorized access to systems/data
- ❌ Cloud account compromise
- ❌ Financial theft
- ❌ Compliance violations (GDPR, HIPAA)
- ❌ Reputational damage

**Mitigation**:
✅ Never hardcode secrets  
✅ Use secrets management tools  
✅ Inject secrets at runtime  
✅ Secret scanning in CI/CD  
✅ Least privilege access control  
✅ Secret rotation policies  
✅ Secure logging practices  

**Tools**: 
- HashiCorp Vault
- Kubernetes Secrets
- AWS Secrets Manager
- Azure Key Vault
- GitGuardian, TruffleHog (scanning)

---

## Shared Responsibility Model

### What is the Shared Responsibility Model?

**Definition**: Security is a collaborative effort between cloud provider and customer. Provider secures "the cloud," customer secures "what is in the cloud."

### Responsibility Breakdown

| Component | Cloud Provider | Customer |
|-----------|---|---|
| **Physical Security** | ✅ | |
| **Network Infrastructure** | ✅ | |
| **Hypervisor/Virtualization** | ✅ | |
| **Managed Services (control plane)** | ✅ | |
| **Data Security** | | ✅ |
| **Application Security** | | ✅ |
| **Identity & Access Management** | | ✅ |
| **Operating System/Runtime** | | ✅ |
| **Network Configuration** | | ✅ |
| **Secrets Management** | | ✅ |
| **Container Images** | | ✅ |

### Service Model Differences

| Layer | IaaS | PaaS (Managed K8s) | SaaS |
|-------|------|---------|------|
| **Provider Manages** | Physical, Network, Hypervisor | + Control Plane | + Everything |
| **Customer Manages** | OS, Runtime, Apps | Apps, Config, IAM | Nothing |

### Implementation Steps

1. **Identify your service model** (IaaS, PaaS, SaaS)
2. **Consult provider documentation** for specific responsibilities
3. **Map your container stack** (base image → orchestration)
4. **Assign responsibilities** clearly to provider or organization
5. **Implement security controls** for your responsibilities
6. **Regularly review and update** as stack evolves

### Example: AWS EKS (Managed Kubernetes)

**AWS Provides**:
- EKS control plane security
- EC2 worker node infrastructure
- AWS network security

**You Provide**:
- Container image security (scanning, signing)
- Kubernetes RBAC configuration
- Network Policies
- Secrets management
- Application code security
- IAM role configuration
- Monitoring and logging

---

## Principle of Least Privilege

### What is Least Privilege for Containers?

**Definition**: Grant only the bare minimum privileges necessary for containers to function.

### Application Areas

| Area | Least Privilege Implementation |
|------|-------------------------------|
| **User Privileges** | Run containers as non-root user |
| **Linux Capabilities** | Drop all, add only essential |
| **Host Access** | Minimal volume mounts, no privileged |
| **Network Access** | Restrict to required services |
| **Kubernetes RBAC** | Scoped service accounts, minimal roles |

---

### 1. Running as Non-Root User

**Problem**: Running as root (UID 0) allows attackers full system access if container is compromised.

**Dockerfile Implementation**:
```dockerfile
FROM ubuntu:latest

# Create non-root user
RUN groupadd -r appgroup && useradd -r -g appgroup appuser

WORKDIR /app
COPY --chown=appuser:appgroup . /app

# Switch to non-root user
USER appuser

CMD ["your_application"]
```

**Kubernetes Implementation**:
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
      runAsUser: 1001
      runAsGroup: 1001
      allowPrivilegeEscalation: false
```

---

### 2. Dropping Linux Capabilities

**Problem**: Containers inherit broad capabilities by default.

**Solution**: Drop all capabilities, add back only what's needed.

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
      runAsUser: 1001
      runAsGroup: 1001
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL  # Drop all capabilities
        add:
        - NET_BIND_SERVICE  # Add only if needed
```

**Common Capabilities**:
- `NET_BIND_SERVICE`: Bind to ports < 1024
- `CAP_SETUID`: Change user IDs
- `CAP_SYS_ADMIN`: Various admin operations
- `CAP_NET_ADMIN`: Network admin

---

### 3. Restricting Host Access

**Best Practices**:
✅ Avoid `--privileged` flag  
✅ Read-only root filesystem: `readOnlyRootFilesystem: true`  
✅ Minimal volume mounts  
✅ Network isolation via policies  
✅ No host network access  

```yaml
securityContext:
  readOnlyRootFilesystem: true
  privileged: false
  allowPrivilegeEscalation: false
```

---

### 4. Kubernetes RBAC Least Privilege

```yaml
# Minimal role for deployment
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: minimal-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get"]  # Only read, not write

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: minimal-role
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: default
```

---

## Image Provenance & Trust

### What is Image Provenance?

**Definition**: The origin and history of a container image - who built it, what's included, build process details.

### What is Image Trust?

**Definition**: Confidence that an image's origin is verified, hasn't been tampered with, and doesn't contain malicious code.

### Why It Matters

✅ Prevent supply chain attacks  
✅ Ensure integrity (not modified)  
✅ Detect vulnerabilities  
✅ Meet compliance requirements  
✅ Enable reproducibility  

---

### Ensuring Image Trust

#### 1. Use Trusted Base Images

- Official Docker Hub images (ubuntu, alpine, nginx)
- Cloud provider base images
- Images from reputable vendors
- High download counts = more scrutiny

---

#### 2. Sign Container Images

**Docker Content Trust (Notary)**:
```bash
docker trust sign my-image:latest
```

**Sigstore (cosign)** - Modern approach:
```bash
# Sign image
cosign sign my-image:latest

# Verify signature
cosign verify my-image:latest
```

**Benefits**:
- Cryptographic verification
- Tamper detection
- Supply chain integrity

---

#### 3. Scan Images for Vulnerabilities

```bash
# Scan with Trivy
trivy image my-app:latest

# Fail on HIGH/CRITICAL
trivy image --severity HIGH,CRITICAL my-app:latest
```

**Tools**: Trivy, Clair, Anchore, Snyk

---

#### 4. Private Registries & Access Control

- Docker Hub Private Repos
- AWS ECR
- Azure ACR
- Google GCR
- Harbor (self-hosted)

**Benefits**:
- ✅ Control who pushes/pulls
- ✅ Audit access
- ✅ Image signing enforcement

---

#### 5. Minimal Base Images

- Alpine Linux (5-10 MB vs 100+ MB)
- Distroless images (Google)
- Scratch images (advanced)

**Reduced attack surface** = fewer vulnerabilities

---

#### 6. Reproducible Builds

Same input → Same output (bit-for-bit identical)

**Benefits**:
- Verifiable provenance
- Supply chain integrity
- Easier debugging

---

#### 7. Software Bill of Materials (SBOM)

List all components and versions in image.

```bash
# Generate SBOM with cosign
cosign download sbom my-image:latest
```

**Tools**: Syft, trivy (SBOM generation)

---

## Container Registry Security

### What are Container Registries?

**Definition**: Centralized services for storing, managing, and distributing container images.

**Examples**:
- Public: Docker Hub, Quay.io
- Cloud: ECR, ACR, GCR
- Self-hosted: Harbor, Nexus

### Security Considerations

#### 1. Authentication & Authorization

| Component | Implementation |
|-----------|---|
| **Strong Auth** | Username/password, API tokens, OAuth, SAML |
| **Cloud IAM** | IAM roles for EC2, pods, services |
| **RBAC** | Admin, Pusher, Puller, Auditor roles |

**Example RBAC**:
- **Admin**: Full control
- **Pusher**: Can upload images
- **Puller**: Can download images
- **Auditor**: Read-only access

---

#### 2. Image Scanning

- Scan on push/pull
- Prevent critical images
- Continuous visibility

**Integration**:
```bash
# Build and scan
docker build -t my-app:latest .
trivy image --severity HIGH,CRITICAL my-app:latest
docker push my-registry.com/my-app:latest
```

---

#### 3. Image Signing & Verification

- Sign images in registry
- Enforce signature verification
- Admission controller policies

---

#### 4. Network Security

✅ HTTPS/TLS only  
✅ Limit public exposure  
✅ Network segmentation  
✅ VPC/subnet isolation  

---

#### 5. Auditing & Logging

**Log all**:
- Image pushes/pulls
- Auth attempts (success/fail)
- Configuration changes
- Scan results

---

#### 6. Encryption

- Encrypt at rest
- Encrypt in transit
- Key management

---

#### 7. Updates & Patching

- Keep registry software updated
- Security patches
- Vulnerability fixes

---

## Best Practices

### 1. Secure Dockerfiles & Images

```dockerfile
# ✅ Good example
FROM alpine:latest  # Minimal base

RUN apk add --no-cache python3  # Only essentials

RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app
COPY --chown=appuser:appgroup . /app

USER appuser
RUN chmod 555 /app  # Read-only

CMD ["python3", "app.py"]
```

**Checklist**:
✅ Minimal base image  
✅ Non-root user  
✅ Minimal layers  
✅ No build tools in final image  
✅ Vulnerability scanned  
✅ Image signed  
✅ No hardcoded secrets  
✅ Regular updates  

---

### 2. Runtime & Host Security

✅ Harden Docker daemon  
✅ Security profiles (AppArmor, SELinux, seccomp)  
✅ Least privilege contexts  
✅ Read-only filesystems  
✅ Limited host access  
✅ Patched host OS  

---

### 3. Orchestration Security

✅ Secure API server access  
✅ RBAC with least privilege  
✅ Network Policies enforced  
✅ Secrets encryption  
✅ Pod Security Standards  
✅ Regular cluster updates  

---

### 4. Registry Security

✅ Private registries  
✅ Strong authentication/authorization  
✅ Image scanning enabled  
✅ Image signing enforced  
✅ Comprehensive auditing  

---

### 5. Runtime Monitoring

✅ Runtime threat detection (Falco, Sysdig)  
✅ Centralized logging  
✅ Intrusion detection  
✅ Alert on suspicious activity  

---

## Defense in Depth Strategy

```
Layer 1: Build Time
├── Code scanning
├── Image scanning
├── Secret scanning
└── Image signing

Layer 2: Registry
├── Authentication/Authorization
├── Access control
├── Vulnerability scanning
└── Audit logging

Layer 3: Deployment
├── Pod Security Standards
├── Network Policies
├── RBAC
└── Resource limits

Layer 4: Runtime
├── Threat detection
├── Anomaly detection
├── Monitoring/Logging
└── Incident response
```

---

## Container Security Tools

| Category | Tools |
|----------|-------|
| **Image Scanning** | Trivy, Clair, Snyk, Anchore |
| **Secret Scanning** | GitGuardian, TruffleHog |
| **Image Signing** | cosign, Notary |
| **Runtime Security** | Falco, Aqua Security, Sysdig |
| **Configuration** | CIS Benchmarks, Kubewarden |
| **SBOM** | Syft, cyclonedx |

---

## OWASP Top 10 for Containers (Summary)

| Risk | Category | Mitigation |
|------|----------|-----------|
| **C1** | Broken Access Control | RBAC, least privilege |
| **C2** | Cryptographic Failures | Encrypt secrets, TLS |
| **C3** | Injection | Input validation |
| **C4** | Insecure Design | Threat modeling, PoLP |
| **C5** | Security Misconfiguration | Configuration scanning |
| **C6** | Vulnerable Components | Image scanning, updates |
| **C7** | Auth Failures | Strong auth, MFA |
| **C8** | Integrity Failures | Image signing, verification |
| **C9** | Logging Failures | Centralized logging |
| **C10** | SSRF | Network isolation |

---

