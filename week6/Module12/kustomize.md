# Kustomize: Kubernetes Configuration Customization

## What is Kustomize? A Template-Free Approach to Kubernetes Configuration

### Definition

Kustomize is a Kubernetes-native configuration management tool that allows you to customize raw, template-free YAML files. Unlike templating tools like Helm, which use Go templating to generate YAML, Kustomize works by applying a set of declarative patches and transformations to existing Kubernetes manifests.

This means you start with your base configuration files and then define modifications for different environments or use cases.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Declarative Customization** | Kustomize operates on the principle of declarative configuration. You define what you want your Kubernetes resources to look like, and Kustomize figures out how to get there by applying specified changes to base manifests |
| **Template-Free** | You do not need to learn a templating language. You work directly with standard Kubernetes YAML files. This simplifies the learning curve and reduces potential syntax errors |
| **Layered Configuration** | Kustomize allows you to build configurations in layers. You can have a base configuration and then create multiple overlays that inherit from the base and apply specific customizations. This promotes reusability and maintainability |
| **Extensibility** | Kustomize supports various types of transformations, including strategic merging, JSON patching, and common label/annotation additions, making it highly flexible |

### Why is Kustomize Important?

In a typical Kubernetes workflow, you might have a single application that needs to be deployed across different environments: development, staging, production. Each environment might have slightly different requirements:

- **Development**: Might require more replicas for local testing, different resource limits, or debug logging enabled
- **Staging**: Might mirror production closely but with fewer replicas or different ingress configurations
- **Production**: Requires robust configurations, specific resource requests/limits, and security-hardened settings

Manually managing these variations by copying and editing YAML files is error-prone and time-consuming.

Kustomize solves this by allowing you to:

- **Reduce Duplication**: A single base configuration serves as the source of truth
- **Increase Consistency**: Environment-specific changes are clearly defined and applied systematically
- **Improve Maintainability**: Updating the base configuration automatically propagates to all overlays
- **Simplify Workflow**: Developers and operators can focus on core application manifests and define variations separately

---

## Understanding Kustomization Files (kustomization.yaml)

### What is a kustomization.yaml file?

A kustomization.yaml file is a YAML document that specifies a set of Kubernetes resources and a list of transformations to apply to them. It's the entry point for Kustomize to understand your desired configuration. You typically find this file in a directory that represents a specific configuration layer (e.g., a base configuration or an environment-specific overlay).

### Key Fields in kustomization.yaml

| Field | Purpose |
|-------|---------|
| **apiVersion** | Kustomization API version (typically kustomize.config.k8s.io/v1beta1 for Kustomize) |
| **kind** | Set to Kustomization |
| **resources** | List of paths to base Kubernetes YAML files (e.g., deployment.yaml, service.yaml) that Kustomize should process. Can be local file paths or URLs to Git repositories |
| **bases** | Used in overlays to specify one or more base Kustomizations from which to inherit |
| **patches** | Where the magic of customization happens. Allows specifying modifications to apply to existing resources |
| **patchStrategicMerge** | Most common type of patch. Performs strategic merge of patch YAML with target resource YAML using Kubernetes' strategic merge patch logic |
| **patchesJson6902** | Applies JSON Patch operations (RFC 6902) to resources. More granular and powerful for specific, complex modifications |
| **commonLabels** | Adds a set of labels to all resources |
| **commonAnnotations** | Adds a set of annotations to all resources |
| **images** | Allows modifying image tags or names for containers across resources |
| **names** | Allows renaming resources |
| **replicas** | Allows setting replica count for Deployments, StatefulSets, and ReplicaSets |
| **namespace** | Sets a default namespace for all resources |
| **namePrefix / nameSuffix** | Adds a prefix or suffix to the names of all resources |
| **configMapGenerator** | Generates ConfigMaps from files or literal values |
| **secretGenerator** | Generates Secrets from files or literal values |

### Example kustomization.yaml

**Scenario**: Base directory with deployment.yaml and service.yaml

**base/deployment.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

**base/service.yaml**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

**base/kustomization.yaml**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml
- service.yaml

commonLabels:
  app.kubernetes.io/part-of: my-application

images:
- name: nginx
  newName: nginx
  newTag: 1.25.3
```

**Explanation**:

- Includes deployment.yaml and service.yaml files
- Adds common label `app.kubernetes.io/part-of: my-application` to all resources
- Specifies that `nginx:latest` should be replaced with `nginx:1.25.3` (pinning to a specific version for reproducibility)

---

## Overlays and Bases: Building Layered Configurations

### What are Bases and Overlays?

| Concept | Description |
|---------|-------------|
| **Base** | A directory containing a kustomization.yaml file and the Kubernetes manifests it references. Represents the fundamental, shared configuration for your application. Should be as generic as possible, containing essential components without environment-specific tuning |
| **Overlay** | A directory representing a specific environment (e.g., overlays/development, overlays/staging, overlays/production). Also contains a kustomization.yaml file that references one or more bases and defines environment-specific customizations |

### Why Use Bases and Overlays?

This pattern is fundamental for managing configuration drift and promoting DRY (Do Not Repeat Yourself) principles:

- **Reusability**: Base configuration is written once and reused across all environments
- **Maintainability**: When updating a common resource, you only modify the base kustomization.yaml or its referenced manifests
- **Environment-Specific Tuning**: Each overlay precisely defines differences required for its target environment
- **Clear Separation of Concerns**: Base focuses on core application; overlays focus on operational aspects for different deployment targets

### Directory Structure Example

```
my-app/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── development/
    │   ├── deployment-patch.yaml
    │   ├── ingress.yaml
    │   └── kustomization.yaml
    └── production/
        ├── deployment-patch.yaml
        ├── replicas-patch.yaml
        ├── service-patch.yaml
        └── kustomization.yaml
```

### Implementing an Overlay: Development Environment

**overlays/development/kustomization.yaml**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base

resources:
- ingress.yaml

patchesStrategicMerge:
- deployment-patch.yaml

commonLabels:
  environment: development

images:
- name: nginx
  newTag: 1.25.3-alpine
```

**overlays/development/deployment-patch.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app-container
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

**overlays/development/ingress.yaml**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
  - host: my-app.dev.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

**Explanation**:

- `bases: - ../../base`: Tells Kustomize to inherit all resources and configurations from the base directory
- `resources: - ingress.yaml`: Adds a new resource (Ingress) specific to development environment
- `patchesStrategicMerge: - deployment-patch.yaml`: Applies modifications to my-app Deployment (only changed fields specified)
- `commonLabels: environment: development`: Adds environment-specific label to all resources
- `images: - name: nginx newTag: 1.25.3-alpine`: Overrides image tag specified in base

### Production Overlay Example

**overlays/production/kustomization.yaml**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base

resources:
- ingress.yaml

patchesStrategicMerge:
- deployment-replicas-patch.yaml
- deployment-resources-patch.yaml
- service-patch.yaml

commonLabels:
  environment: production

images:
- name: nginx
  newTag: 1.25.3
```

**overlays/production/deployment-replicas-patch.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5
```

**overlays/production/deployment-resources-patch.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app-container
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "1"
            memory: "2Gi"
```

**overlays/production/service-patch.yaml**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  type: LoadBalancer
```

This production overlay inherits from base and applies specific production-grade configurations including increased replicas, resource requirements, and external LoadBalancer access.

---

## Applying Patches and Transformations for Granular Control

### Understanding Patches

Patches are used to alter specific fields within existing Kubernetes resources. Kustomize supports several patching strategies:

| Patch Type | Description | Use Case |
|-----------|-------------|----------|
| **patchStrategicMerge** | Performs strategic merge of patch YAML with target YAML using Kubernetes' merge logic. Intelligently handles lists like container definitions by matching by key | Most common; intuitive for typical modifications |
| **patchesJson6902** | Applies JSON Patch operations (RFC 6902) to resources. Defines standard operations like add, remove, replace, copy, move, test | Complex modifications where strategic merging is insufficient |
| **patches** | More general way to specify patches with target selection (by name, namespace, kind, labels) and patch specification | Explicit targeting of specific resources |

### Understanding Transformations

Transformations are applied to all resources processed by a Kustomization file:

| Transformation | Purpose |
|----------------|---------|
| **commonLabels** | Adds specified labels to all resources |
| **commonAnnotations** | Adds specified annotations to all resources |
| **images** | Modifies container image names and tags across all resources |
| **names** | Renames resources |
| **replicas** | Sets replica count for Deployments, StatefulSets, ReplicaSets |
| **namespace** | Sets default namespace for resources without namespace defined |
| **namePrefix / nameSuffix** | Adds prefix or suffix to resource names |

### Practical Application: Modifying Resources with Patches

**Base configuration** (base/deployment.yaml and base/service.yaml as shown earlier)

**base/kustomization.yaml**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- deployment.yaml

images:
- name: nginx
  newName: nginx
  newTag: 1.25.3
```

**Staging overlay using patchStrategicMerge**:

**overlays/staging/kustomization.yaml**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
- ../../base

resources:
- ingress.yaml

patchesStrategicMerge:
- deployment-resources-patch.yaml

commonLabels:
  environment: staging

images:
- name: nginx
  newTag: 1.25.3-alpine
```

**overlays/staging/deployment-resources-patch.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app-container
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "400m"
            memory: "512Mi"
```

The strategic merge ensures that only the resources field is added/modified, leaving other parts of the Deployment untouched.

### Using JSON Patch (patchesJson6902)

For adding an environment variable using JSON Patch:

**overlays/staging/deployment-json-patch.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
      - name: my-app-container
        env:
        - name: LOG_LEVEL
          value: DEBUG
```

And in overlays/staging/kustomization.yaml:

```yaml
patchesJson6902:
- target:
    group: apps
    version: v1
    kind: Deployment
    name: my-app
  patch: |-
    - op: add
      path: /spec/template/spec/containers/0/env
      value:
        - name: LOG_LEVEL
          value: DEBUG
```

This JSON Patch adds an environment variable to the first container, demonstrating the precision offered by JSON Patch.

---

## Integrating Kustomize with kubectl for Seamless Deployment

### The kubectl apply -k Command

The primary way to use Kustomize with kubectl is through the `-k` flag. When you use `kubectl apply -k <directory>`, kubectl internally invokes Kustomize to build the configuration from the specified directory (which must contain a kustomization.yaml file) and then applies the resulting manifests to your cluster.

### How it Works

1. **Locate kustomization.yaml**: kubectl looks for a kustomization.yaml file in the specified directory
2. **Build Configuration**: Uses Kustomize's engine to process the kustomization.yaml, resolving bases, applying patches, and performing transformations
3. **Generate Manifests**: Kustomize generates a single, consolidated set of Kubernetes YAML manifests
4. **Apply Manifests**: kubectl apply takes these generated manifests and applies them to your active Kubernetes cluster

### Applying Kustomized Configurations

**Applying the development overlay**:

```bash
kubectl apply -k overlays/development/
```

This command will:

- Find kustomization.yaml in overlays/development/
- Process the base configuration
- Apply patches and transformations
- Add the specified labels
- Apply resulting manifests to cluster

**Verifying applied resources**:

```bash
kubectl get deployments
kubectl get services
kubectl get ingress
kubectl get pods -l app.kubernetes.io/part-of=my-application
kubectl get pods -l environment=development
```

**Applying the production overlay**:

```bash
kubectl apply -k overlays/production/
```

### Viewing Generated Manifests (without applying)

Sometimes you want to see the final YAML that Kustomize generates before applying:

```bash
# View built output for the base
kustomize build base

# View built output for development overlay
kustomize build overlays/development

# Save to file
kustomize build overlays/development > dev-manifests.yaml
```

### Updating and Deleting Resources

**Updating resources**:

Simply update the relevant Kustomize files and re-run `kubectl apply -k <directory>`. kubectl will detect changes and perform appropriate update strategy.

**Deleting resources**:

```bash
kubectl delete -k overlays/development/
```

This applies the same Kustomization logic to determine which resources to delete.

---

## Kustomize vs. Helm: Choosing the Right Tool for the Job

### Helm: The Kubernetes Package Manager

Helm uses charts (collections of pre-configured Kubernetes resources packaged together). Helm charts are templated using Go templating language for dynamic generation of YAML manifests.

**Key Characteristics of Helm**:

- **Templating**: Uses Go templating language for dynamic YAML generation
- **Packaging**: Charts designed for packaging and distributing applications
- **Release Management**: Provides robust installation, upgrade, rollback, and deletion capabilities
- **Dependency Management**: Charts can depend on other charts
- **Extensibility**: Supports hooks for custom scripts during release lifecycle
- **Complexity**: Can have steeper learning curve due to templating syntax and chart structure

### Kustomize: The Declarative Customizer

Template-free tool for customizing raw Kubernetes YAML files by applying patches and transformations.

**Key Characteristics of Kustomize**:

- **Template-Free**: Works directly with standard Kubernetes YAML
- **Declarative Customization**: Focuses on defining desired state through overlays and patches
- **Layering**: Excellent for building configurations in layers (bases and overlays)
- **Integration**: Built directly into kubectl
- **Simplicity**: Generally easier to learn for straightforward customization tasks
- **No Release Management**: Does not provide built-in release management features like Helm's rollback or upgrade tracking

### When to Use Kustomize

Kustomize is excellent when:

- You have existing Kubernetes manifests and need to manage variations for different environments
- You prefer working with raw YAML
- You need simple environment-specific overrides (replica counts, image tags, resource limits, labels/annotations)
- You want to avoid Helm's complexity
- You are already using kubectl extensively
- Managing configurations for internal tooling or platform components

### When to Use Helm

Helm is preferred when:

- Deploying third-party applications (many provide official Helm charts)
- You need robust release management (tracking releases, performing rollbacks, managing upgrades)
- Complex application deployments with many interconnected resources and dependencies
- Distributing your own applications (Helm chart is a standard practice)
- Managing complex dependencies between charts
- You need advanced templating capabilities (complex conditional logic, loops, custom functions)

### Comparison Table

| Feature | Kustomize | Helm |
|---------|-----------|------|
| **Approach** | Template-free, declarative patching | Templated, package management |
| **Core Unit** | Directory with kustomization.yaml and YAML files | Chart (templates and values) |
| **Customization** | Strategic merge, JSON patches, labels/annotations, image overrides | Go templating, values files |
| **Release Management** | None (relies on kubectl apply) | Built-in (install, upgrade, rollback, history) |
| **Learning Curve** | Lower (standard YAML) | Higher (templating language, chart structure) |
| **Use Cases** | Environment-specific overrides, managing raw YAML, simple customization | Third-party apps, complex applications, release lifecycle management, packaging |
| **Integration** | Built into kubectl | Standalone CLI tool |

### Can They Be Used Together?

Yes, Kustomize and Helm can be used together. A common pattern is to use Helm for base application deployment and Kustomize to manage application-specific configurations and deployments that depend on those Helm-deployed services. Alternatively, use Kustomize to manage Helm chart values files, effectively using Kustomize's layering to manage different sets of Helm values.

---
