# Helm: Kubernetes Package Manager

## What is Helm? The Package Manager for Kubernetes

### Definition

Helm is an open-source package manager designed specifically for Kubernetes. It streamlines the deployment and management of applications on Kubernetes clusters.

Before Helm, deploying applications often involved manually creating and managing numerous YAML files for Deployments, Services, ConfigMaps, Secrets, and more. This process was tedious, error-prone, and difficult to version control and reproduce.

Helm addresses these challenges by introducing a standardized way to package, deploy, and manage Kubernetes applications. It brings concepts familiar to users of traditional package managers like apt (Debian/Ubuntu), yum (RHEL/CentOS), or brew (macOS) to the Kubernetes ecosystem.

### Key Concepts of Helm

| Concept | Description |
|---------|-------------|
| **Charts** | Collections of files that describe a related set of Kubernetes resources. Charts are versioned and can be shared, making them ideal for distributing applications and configurations |
| **Repositories** | Collections of charts. Can be public (like Artifact Hub) or private (hosted on ChartMuseum or cloud provider registries) |
| **Releases** | An instance of a chart running in a Kubernetes cluster. When you install a chart, you create a release. Helm tracks the state of each release, allowing for easy upgrades and rollbacks |

### Why is Helm Important?

- **Simplifies Complex Deployments**: Helm allows you to define complex applications with multiple Kubernetes resources in a single, manageable package (a chart)
- **Templating Kubernetes Manifests**: Charts use templating engines (like Go's text/template) to dynamically generate Kubernetes YAML manifests. This means you can parameterize deployments, making them reusable across different environments (development, staging, production) without modifying core manifest files
- **Version Control and Rollbacks**: Helm tracks releases, providing a history of deployed versions. This makes it straightforward to roll back to a previous stable version if an upgrade introduces issues
- **Dependency Management**: Charts can declare dependencies on other charts, allowing you to bundle complex applications that rely on multiple components (e.g., a web application chart depending on a database chart)
- **Sharing and Reusability**: Charts can be easily shared, promoting collaboration and the reuse of pre-built application configurations. Many popular applications (databases, monitoring tools, web servers) are available as Helm charts
- **Consistency**: By using charts, you ensure that your applications are deployed consistently across different clusters and environments, reducing the "it works on my machine" problem

---

## Installing and Configuring Helm on Your Local Environment

### Prerequisites

- A running Kubernetes cluster (e.g., Minikube, kind)
- kubectl configured to communicate with your cluster
- Internet access

### Installation Steps

**Step 1: Download the Installation Script**

Open your terminal and run the following command. This script automatically detects your operating system and architecture.

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```

**Step 2: Make the Script Executable**

```bash
chmod 700 get_helm.sh
```

**Step 3: Run the Installation Script**

```bash
./get_helm.sh
```

The script downloads and installs the latest Helm 3 binary. By default, it installs into `/usr/local/bin`, which is typically in your system's PATH.

**Step 4: Verify the Installation**

```bash
helm version
```

Output should show the installed Helm version:

```
version.BuildInfo{Version:"v3.14.0", GitCommit:"...", GitTreeState:"clean", GoVersion:"go1.21.5"}
```

### Configuring Helm Repositories

**Step 5: Add a Helm Repository**

To add the Bitnami repository (well-maintained and widely used):

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

This associates a short name (bitnami) with the repository URL.

**Step 6: Update Your Local Repository Cache**

After adding a new repository, update your local Helm repository cache:

```bash
helm repo update
```

This fetches the latest list of charts available in that repository.

### Troubleshooting Common Installation Issues

| Issue | Solution |
|-------|----------|
| **Command not found** | Ensure helm binary is in your system's PATH. Restart terminal or log out and back in |
| **Permissions errors** | Run with sufficient privileges (e.g., using sudo ./get_helm.sh if necessary) |
| **Network issues** | Ensure stable internet connection to download the Helm binary and update repositories |

---

## Mastering Basic Helm Commands: Install, Upgrade, Uninstall, and List

### 1. Installing a Chart: helm install

The helm install command deploys a chart into your Kubernetes cluster. It takes a release name (unique name for this deployment) and a chart reference (repository/chart-name).

```bash
helm install my-nginx bitnami/nginx
```

When you run this command, Helm will:

1. Fetch the latest version of the nginx chart from the bitnami repository
2. Use default values defined within the chart
3. Render the Kubernetes manifests (Deployment, Service, etc.) based on chart templates and default values
4. Apply these manifests to your Kubernetes cluster
5. Create a new release named my-nginx

**Verify the installation with kubectl**:

```bash
kubectl get pods
# Should show a pod named something like my-nginx-nginx-xxxxxxxxxx-yyyyy

kubectl get services
# Should show a service named my-nginx-nginx
```

### 2. Listing Releases: helm list

To see all Helm releases currently deployed in your cluster:

```bash
helm list
```

Output includes:

| Column | Description |
|--------|-------------|
| **NAME** | Name of the release (e.g., my-nginx) |
| **NAMESPACE** | Kubernetes namespace where release is deployed |
| **REVISION** | Revision number of the release |
| **UPDATED** | When the release was last updated |
| **STATUS** | Current status (e.g., deployed, failed) |
| **CHART** | Chart used for the release (e.g., bitnami/nginx-15.6.0) |
| **APP VERSION** | Version of the application (e.g., 1.25.3) |

To list releases across all namespaces:

```bash
helm list -A
```

### 3. Upgrading a Release: helm upgrade

As applications evolve, use helm upgrade to update a release to a new chart version or change configuration values.

```bash
helm upgrade my-nginx bitnami/nginx --set replicaCount=2
```

Helm will:

1. Identify the existing release my-nginx
2. Fetch the specified chart
3. Apply the new configuration (replicaCount=2)
4. Generate new Kubernetes manifests
5. Apply these manifests to update existing resources
6. Increment the revision number for the release

**Verify the upgrade**:

```bash
kubectl get deployment my-nginx-nginx -o=jsonpath='{.spec.replicas}'
# Should output 2
```

### 4. Uninstalling a Release: helm uninstall

When you no longer need an application, remove it cleanly:

```bash
helm uninstall my-nginx
```

Helm will:

1. Identify the release my-nginx
2. Retrieve the manifests originally applied for this release
3. Delete all associated Kubernetes resources (Deployments, Services, Pods, etc.)
4. Mark the release as uninstalled

**Confirm uninstallation**:

```bash
helm list
# my-nginx should no longer appear
```

### Important Note on Rollbacks

Helm keeps a history of revisions for each release. If an upgrade fails or introduces issues, you can easily roll back to a previous revision:

```bash
helm rollback my-nginx 1
# Reverts to revision 1
```

---

## Understanding Helm Chart Structure: Chart.yaml, values.yaml, and Templates

### Chart Directory Structure

A typical Helm chart directory looks like this:

```
mychart/
├── Chart.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
└── values.yaml
```

### 1. Chart.yaml: Chart Metadata

This file contains descriptive metadata about the chart. It's essential for identifying and managing the chart.

**Key Fields**:

| Field | Purpose |
|-------|---------|
| **apiVersion** | Helm API version (v2 or v3 for Helm 3) |
| **name** | Name of the chart (e.g., nginx) |
| **version** | Semantic version of the chart itself (e.g., 15.6.0) |
| **appVersion** | Version of the application this chart deploys (e.g., 1.25.3) |
| **description** | Brief description of what the chart does |
| **type** | application or library |
| **keywords** | List of keywords for searching |
| **maintainers** | Information about chart maintainers |
| **dependencies** | List of other charts this chart depends on |

**Example Chart.yaml**:

```yaml
apiVersion: v2
name: my-custom-app
version: 0.1.0
appVersion: "1.16.0"
description: A simple web application chart
type: application
keywords:
  - web
  - application
maintainers:
  - name: Your Name
    email: your.email@example.com
dependencies:
  - name: postgresql
    version: "11.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
```

### 2. values.yaml: Default Configuration Values

This file defines the default configuration parameters for your chart. These parameters can be used to customize the deployment without modifying template files directly.

**Key Concepts**:

- **Parameters**: Key-value pairs representing configurable aspects (e.g., number of replicas, image tag, service type, resource limits)
- **Nesting**: Values can be nested to create complex configurations, mirroring Kubernetes object structure
- **Templating**: Values are accessible within template files using Go templating syntax (e.g., `{{ .Values.replicaCount }}`)

**Example values.yaml**:

```yaml
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []

resources: {}
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80

postgresql:
  enabled: false
```

### 3. templates/ Directory: Kubernetes Manifests

This directory contains Kubernetes manifest files (YAML) that define the resources for your application. These are not static YAML; they are templates using Go's templating language.

**Key Concepts**:

- **Templating Engine**: Helm uses Go's text/template and sprig functions
- **Variables**: Reference values from values.yaml using `{{ .Values.someValue }}`
- **Built-in Objects**: Access to built-in objects like `.Release`, `.Chart`, and `.Capabilities`
- **Control Flow**: Use conditional logic (if/else), loops (range), and functions
- **Helpers**: The `_helpers.tpl` file defines reusable template snippets

**Example templates/deployment.yaml**:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-custom-app.fullname" . }}
  labels:
    {{- include "my-custom-app.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-custom-app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-custom-app.selectorLabels" . | nindent 8 }}
    spec:
      serviceAccountName: {{ include "my-custom-app.serviceAccountName" . }}
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

### 4. charts/ Directory: Subcharts and Dependencies

This directory stores subcharts that your main chart depends on. When you declare dependencies in Chart.yaml and run `helm dependency build`, the subcharts are placed here. This allows for complex applications composed of multiple independently managed components.

---

## Using Helm Repositories: Discovering and Deploying Applications

### Types of Helm Repositories

| Type | Description |
|------|-------------|
| **Public Repositories** | Hosted by communities or vendors, accessible to anyone (e.g., Bitnami, Artifact Hub) |
| **Private Repositories** | Hosted within your organization or by a third-party service for internal use (e.g., ChartMuseum, Harbor, cloud registries) |

### Key Helm Repository Commands

#### 1. Adding a Repository: helm repo add

```bash
helm repo add <name> <url>
```

**Examples**:

```bash
# Add Bitnami repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Add official stable repository
helm repo add stable https://charts.helm.sh/stable
```

#### 2. Updating Local Repository Cache: helm repo update

```bash
helm repo update
```

Fetches the latest index file from all configured repositories, updating the list of available charts and their versions.

#### 3. Listing Configured Repositories: helm repo list

```bash
helm repo list
```

Output example:

```
NAME    URL                                     
bitnami  https://charts.bitnami.com/bitnami    
stable  https://charts.helm.sh/stable
```

#### 4. Searching for Charts: helm search repo

```bash
helm search repo <keyword>
```

**Examples**:

```bash
# Search for all charts related to 'mysql'
helm search repo mysql

# Search for specific chart (e.g., PostgreSQL from Bitnami)
helm search repo bitnami/postgresql
```

#### 5. Removing a Repository: helm repo remove

```bash
helm repo remove <name>
```

**Example**:

```bash
helm repo remove stable
```

### Using Artifact Hub for Discovery

Artifact Hub (https://artifacthub.io/) is a web-based platform that aggregates Helm charts, container images, and other artifacts from various sources.

**Example Workflow: Deploying Prometheus**

1. Discover: Search for 'Prometheus' on Artifact Hub
2. Add Repository: Found chart is from https://prometheus-community.github.io/helm-charts
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   ```
3. Update Cache:
   ```bash
   helm repo update
   ```
4. Search: Verify chart is available
   ```bash
   helm search repo prometheus-community/prometheus
   ```
5. Install: Deploy with custom configuration
   ```bash
   helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace --values prometheus-values.yaml
   ```

---

## Customizing Helm Installations with values.yaml

### Creating a Custom Values File

Instead of using `--set` flags repeatedly, create a custom YAML file containing desired overrides.

**Example: my-nginx-values.yaml**

```yaml
replicaCount: 3

service:
  type: LoadBalancer

ingress:
  enabled: true
  className: "nginx"
  hosts:
    - host: nginx.local
      paths:
        - path: /
          pathType: Prefix
  tls: []
```

### Installing with Custom Values

```bash
helm install my-custom-nginx bitnami/nginx -f my-nginx-values.yaml
```

Helm uses values from `my-nginx-values.yaml` to render templates.

### Upgrading with New Values

Modify your values file and upgrade:

```bash
helm upgrade my-custom-nginx bitnami/nginx -f my-nginx-values.yaml
```

Helm performs an upgrade, creating a new revision for the release.

---

## Summary: Key Takeaways

- **Helm as Package Manager**: Standardizes deployment of Kubernetes applications, similar to apt, yum, or brew
- **Charts**: Core packaging format containing Kubernetes manifests as templates
- **values.yaml**: Central file for parameterizing charts, allowing customization without altering templates
- **Chart.yaml**: Provides metadata about the chart (name, version, appVersion, description, dependencies)
- **templates/**: Contains Go templated Kubernetes manifest files
- **Repositories**: Collections of charts that Helm can access (public and private)
- **Core Commands**: `helm install`, `helm upgrade`, `helm uninstall`, `helm list` are essential for managing application lifecycles
- **Customization**: Using custom values.yaml files or `--set` flags enables flexible deployments tailored to specific environments
- **Releases**: Tracked instances of charts running in cluster, with history for upgrades and rollbacks

- **Test Thoroughly**: Test Helm deployments in development and staging before production
- **Use helm template**: Use `helm template` to see generated Kubernetes manifests before applying for debugging
- **Consider Private Repositories**: For internal use, set up your own private Helm repository (ChartMuseum or cloud registries)
