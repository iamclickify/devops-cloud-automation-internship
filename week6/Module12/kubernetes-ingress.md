# Kubernetes Ingress: Exposing Applications Externally

## Understanding Kubernetes Ingress: The API Object for External Access

### What is Ingress?

Kubernetes Ingress is an API object that defines rules for how external HTTP and HTTPS traffic should be routed to services within your Kubernetes cluster. It acts as a layer of abstraction, allowing you to manage external access to your applications without directly exposing each Service via a LoadBalancer or NodePort.

Think of Ingress as a smart router for your cluster. It does not perform the actual routing itself; instead, it provides a declarative configuration that an Ingress controller consumes.

### Primary Purposes of Ingress Resources

- **Expose HTTP and HTTPS routes**: Route external traffic from outside the cluster to services within the cluster
- **Provide load balancing**: Distribute traffic across multiple pods of a service
- **Offer name-based virtual hosting**: Allow multiple hostnames to be served from a single IP address
- **Enable path-based routing**: Direct traffic to different services based on the URL path
- **Facilitate TLS termination**: Handle SSL/TLS certificates centrally

### Why is Ingress Important?

Before Ingress, exposing services often involved:

- **type: LoadBalancer** for each Service (expensive, as each LoadBalancer incurs costs)
- **type: NodePort** (less flexible, requires external mechanisms for load balancing and SSL termination)

Ingress consolidates these responsibilities:

- **Cost Efficiency**: A single Ingress controller manages traffic for many services, utilizing a single external IP address or load balancer
- **Simplified Management**: Centralizes routing rules, SSL certificates, and external access configurations in one place
- **Flexibility**: Easily add, remove, or modify routing rules without redeploying applications
- **Advanced Routing**: Supports path-based routing and host-based routing
- **Security**: Enables centralized TLS termination; application pods do not need to manage SSL certificates directly

### Ingress Resource Structure

An Ingress resource is defined in YAML. It typically includes:

| Field | Purpose |
|-------|---------|
| **apiVersion** | API version for Ingress (e.g., networking.k8s.io/v1) |
| **kind** | Set to Ingress |
| **metadata** | Name, namespace, labels, annotations |
| **spec** | Core routing logic containing rules |

**Key Spec Fields**:

| Field | Purpose |
|-------|---------|
| **rules** | List of routing rules |
| **host** | Domain name (e.g., myapp.example.com). If omitted, applies to all hostnames |
| **http** | Contains paths list |
| **paths** | List of path-based routing rules |
| **path** | URL path (e.g., /, /api, /static) |
| **pathType** | How path is matched (Prefix, Exact, ImplementationSpecific) |
| **backend** | Where traffic should be sent (Service and port) |
| **tls** | TLS certificate configuration |

### Example Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
  - host: myapp.example.com
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

This defines an Ingress resource that routes all requests to `myapp.example.com` with a path starting with `/` to `my-app-service` on port 80.

---

## Ingress Controllers: The Engine Behind Ingress Resources

### What is an Ingress Controller?

An Ingress controller is a deployment within your Kubernetes cluster that watches the Kubernetes API for Ingress resources and configures a load balancer or reverse proxy to implement the rules defined in those resources.

### Responsibilities of an Ingress Controller

- **Watch for Ingress resources**: Continuously monitor the Kubernetes API server for changes to Ingress objects
- **Interpret Ingress rules**: Parse defined rules (hosts, paths, backends)
- **Configure a proxy/load balancer**: Dynamically configure an underlying proxy (Nginx, HAProxy, Traefik, Envoy) to route traffic according to Ingress rules
- **Manage TLS certificates**: Fetch and manage TLS certificates from Kubernetes Secrets
- **Handle health checks**: Perform health checks on backend pods to ensure traffic is sent only to healthy instances

### Popular Ingress Controllers

| Controller | Features | Strengths |
|------------|----------|-----------|
| **Nginx Ingress Controller** | Host/path routing, TLS, auth, rate limiting, rewrites | Robust, extensive features, active community |
| **Traefik** | Automatic discovery, Let's Encrypt integration, dashboard | Modern, easy to use, dynamic configuration |
| **HAProxy Ingress Controller** | High performance, robust load balancing | Reliability and performance |
| **Cloud Provider Controllers** | Integration with managed load balancing services | Tight cloud integration (AWS, GCP, Azure) |

### Why Nginx Ingress Controller is a Good Starting Point

- **Ubiquity**: Nginx is a de facto standard for web servers and reverse proxies
- **Rich Feature Set**: Supports wide range of functionalities for modern web applications
- **Extensive Documentation**: Abundant resources and community support
- **Compatibility**: Adheres to the Kubernetes Ingress API specification

### How Ingress Controllers Work with Ingress Resources

1. Deploy an Ingress controller (e.g., Nginx Ingress Controller) as a Deployment in your cluster
2. The controller is exposed via a Service (often type: LoadBalancer or NodePort) with an external IP address or port
3. Create an Ingress resource (YAML definition) specifying routing rules
4. The Ingress controller watches the Kubernetes API and detects your Ingress resource
5. Based on these rules, the controller configures the underlying Nginx proxy (generates configuration, reloads it)
6. External traffic hitting the Ingress controller's external IP address is routed by Nginx according to your rules, forwarding it to appropriate backend Services and pods

---

## Creating Ingress Resources: Defining Rules, Paths, and Hosts

### The spec Field: The Core of Ingress Configuration

The spec field contains a list of rules. Each rule dictates how traffic should be handled for a specific host or set of hosts.

### 1. Hosts: Specifying Target Domains

The host field enables name-based virtual hosting, where a single Ingress controller manages traffic for multiple domain names. If the host field is omitted, the rule applies to all hostnames not explicitly matched by other rules.

**Example**:

```yaml
spec:
  rules:
  - host: app.example.com
    http:
      paths:
        # ... path definitions ...
  - host: api.example.com
    http:
      paths:
        # ... path definitions ...
```

Requests to `app.example.com` are handled by the first set of paths, while requests to `api.example.com` are handled by the second set.

### 2. Paths: Routing Based on URL Segments

Within each host's rule, the http field contains a list of paths. Each path entry specifies:

| Field | Purpose |
|-------|---------|
| **path** | URL path to match (e.g., /, /api, /static) |
| **pathType** | How the path is matched |
| **backend** | Where traffic should be sent (Service and port) |

### Path Types

| Path Type | Behavior | Examples |
|-----------|----------|----------|
| **Prefix** | Matches any URL path starting with the specified path | path: /api matches /api, /api/users, /api/v1/items |
| **Exact** | Matches the URL path exactly | path: /login matches only /login |
| **ImplementationSpecific** | Matching determined by controller implementation; not recommended for portability | Varies by controller |

### Example: Path-Based Routing

```yaml
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-api-service
            port:
              number: 8080
```

- Requests to `myapp.example.com/` and sub-paths → `frontend-service:80`
- Requests to `myapp.example.com/api` and sub-paths → `backend-api-service:8080`

### 3. Default Backend: Handling Unmatched Requests

You can define a defaultBackend at the top level of the spec. This backend receives any traffic that does not match any defined rules.

```yaml
spec:
  defaultBackend:
    service:
      name: default-error-service
      port:
        number: 80
  rules: []
```

### Comprehensive Example: Combining Hosts and Paths

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mycompany-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: www.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service
            port:
              number: 80
      - path: /api/v1
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 3000
  - host: blog.mycompany.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: blog-service
            port:
              number: 80
```

### Important Considerations

- **Ingress Controller Specific Annotations**: Many controllers support custom annotations to enable advanced features (e.g., `nginx.ingress.kubernetes.io/rewrite-target` for URL rewriting). These are controller-specific
- **Order of Rules**: More specific rules (e.g., exact paths) should be defined before general ones (e.g., prefix paths or default backends)
- **TLS Configuration**: Configured within the Ingress resource in the spec.tls section

---

## Securing Your Applications: TLS Termination with Ingress

### What is TLS Termination?

TLS (Transport Layer Security) termination is the process of decrypting encrypted HTTPS traffic at a point before it reaches your backend application servers. In Kubernetes Ingress, the Ingress controller performs this decryption.

The traffic between the Ingress controller and application pods can then be unencrypted HTTP, simplifying certificate management within applications.

### Why Use Ingress for TLS Termination?

- **Centralized Certificate Management**: Manage certificates in one place (Kubernetes Secrets) instead of on each application pod
- **Simplified Application Code**: Applications do not need to handle SSL/TLS certificates or encryption/decryption; they listen on HTTP
- **Cost and Resource Efficiency**: TLS operations can be CPU-intensive; offloading to the Ingress controller saves resources
- **Easier Certificate Rotation**: Update certificates in one place rather than redeploying multiple applications
- **Let's Encrypt Integration**: Many controllers integrate with Let's Encrypt for automatic certificate provisioning and renewal

### How to Configure TLS Termination in Ingress

#### Step 1: Creating a TLS Secret

Create a Kubernetes Secret of type `kubernetes.io/tls` using kubectl:

```bash
kubectl create secret tls my-tls-secret \
  --cert=/path/to/certificate.crt \
  --key=/path/to/private.key \
  -n your-namespace
```

Replace:
- `my-tls-secret` with desired secret name
- `/path/to/certificate.crt` with actual certificate file path
- `/path/to/private.key` with actual private key file path
- `your-namespace` with namespace where Ingress will reside

This creates a Secret containing two keys: `tls.crt` (certificate) and `tls.key` (private key).

#### Step 2: Referencing the Secret in the Ingress Resource

Reference the Secret in the `spec.tls` section of your Ingress resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-app-ingress
  namespace: default
spec:
  tls:
  - hosts:
    - secure.example.com
    secretName: my-tls-secret
  rules:
  - host: secure.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: secure-app-service
            port:
              number: 80
```

When a client connects to `secure.example.com` via HTTPS:

1. Ingress controller intercepts the request
2. Uses certificate from `my-tls-secret` to establish secure TLS connection
3. Forwards decrypted HTTP traffic to `secure-app-service:80`

### Handling Multiple Hosts and Certificates

For multiple hosts with different certificates:

```yaml
spec:
  tls:
  - hosts:
    - secure.example.com
    secretName: my-tls-secret
  - hosts:
    - another.example.com
    secretName: another-tls-secret
  rules:
    # ... rules for both hosts ...
```

### Wildcard Certificates

Use wildcard certificates to secure multiple subdomains:

```yaml
spec:
  tls:
  - hosts:
    - '*.example.com'
    secretName: wildcard-tls-secret
```

A certificate for `*.example.com` secures all subdomains (e.g., `app.example.com`, `api.example.com`).

### Automatic Certificate Management with Cert-Manager

For automated certificate management (especially with Let's Encrypt), use the cert-manager project. It automatically provisions certificates from various issuers and stores them as Kubernetes Secrets, which Ingress resources then reference.

---

## Load Balancing and Routing: The Power of Ingress

### Load Balancing with Ingress

When you define an Ingress resource pointing to a Service with multiple pods, the Ingress controller acts as a load balancer, distributing incoming requests across these pods. The specific load balancing algorithm depends on the underlying proxy configuration.

### Common Load Balancing Algorithms

| Algorithm | Behavior | Use Case |
|-----------|----------|----------|
| **Round Robin** | Distributes requests sequentially to each backend server | General purpose |
| **Least Connections** | Sends request to server with fewest active connections | Long-lived connections |
| **IP Hash** | Routes requests from same client IP to same backend server | Session affinity, sticky sessions |
| **Weighted Round Robin/Least Connections** | Assign different weights to servers | Varying server capacities |

The Ingress controller dynamically updates its configuration to reflect the current state of backend pods, ensuring traffic is directed to healthy and available instances.

### Advanced Routing Strategies

#### 1. URL Rewriting

Sometimes the external URL structure differs from the internal path expected by backend services. Ingress controllers often provide URL rewriting mechanisms.

**Example: URL Rewriting with Nginx Ingress Controller**:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /app(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80
```

The path `/app(/|$)(.*)` matches requests like `/app`, `/app/`, `/app/some/path`. The annotation rewrites the matched path to `/` before forwarding to the service. A request to `myapp.example.com/app/users` is rewritten to `/users` before being sent to the service.

#### 2. Request Buffering and Timeouts

Ingress controllers can manage request buffering and set timeouts for client connections and backend responses, preventing resource exhaustion and ensuring timely responses.

#### 3. Rate Limiting

Restrict the number of requests a client can make within a certain time period to protect services from being overwhelmed by excessive traffic.

#### 4. Authentication and Authorization

Some controllers support basic authentication (HTTP Basic Auth) or can integrate with external authentication services (like OAuth2 proxies).

#### 5. WebSocket Support

Ingress controllers typically support protocols beyond HTTP, including WebSockets for real-time applications.

#### 6. Session Affinity (IP Hash)

For applications requiring users to connect to the same backend pod for a session:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: sticky-session-ingress
  annotations:
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/session-cookie-name: MYAPP_SESSION
spec:
  rules:
  - host: sticky.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-sticky-app-service
            port:
              number: 80
```

---

- **Keep Ingress Resources Organized**: Group related rules or use tools like Helm for management
- **Understand Path Types**: Be mindful of Prefix vs Exact to ensure correct routing behavior
