# Prometheus: Time-Series Monitoring Fundamentals

## Deconstructing Prometheus: Architecture and Core Components

### Prometheus Server: The Heart of the System

The Prometheus server is the central piece of the Prometheus ecosystem. Its primary responsibilities include:

| Responsibility | Description |
|----------------|-------------|
| **Scraping** | Periodically fetches metrics from configured targets (applications, services, exporters) over HTTP |
| **Storage** | Stores collected time-series data efficiently in its local time-series database (TSDB) |
| **Querying** | Exposes an API allowing users to query stored data using PromQL |
| **Configuration** | Reads configuration files determining what targets to scrape, how to evaluate alerting rules, and how to handle remote storage |

The Prometheus server is designed to be highly available and resilient. It can run as a single binary, making deployment straightforward. For larger deployments, federation and remote read/write capabilities allow for scaling.

### Exporters: The Metric Providers

Exporters are standalone applications that:

- Collect metrics from a specific source (database, web server, operating system)
- Expose these metrics in Prometheus format via an HTTP endpoint (e.g., /metrics)

**Why are exporters important?** They decouple metric collection from the application itself. You can monitor applications without modifying their core code, promoting cleaner architecture and easier integration.

**Common Exporters**:
- Node Exporter: Hardware and OS metrics
- Database Exporters: MySQL, PostgreSQL
- Application-Specific: Nginx, Redis, Kafka
- Blackbox Exporter: Probes endpoints for availability

### Pushgateway: For Short-Lived Jobs

While Prometheus primarily uses a pull model, the Pushgateway acts as a buffer for scenarios where targets might not be continuously available or are short-lived batch jobs.

**Use Cases**:
- Batch jobs running for limited time
- CI/CD pipelines where metrics are generated during build/test phases
- Services that are not always reachable via direct scraping

**Important Note**: Pushgateway is not intended for long-term metric storage; it's a temporary intermediary.

### Alertmanager: Handling Alerts

The Alertmanager is responsible for:

- **Receiving alerts**: From Prometheus based on configured alerting rules
- **Deduplicating, grouping, and silencing**: Handling incoming alerts to avoid alert storms
- **Routing**: Routes alerts to correct receivers based on configured rules (email, Slack, PagerDuty, OpsGenie)

---

## The Language of Monitoring: Understanding Metrics, Labels, and Time Series

### Metrics: The Raw Data Points

A metric is a measurement at a particular point in time. There are four main types:

| Metric Type | Description | Characteristics |
|------------|-------------|-----------------|
| **Counter** | Cumulative metric that only ever increases | Represents count of events; resets on service restart |
| **Gauge** | Single numerical value that can go up or down | Current memory, active users, CPU temperature |
| **Histogram** | Samples observations and counts them in buckets | Excellent for understanding distribution; includes sum and count |
| **Summary** | Similar to histogram; calculates configurable quantiles | Useful for latency distributions; more resource-intensive |

Each metric has a name (e.g., `http_requests_total`, `process_resident_memory_bytes`) and can optionally have labels.

### Labels: Adding Context to Metrics

Labels are key-value pairs that attach metadata to metrics. They are the primary way to slice and dice time-series data.

**Example**: Metric `http_requests_total` with labels:
- `method='GET'`
- `path='/api/v1/users'`
- `status_code='200'`
- `instance='192.168.1.10:8080'`

**Allows querying for**:
- Total GET requests: `http_requests_total{method='GET'}`
- Requests to specific path: `http_requests_total{path='/api/v1/users'}`
- Successful requests: `http_requests_total{status_code=~'2..'}`
- Per-instance requests: `http_requests_total{instance!=''}`

**Important Label Notes**:
- Label names must match regex: `[a-zA-Z_:][a-zA-Z0-9_:]*`
- Label values can be any sequence of UTF-8 characters
- Special label `__name__` is reserved for metric name
- Avoid high-cardinality labels (very large number of unique values) as they impact performance

### Time Series: The Complete Picture

A time series is a sequence of data points (timestamp and value) associated with a unique combination of metric name and label set.

Example: `http_requests_total{method='GET', path='/home', status_code='200'}` is one unique time series.

Prometheus stores and queries these time series. A PromQL query essentially selects one or more time series based on metric name and labels, then performs operations on their values.

---

## Setting Up Your Monitoring Foundation: Installing and Configuring Prometheus

### Prerequisites

- Linux-based OS (Ubuntu, CentOS, macOS with Homebrew)
- Basic command-line proficiency
- wget or curl for downloading files
- tar for extracting archives

### Installation Steps

**Step 1: Download the Prometheus binary**

```bash
PROMETHEUS_VERSION='2.51.1'  # Replace with latest version
wget https://github.com/prometheus/prometheus/releases/download/v${PROMETHEUS_VERSION}/prometheus-${PROMETHEUS_VERSION}.linux-amd64.tar.gz
```

**Step 2: Extract the archive**

```bash
tar -xvzf prometheus-${PROMETHEUS_VERSION}.linux-amd64.tar.gz
```

**Step 3: Organize the files**

```bash
mkdir ~/prometheus
mv prometheus-${PROMETHEUS_VERSION}.linux-amd64/prometheus ~/prometheus/
mv prometheus-${PROMETHEUS_VERSION}.linux-amd64/promtool ~/prometheus/
mv prometheus-${PROMETHEUS_VERSION}.linux-amd64/consoles ~/prometheus/
mv prometheus-${PROMETHEUS_VERSION}.linux-amd64/console_libraries ~/prometheus/
cd ~/prometheus/
```

**Step 4: Create a configuration file**

Create `prometheus.yml` in the `~/prometheus` directory:

```yaml
global:
  scrape_interval: 15s  # Default scrape interval

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']  # Prometheus itself

  # Add more jobs as needed for other targets
```

**Step 5: Run Prometheus**

```bash
./prometheus --config.file='prometheus.yml'
```

Prometheus will start and be accessible at `http://localhost:9090`.

### Verifying Installation

Navigate to `http://localhost:9090` in your browser. Key sections:

| Section | Purpose |
|---------|---------|
| **Graph** | Execute PromQL queries and visualize results |
| **Targets** | Lists configured scrape targets and their state (UP/DOWN) |
| **Status** | Server health, configuration, and target info |
| **Alerts** | Shows active alerts fired by Prometheus |
| **Configuration** | Displays currently loaded configuration |

---

## Exploring Your Data: Navigating the Prometheus UI

### The Graph Page: Your Query Playground

The Graph page (`http://localhost:9090/graph`) has two modes:

- **Graph**: Displays PromQL query results as line graphs
- **Console**: Displays raw JSON output

### Executing Your First Queries

**Query a counter**:
```promql
myapp_requests_total
```
Shows cumulative request count increasing over time.

**Query a gauge**:
```promql
myapp_active_requests
```
Shows current number of active requests, fluctuating as requests are processed.

**Query request rate**:
```promql
rate(myapp_requests_total[5m])
```
Shows requests per second over last 5 minutes.

**Query average request duration**:
```promql
rate(myapp_request_duration_seconds_sum[5m]) / rate(myapp_request_duration_seconds_count[5m])
```

**Query histogram buckets**:
```promql
sum(rate(myapp_request_duration_seconds_bucket[5m])) by (le)
```
Shows distribution of request durations across buckets.

### UI Elements

| Element | Purpose |
|---------|---------|
| **Query Input** | Where you type PromQL queries |
| **Execute Button** | Runs your query |
| **Mode Toggle** | Switch between Graph and Console output |
| **Time Range** | Select time window for queries (last 5m, 1h, etc.) |
| **Expression Browser** | Left sidebar for discovering available metrics and labels |
| **Results Panel** | Displays query output as graph or raw data |

---

## Mastering Metrics: Writing Basic PromQL Queries

### The Anatomy of a PromQL Query

A basic PromQL query consists of:

| Component | Example |
|-----------|---------|
| **Metric Name** | `http_requests_total` |
| **Label Selectors** | `{method='GET', status_code='200'}` |
| **Operators** | `+`, `-`, `*`, `/` |
| **Functions** | `rate()`, `sum()`, `avg()`, `histogram_quantile()` |
| **Range Vectors** | `[5m]` (5 minutes) |

### Basic Selectors

**Simplest query** - select all time series for a metric:
```promql
http_requests_total
```

**Filter by labels** - exact match:
```promql
http_requests_total{method='GET'}
```

**Regex matching**:
```promql
http_requests_total{status_code=~'2..|3..'}  # Status 2xx or 3xx
```

**Label matching operators**:

| Operator | Meaning |
|----------|---------|
| `=` | Exact match |
| `!=` | Not equal |
| `=~` | Regex match |
| `!~` | Not regex match |

### Instant vs. Range Queries

| Query Type | Purpose | Example |
|-----------|---------|---------|
| **Instant** | Single value at specific time (usually now) | `http_requests_total` |
| **Range** | Values over specified duration | `rate(http_requests_total[5m])` |

### Working with Counters: rate() and increase()

**rate()** - calculates per-second average rate of increase:
```promql
rate(myapp_requests_total[5m])  # Average requests/sec over last 5 minutes
```

**increase()** - calculates total increase over duration:
```promql
increase(myapp_requests_total[10m])  # Total requests in last 10 minutes
```

**With label filtering**:
```promql
rate(myapp_requests_total{method='GET'}[5m])  # Rate of GET requests
```

### Aggregating Metrics: sum(), avg(), count()

| Function | Purpose |
|----------|---------|
| **sum()** | Sums values of all selected time series |
| **avg()** | Calculates average of values |
| **count()** | Counts number of selected time series |
| **min(), max()** | Minimum and maximum values |

**Examples**:
```promql
sum(myapp_requests_total)  # Total across all instances

avg(rate(myapp_request_duration_seconds_sum[5m]) / rate(myapp_request_duration_seconds_count[5m]))  # Average duration

count(myapp_requests_total) by (status_code)  # Count by status code
```

**by clause**: Specifies which labels to keep in aggregated results.

```promql
rate(myapp_requests_total[5m]) by (method)  # Request rate per method
```

### Working with Gauges: avg_over_time()

For gauges representing instantaneous values:
```promql
avg_over_time(myapp_active_requests[10m])  # Average active requests over 10 minutes
```

### Working with Histograms: histogram_quantile()

```promql
histogram_quantile(0.95, sum(rate(myapp_request_duration_seconds_bucket[5m])) by (le))
```

Calculates 95th percentile of request duration.

---

## Exposing Metrics from Applications and Services

### The Pull Model: Prometheus Scrapes Targets

Prometheus operates on a pull model: the server actively connects to targets at regular intervals (defined by `scrape_interval`) and requests metrics via HTTP endpoint (usually `/metrics`).

For this to work, targets must:
- Be running and accessible over the network
- Expose HTTP endpoint serving metrics in Prometheus text format

### Types of Metric Exposure

#### 1. Instrumenting Applications Directly

Add Prometheus client libraries to application codebase. Application exposes metrics directly via HTTP endpoint.

**Pros**:
- Most granular and application-specific metrics
- Low overhead

**Cons**:
- Requires modifying application code
- Not feasible for third-party applications

**Examples**: 
- Go: `github.com/prometheus/client_golang`
- Java: `prometheus/client_java`
- Python: `prometheus_client`

#### 2. Using Exporters

Standalone applications that collect metrics from a specific system and expose in Prometheus format.

**How exporters work**:
1. Run as separate process
2. Connect to target service
3. Query service for metrics
4. Transform metrics into Prometheus text format
5. Expose via HTTP endpoint

**Pros**:
- No need to modify application
- Wide range of pre-built exporters
- Decouples monitoring from application logic

**Cons**:
- Requires deploying additional processes
- May not provide as granular metrics as direct instrumentation

### Service Discovery

In dynamic environments (Kubernetes, cloud platforms), manually configuring targets is not scalable.

**Prometheus supports various service discovery mechanisms**:
- Kubernetes SD: Discovers pods and services
- Consul SD: Integrates with Consul
- EC2/Azure/GCP SD: Cloud platform discovery

Allows Prometheus to automatically find new targets as deployed or removed.

### Push Model (via Pushgateway)

While Prometheus primarily pulls, Pushgateway handles pushing for short-lived jobs.

**Use cases**:
- Batch jobs, CI/CD pipelines
- Ephemeral services spinning up/down

**Workflow**: Jobs push metrics to Pushgateway → Prometheus scrapes Pushgateway

**Caution**: Not designed for long-term storage; temporary buffer only.

### Best Practices for Exposing Metrics

- Use standard exporters when possible
- Choose appropriate metric types (Counter, Gauge, Histogram)
- Use meaningful metric names (e.g., `http_requests_total`)
- Add labels for context but avoid high-cardinality labels
- Monitor your exporters themselves
- Secure metrics endpoints in production (authentication/authorization)

---

## Summary: Key Takeaways

### Core Components

- **Prometheus Server**: Central hub for scraping, storage, and querying
- **Exporters**: Standalone applications exposing metrics from specific systems
- **Pushgateway**: Temporary buffer for short-lived jobs
- **Alertmanager**: Handles alert routing and notifications

### Fundamental Concepts

- **Metrics**: Numerical measurements (Counter, Gauge, Histogram, Summary)
- **Labels**: Key-value pairs providing context and dimensionality
- **Time Series**: Sequence of data points for unique metric + label combination
- **Pull Model**: Prometheus actively scrapes targets at intervals

### Prometheus Architecture Benefits

- Highly available and resilient
- Local storage efficient for time-series data
- Powerful query language (PromQL)
- Flexible service discovery
- Extensive exporter ecosystem
