# GitOps Fundamentals

## What is GitOps?

GitOps is a DevOps practice where Git acts as the Single Source of Truth (SSOT) for infrastructure and application deployments.

All infrastructure changes are stored, reviewed, and deployed through Git repositories.

---

## Core Principles of GitOps

### 1. Declarative Configuration

Infrastructure is described by its desired state.

Example:

```yaml
replicas: 3
image: nginx
```

### 2. Git as Single Source of Truth

All configurations are stored in Git.

Benefits:

- Version Control
- Audit Trail
- Collaboration
- Rollback Support

### 3. Automated Reconciliation

GitOps tools automatically synchronize infrastructure with Git.

### 4. Continuous Monitoring

GitOps continuously checks whether the actual infrastructure matches the desired configuration.

---

## Infrastructure Drift

Drift occurs when:

Actual State ≠ Desired State

Example:

Git:

```yaml
replicas: 3
```

Cluster:

```yaml
replicas: 1
```

GitOps tools automatically correct the drift.

---

## Pull-Based Deployment Model

Traditional CI/CD:

```text
GitHub Actions → Kubernetes
```

GitOps:

```text
Git Repository ← GitOps Tool ← Kubernetes
```

The cluster pulls updates from Git instead of receiving pushed deployments.

---

## Benefits of GitOps

- Faster Deployments
- Improved Reliability
- Enhanced Security
- Easy Rollbacks
- Better Collaboration
- Complete Audit Trail

---

## Popular GitOps Tools

### ArgoCD

- Declarative Continuous Delivery
- Kubernetes Native
- Automated Synchronization
- Visual Dashboard

### Flux

- Lightweight GitOps Toolkit
- Kubernetes Native
- Supports Helm and Kustomize

---

## GitOps Workflow

Developer
↓
Git Commit
↓
Pull Request
↓
Code Review
↓
Merge
↓
GitOps Tool (ArgoCD/Flux)
↓
Kubernetes Cluster

---

## Key Takeaways

- GitOps uses Git as the source of truth.
- Infrastructure is managed declaratively.
- Deployments are automated.
- Drift is automatically corrected.
- ArgoCD and Flux are the most popular GitOps tools.
