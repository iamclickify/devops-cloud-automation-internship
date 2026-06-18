# Docker Networking & Volumes: Advanced Guide

## Overview

Docker networking enables seamless inter-container communication, while volumes provide persistent data storage. Mastering these capabilities is essential for building scalable, stateful applications and managing complex containerized architectures.

---

## Table of Contents

1. [Docker Networking Fundamentals](#docker-networking-fundamentals)
2. [Built-in Network Drivers](#built-in-network-drivers)
3. [Custom Bridge Networks](#custom-bridge-networks)
4. [Advanced Container Connectivity](#advanced-container-connectivity)
5. [Docker Volumes](#docker-volumes)
6. [Bind Mounts vs Volumes](#bind-mounts-vs-volumes)
7. [Essential Commands](#essential-commands)

---

## Docker Networking Fundamentals

Docker networking allows containers to communicate with each other, with the host, and with external networks. Understanding network drivers is crucial for designing secure, efficient container architectures.

### Key Concepts

- **Network Isolation**: Containers can be isolated or connected based on business logic
- **Service Discovery**: Containers can communicate using DNS resolution
- **Port Mapping**: Control external access to container services
- **Multi-Network**: Containers can connect to multiple networks simultaneously

---

## Built-in Network Drivers

### 1. Bridge Network Driver (Default)

**Description**: Creates a private internal network on the Docker host.

**How it Works**:
- Docker creates a virtual bridge interface on the host
- Each container gets its own virtual network interface
- IP addresses assigned from private subnet
- Containers communicate via NAT (Network Address Translation)

**Characteristics**:
- ✅ Default network for containers
- ✅ Good for general-purpose networking
- ✅ Port mapping support
- ❌ Default bridge has limited DNS resolution
- ❌ Harder to manage as container count grows

**When to Use**: General-purpose applications needing inter-container communication

### 2. Host Network Driver

**Description**: Removes network isolation; container shares host's network namespace.

**How it Works**:
- Container shares host's network stack
- No virtual network interfaces
- Container uses host's IP address directly
- Ports directly exposed on host

**Characteristics**:
- ✅ Better network performance
- ✅ Direct access to host network interfaces
- ✅ Useful for network-intensive services
- ❌ No isolation (security risk)
- ❌ Port conflicts possible
- ❌ Reduced security

**When to Use**: Performance-critical applications, VPN clients, network services

### 3. None Network Driver

**Description**: Isolates container from all network connectivity.

**How it Works**:
- Container has network stack but no interfaces
- Only loopback (lo) interface available
- Complete network isolation

**Characteristics**:
- ✅ Maximum isolation
- ✅ Security-sensitive tasks
- ❌ No external connectivity
- ❌ No inter-container communication

**When to Use**: Batch processing jobs, security hardening, containers without network needs

### Network Driver Comparison

| Feature | Bridge | Host | None |
|---------|--------|------|------|
| **Isolation** | Good | None | Complete |
| **Inter-container Comm** | Yes | Yes | No |
| **Performance** | Good | Best | N/A |
| **Port Mapping** | Yes | N/A | N/A |
| **Security** | Better | Poor | Excellent |
| **Use Case** | Default/General | Performance | No Network |

---

## Custom Bridge Networks

While the default bridge network works, **user-defined bridge networks** offer significant advantages:

### Advantages of Custom Bridge Networks

| Feature | Default Bridge | Custom Bridge |
|---------|----------------|---------------|
| **DNS Resolution** | Limited | Automatic by container name |
| **Isolation** | Partial | Complete |
| **Management** | Rigid | Flexible |
| **Network Config** | Fixed | Customizable |

### Creating Custom Bridge Networks

**Basic Creation**:
```bash
docker network create my-custom-bridge-network
```

**With Specific Configuration**:
```bash
docker network create \
  --driver bridge \
  --subnet=172.28.0.0/16 \
  --gateway=172.28.0.1 \
  my-app-network
```

**Parameters**:
- `--driver bridge`: Specifies bridge driver (default)
- `--subnet=172.28.0.0/16`: IP address range
- `--gateway=172.28.0.1`: Gateway IP address
- `--label`: Add metadata labels for organization

### Managing Custom Networks

**List Networks**:
```bash
docker network ls
```

**Inspect Network Details**:
```bash
docker network inspect my-custom-bridge-network
```
Returns JSON with network configuration, connected containers, and IP assignments.

**Remove Network**:
```bash
docker network rm my-custom-bridge-network
```
⚠️ **Cannot remove networks with attached containers**. Disconnect containers first.

---

## Advanced Container Connectivity

### Connecting Existing Containers to Networks

**Connect a Running Container**:
```bash
docker network connect my-app-net my-existing-container
```

**Disconnect a Container**:
```bash
docker network disconnect my-app-net my-existing-container
```

### Multiple Networks

A container can connect to multiple networks simultaneously:

**At Runtime**:
```bash
docker run -d --name multi-net-container \
  --network network-a \
  --network network-b \
  my-image
```

**For Existing Container**:
```bash
docker network connect network-c multi-net-container
docker network connect network-d multi-net-container
```

**Result**: Container gets separate IP address on each network

### Network Aliases

Assign alternative names to containers for service discovery:

**At Runtime**:
```bash
docker run -d --name webserver \
  --network my-app-net \
  --network-alias web \
  nginx
```

**For Existing Container**:
```bash
docker network connect --alias api-gateway api-network my-gateway-container
```

**Use Cases**:
- Multiple service instances (load balancing)
- Service discovery
- Flexible naming conventions

### Advanced Use Cases

| Scenario | Solution |
|----------|----------|
| **Microservices** | Separate networks for isolation + aliases for communication |
| **Development** | Multiple local services on custom networks |
| **Security** | Restricted networks for sensitive services |

---

## Docker Volumes

Docker volumes are the **preferred mechanism** for persistent data storage in containers.

### What are Docker Volumes?

**Definition**: Docker-managed storage outside the container's writable layer, stored in dedicated host location.

**Location**: Typically `/var/lib/docker/volumes/` on Linux

**Key Feature**: Data persists even when containers are stopped, removed, or replaced

### Why Volumes Matter

| Benefit | Impact |
|---------|--------|
| **Data Persistence** | Survives container removal/restart |
| **Data Sharing** | Multiple containers access same data |
| **Decoupling** | Separates code from data lifecycle |
| **Management** | Docker handles backup/migration |
| **Performance** | Optimized I/O operations |

### How Volumes Work

1. **Creation**: Volume created (if doesn't exist)
2. **Mounting**: Volume mounted to container directory
3. **Storage**: Data written to mount point persists in volume
4. **Retrieval**: Data available when container restarts
5. **Lifecycle**: Independent of container lifecycle

### Volume Use Cases

| Use Case | Benefit |
|----------|---------|
| **Databases** | PostgreSQL, MySQL data directories |
| **Configuration** | Persistent app settings |
| **Logs** | Application log aggregation |
| **User Uploads** | User-generated content |
| **Caching** | Performance optimization |

### Managing Volumes

**List Volumes**:
```bash
docker volume ls
docker volume ls -f name=pg  # Filter by name
```

**Create Volume**:
```bash
docker volume create my-new-volume

# With labels and options
docker volume create \
  --driver local \
  --label environment=production \
  --label app=my-app \
  my-labeled-volume
```

**Inspect Volume**:
```bash
docker volume inspect my-labeled-volume
```

Returns:
- `Name`: Volume name
- `Driver`: Storage driver
- `Mountpoint`: Host filesystem path
- `Labels`: Metadata
- `Options`: Creation options

**Remove Volume**:
```bash
docker volume rm my-new-volume
```

**Prune Unused Volumes**:
```bash
docker volume prune  # Remove all unused volumes
```

### Using Volumes with Containers

**Mount at Runtime**:
```bash
docker run -v volume-name:/container/path my-image
docker run --mount source=volume-name,target=/container/path my-image
```

**Mount Syntax**:
- `-v VOLUME_NAME:CONTAINER_PATH` (concise)
- `--mount type=volume,source=VOLUME_NAME,target=CONTAINER_PATH` (explicit)



## Bind Mounts vs Volumes

### What are Bind Mounts?

**Definition**: Direct mount of host filesystem path into container

**Syntax**:
```bash
docker run -v /path/on/host:/path/in/container my-image
docker run --mount type=bind,source=/path/on/host,target=/path/in/container my-image
```

### Detailed Comparison

| Aspect | Docker Volumes | Bind Mounts |
|--------|----------------|------------|
| **Management** | Docker-managed | Host-dependent |
| **Storage Location** | `/var/lib/docker/volumes/` | Anywhere on host |
| **Portability** | Cross-platform | Host-specific |
| **Performance** | Optimized | Variable |
| **Initialization** | Can auto-populate | No initialization |
| **Use Case** | Production data | Development |
| **Security** | Contained | Needs care |
| **Backup** | Easy | Manual |

### Docker Volumes: Best For

Persistent databases 
Production environments  
Data durability critical  
Cross-platform deployments  
Complex backup/migration  
I/O-intensive operations  

### Bind Mounts: Best For

Development workflows  
Code editing on host  
Immediate feedback  
Host-specific configs  
Docker socket sharing  
Rapid iteration  

## Essential Commands

### Network Commands

```bash
# List networks
docker network ls

# Create network
docker network create my-network
docker network create --subnet=172.28.0.0/16 my-network

# Inspect network
docker network inspect my-network

# Connect container to network
docker network connect my-network container-name

# Disconnect container from network
docker network disconnect my-network container-name

# Remove network
docker network rm my-network
```

### Volume Commands

```bash
# List volumes
docker volume ls
docker volume ls -f name=pattern

# Create volume
docker volume create my-volume
docker volume create --label env=prod my-volume

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Prune unused volumes
docker volume prune

# Mount volume at runtime
docker run -v my-volume:/container/path my-image
docker run --mount source=my-volume,target=/container/path my-image
```

### Container with Networks & Volumes

```bash
# Container with custom network and volume
docker run -d \
  --name my-app \
  --network my-app-net \
  -v my-data:/data \
  my-image

# Multiple networks
docker run -d \
  --name my-app \
  --network network-a \
  --network network-b \
  my-image
```
