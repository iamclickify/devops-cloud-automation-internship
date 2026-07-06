# Kubernetes Fundamentals

## Overview

Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications across clusters of machines at scale.

---

## Table of Contents

1. [What is Kubernetes](#what-is-kubernetes)
2. [Architecture](#architecture)
3. [Core Objects](#core-objects)
4. [Pods](#pods)
5. [Controllers](#controllers)
6. [Services & Networking](#services--networking)
7. [Storage](#storage)
8. [ConfigMaps & Secrets](#configmaps--secrets)
9. [Namespaces & RBAC](#namespaces--rbac)
10. [Deployments](#deployments)
11. [Best Practices](#best-practices)

---

## What is Kubernetes

### Definition

**Kubernetes** is a container orchestration platform that manages containerized applications across a cluster of machines, automating deployment, scaling, and operational tasks.

### Why Kubernetes?

| Challenge | Solution |
|-----------|----------|
| **Scaling containers** | Automatic scaling based on demand |
| **High availability** | Self-healing, redundancy |
| **Resource management** | Optimal resource utilization |
| **Rolling updates** | Zero-downtime deployments |
| **Service discovery** | Automatic DNS, load balancing |
| **Storage** | Persistent volume management |
| **Configuration** | Centralized config/secrets |

### Key Features

✅ **Automated deployment** - Deploy containers where needed  
✅ **Self-healing** - Restart failed containers, replace dead nodes  
✅ **Horizontal scaling** - Scale up/down based on metrics  
✅ **Rolling updates** - Update without downtime  
✅ **Service discovery** - DNS-based service location  
✅ **Load balancing** - Automatic load distribution  
✅ **Storage orchestration** - Mount storage systems  
✅ **Resource management** - CPU/memory constraints  

### Use Cases

✅ Microservices architectures  
✅ Multi-cloud deployments  
✅ High-availability applications  
✅ Stateless applications  
✅ Batch jobs  
✅ CI/CD pipelines  

---

## Architecture

### Cluster Components

```
┌─────────────────────────────────────────┐
│         Kubernetes Cluster              │
│                                         │
│  ┌──────────────────────────────────┐  │
│  │    Control Plane (Master)        │  │
│  │                                  │  │
│  │  • API Server                    │  │
│  │  • Scheduler                     │  │
│  │  • Controller Manager            │  │
│  │  • etcd (State Store)            │  │
│  └──────────────────────────────────┘  │
│                                         │
│  ┌──────────┬──────────┬──────────┐   │
│  │  Worker  │  Worker  │  Worker  │   │
│  │  Node 1  │  Node 2  │  Node 3  │   │
│  │          │          │          │   │
│  │ kubelet  │ kubelet  │ kubelet  │   │
│  │ kube-   │ kube-   │ kube-   │   │
│  │ proxy   │ proxy   │ proxy   │   │
│  │          │          │          │   │
│  │ Pods     │ Pods     │ Pods     │   │
│  └──────────┴──────────┴──────────┘   │
└─────────────────────────────────────────┘
```

### Control Plane Components

| Component | Purpose |
|-----------|---------|
| **API Server** | Central hub, handles all API requests |
| **Scheduler** | Assigns pods to nodes |
| **Controller Manager** | Runs controller processes |
| **etcd** | Key-value store for cluster state |
| **Cloud Controller Manager** | Integrates with cloud providers |

### Worker Node Components

| Component | Purpose |
|-----------|---------|
| **kubelet** | Agent running on every node |
| **kube-proxy** | Network proxy, load balancing |
| **Container Runtime** | Docker, containerd, CRI-O |

### Control Plane vs Worker Nodes

```
┌─────────────────────────────────────────┐
│    Control Plane (Single or HA)         │
│  • API Server                           │
│  • Scheduler                            │
│  • Controller Manager                   │
│  • etcd (state database)                │
│                                         │
│  Responsibilities:                      │
│  • Cluster decisions                    │
│  • State management                     │
│  • Deployment scheduling                │
└─────────────────────────────────────────┘
           ↓ (API Requests)
┌─────────────────────────────────────────┐
│    Worker Nodes (Many)                  │
│  • kubelet                              │
│  • kube-proxy                           │
│  • Container runtime                    │
│  • Running pods                         │
│                                         │
│  Responsibilities:                      │
│  • Execute containers                   │
│  • Report node health                   │
│  • Handle networking                    │
└─────────────────────────────────────────┘
```

---

## Core Objects

### Kubernetes Objects

**Objects** are persistent entities in Kubernetes system, representing cluster state

### Object Anatomy

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: default
  labels:
    app: myapp
    version: v1
spec:
  # Object specification (desired state)
  containers:
  - name: mycontainer
    image: myimage:latest
status:
  # Current state (filled by Kubernetes)
  phase: Running
  conditions:
  - type: Ready
    status: "True"
```

### Key Fields

| Field | Purpose |
|-------|---------|
| **apiVersion** | API version (v1, apps/v1, etc.) |
| **kind** | Object type (Pod, Deployment, etc.) |
| **metadata** | Object metadata (name, labels, namespace) |
| **spec** | Desired state |
| **status** | Current state (read-only) |

### Common Objects

| Object | Purpose |
|--------|---------|
| **Pod** | Smallest deployable unit |
| **Deployment** | Manage pod replicas |
| **Service** | Network access to pods |
| **ConfigMap** | Configuration data |
| **Secret** | Sensitive data |
| **Namespace** | Virtual cluster isolation |
| **PersistentVolume** | Storage resource |
| **Ingress** | HTTP/HTTPS routing |

---

## Pods

### What is a Pod?

**Pod** is the smallest deployable Kubernetes object - wraps one or more containers

**Characteristics**:
- Shared network namespace (single IP)
- Shared storage (volumes)
- Container specifications
- Restart policy
- Resource requests/limits

### Single-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

### Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-app
spec:
  containers:
  # Main application
  - name: app
    image: myapp:latest
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: shared-data
      mountPath: /var/shared
  
  # Sidecar: logging
  - name: logging
    image: fluent-bit:latest
    volumeMounts:
    - name: shared-data
      mountPath: /var/shared
  
  # Shared storage
  volumes:
  - name: shared-data
    emptyDir: {}
```

### Pod Lifecycle

```
Pending → Running → Succeeded/Failed/Unknown

Pending: Pod accepted, waiting for resources
Running: Container(s) executing
Succeeded: All containers exited successfully
Failed: At least one container failed
Unknown: Pod state unknown
```

### Pod Probe Types

**Liveness Probe** - Is container alive?
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
```

**Readiness Probe** - Can receive traffic?
```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

**Startup Probe** - Has app started?
```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  initialDelaySeconds: 0
  periodSeconds: 10
```

---

## Controllers

### What are Controllers?

**Controllers** are control loops that manage Kubernetes objects, ensuring current state matches desired state

### Deployment

**Purpose**: Manage pod replicas, rolling updates, rollbacks

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # 1 extra pod during update
      maxUnavailable: 0  # 0 pods unavailable
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### StatefulSet

**Purpose**: Manage stateful applications (databases, caches)

**Characteristics**:
- Stable network identity
- Ordered scaling
- Persistent storage
- Graceful scaling

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  replicas: 3
  serviceName: mysql-service
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast-ssd"
      resources:
        requests:
          storage: 10Gi
```

### DaemonSet

**Purpose**: Run pod on every (or selected) node

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
```

### Job & CronJob

**Job**: Run to completion once
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-job
spec:
  template:
    spec:
      containers:
      - name: backup
        image: backup-tool:latest
        command: ["./backup.sh"]
      restartPolicy: Never
  backoffLimit: 3
```

**CronJob**: Run on schedule
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
          restartPolicy: OnFailure
```

---

## Services & Networking

### What is a Service?

**Service** exposes pods to network traffic, providing stable endpoint

### Service Types

| Type | Purpose | Use Case |
|------|---------|----------|
| **ClusterIP** | Internal cluster access | Inter-pod communication |
| **NodePort** | External access via node port | Development, testing |
| **LoadBalancer** | External load balancer | Production, public apps |
| **ExternalName** | DNS mapping | External services |

### ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80           # Service port
    targetPort: 8080   # Pod port
```

**Access**: `http://web-service:80` within cluster

### NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080    # External port (30000-32767)
```

**Access**: `http://node-ip:30080` from outside

### LoadBalancer Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
```

**Access**: Cloud provider assigns external IP

### Ingress

**Purpose**: HTTP/HTTPS routing rules

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

### Network Policies

**Control pod-to-pod communication**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: client
    ports:
    - protocol: TCP
      port: 80
```

---

## Storage

### PersistentVolume (PV)

**Cluster-level storage resource**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-aws
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  awsElasticBlockStore:
    volumeID: vol-1234567890abcdef0
    fsType: ext4
```

### PersistentVolumeClaim (PVC)

**Request for storage**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

### Using PVC in Pods

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-storage
spec:
  containers:
  - name: app
    image: myapp:latest
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-claim
```

### Storage Classes

**Dynamic provisioning template**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
allowVolumeExpansion: true
```

---

## ConfigMaps & Secrets

### ConfigMap

**Store non-sensitive configuration**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    database.host=db.example.com
    database.port=5432
  log_level: "INFO"
```

**Using in Pod**:
```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: log_level
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config
```

### Secret

**Store sensitive data (encrypted at rest)**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: dXNlcm5hbWU=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

**Using in Pod**:
```yaml
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

### Secret Types

| Type | Purpose |
|------|---------|
| **Opaque** | General base64 data |
| **kubernetes.io/service-account-token** | Service account token |
| **kubernetes.io/dockercfg** | Docker config |
| **kubernetes.io/basicauth** | Basic auth credentials |
| **kubernetes.io/tls** | TLS certificate |

---

## Namespaces & RBAC

### Namespaces

**Virtual cluster isolation**

```bash
# Create namespace
kubectl create namespace production

# List namespaces
kubectl get namespaces

# Use namespace
kubectl get pods -n production
kubectl apply -f deploy.yaml -n production

# Set default namespace
kubectl config set-context --current --namespace=production
```

**In YAML**:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  namespace: production  # Specify namespace
spec:
  containers:
  - name: app
    image: myapp:latest
```

### Role-Based Access Control (RBAC)

**Principles**:
- ServiceAccount
- Role (permissions)
- RoleBinding (connect account to role)

**Example RBAC**:
```yaml
# ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: default

---
# Role with permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
spec:
  rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]

---
# RoleBinding connects ServiceAccount to Role
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: ServiceAccount
  name: app-sa
```

---

## Deployments

### Deployment Strategy

**Rolling Update**:
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Extra pods during update
      maxUnavailable: 0  # Minimum pods available
```

**Blue-Green**:
```yaml
# Deploy new version (green)
# Test it
# Switch traffic
# Remove old version (blue)
```

**Canary**:
```yaml
# Deploy to small subset
# Monitor metrics
# Gradually increase traffic
# Full rollout on success
```

### Common Deployment Commands

```bash
# View deployment
kubectl get deployments
kubectl describe deployment nginx-deployment

# Scale deployment
kubectl scale deployment nginx-deployment --replicas=5

# Update image
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Rollout history
kubectl rollout history deployment/nginx-deployment

# Rollback to previous
kubectl rollout undo deployment/nginx-deployment

# Pause/resume rollout
kubectl rollout pause deployment/nginx-deployment
kubectl rollout resume deployment/nginx-deployment
```

---

## Best Practices

### Security

✅ **Use non-root containers** - Run with least privilege  
✅ **Image scanning** - Check for vulnerabilities  
✅ **Network policies** - Restrict pod communication  
✅ **RBAC** - Implement least privilege access  
✅ **Secrets management** - Use external vaults  
✅ **Pod security policies** - Enforce standards  

### Resource Management

✅ **Set resource requests** - CPU, memory guarantees  
✅ **Set resource limits** - CPU, memory caps  
✅ **Use namespaces** - Logical isolation  
✅ **Implement quotas** - Limit namespace resources  
✅ **Use HPA** - Horizontal Pod Autoscaling  

### Deployment

✅ **Health checks** - Liveness, readiness, startup  
✅ **Graceful shutdown** - Handle SIGTERM  
✅ **Rolling updates** - Zero-downtime deploys  
✅ **Version everything** - Container images, manifests  
✅ **Use GitOps** - Store K8s configs in Git  

### Monitoring & Logging

✅ **Prometheus** - Metrics collection  
✅ **Grafana** - Metrics visualization  
✅ **ELK/Loki** - Log aggregation  
✅ **Alerts** - Proactive notifications  
✅ **Resource monitoring** - Track utilization  

### Organization

✅ **Clear naming** - Descriptive resource names  
✅ **Labels & annotations** - Organize resources  
✅ **Documentation** - README for deployments  
✅ **Kustomize/Helm** - Template management  
✅ **Environment parity** - Dev/staging/prod consistency  

---

## Quick Reference

### Essential Commands

```bash
# Context & cluster
kubectl cluster-info
kubectl config current-context
kubectl config use-context cluster-name

# Namespaces
kubectl get namespaces
kubectl create namespace prod
kubectl get pods -n prod

# Deployments
kubectl get deployments
kubectl apply -f deployment.yaml
kubectl rollout status deployment/myapp
kubectl rollout undo deployment/myapp

# Pods
kubectl get pods
kubectl describe pod pod-name
kubectl logs pod-name
kubectl exec -it pod-name -- /bin/sh

# Services
kubectl get svc
kubectl port-forward svc/my-service 8080:80

# Scaling
kubectl scale deployment myapp --replicas=5
```

### YAML Templates

```yaml
# Deployment template
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"

---
# Service template
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

---

## Summary

**Kubernetes** provides:

✅ Container orchestration at scale  
✅ Automated deployment & scaling  
✅ Self-healing infrastructure  
✅ Rolling updates & rollbacks  
✅ Network abstraction (Services)  
✅ Storage management (Volumes)  
✅ Configuration management (ConfigMaps, Secrets)  
✅ Access control (RBAC, Namespaces)  

**Core concepts**:
- **Cluster**: Control plane + worker nodes
- **Pods**: Smallest deployable unit
- **Deployments**: Manage pod replicas
- **Services**: Network access
- **Storage**: Persistent data
- **ConfigMaps/Secrets**: Configuration

---
