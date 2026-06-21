# Docker Compose: Multi-Container Application Management

## Overview

Docker Compose is a tool for defining and running multi-container Docker applications using a declarative YAML configuration file (`docker-compose.yml`). It simplifies the orchestration of multiple interconnected services, enabling consistent development, testing, and deployment workflows.

---

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Docker Compose Benefits](#docker-compose-benefits)
3. [docker-compose.yml Structure](#docker-composeyml-structure)
4. [Services Definition](#services-definition)
5. [Networks in Docker Compose](#networks-in-docker-compose)
6. [Volumes in Docker Compose](#volumes-in-docker-compose)
7. [Essential Commands](#essential-commands)
8. [Common Configurations](#common-configurations)

---

## Core Concepts

### What is Docker Compose?

Docker Compose is a tool for defining and running multi-container applications. It:
- Uses a declarative YAML file to configure services, networks, and volumes
- Creates and manages the entire application stack with single commands
- Provides service discovery and automatic networking
- Ensures consistent environments across development, testing, and production

## Docker Compose Benefits

### Development & Testing

✅ **Simplified Setup**: Entire stack with single `docker-compose up` command  
✅ **Reproducible Environments**: Identical setup across all developers  
✅ **Eliminates "Works on My Machine"**: Consistent configurations  
✅ **Efficient Testing**: Quick isolated test environments  

### Deployment & Operations

✅ **Consistent Deployments**: Same file used for dev, staging, production  
✅ **Declarative Configuration**: Define desired state, not steps  
✅ **Resource Management**: Control per-service resource allocation  
✅ **Version Control Friendly**: Store configurations in Git  

### Architecture

✅ **Microservices Support**: Define multiple services independently  
✅ **Service Discovery**: Automatic DNS between services  
✅ **Orchestration Foundation**: Concepts transfer to Swarm/Kubernetes  

---

## docker-compose.yml Structure

### Basic File Format

```yaml
version: '3.8'  # Compose file specification version

services:
  # Service definitions here
  
networks:
  # Custom network definitions here
  
volumes:
  # Named volume definitions here
```

### Top-Level Keys

| Key | Purpose |
|-----|---------|
| **version** | Specifies Compose file format version (e.g., '3.8') |
| **services** | Defines containerized application components (required) |
| **networks** | Defines custom networks for inter-service communication |
| **volumes** | Defines named volumes for persistent data storage |
| **configs** | Manages configuration files (advanced) |
| **secrets** | Manages sensitive data (advanced) |

### YAML Syntax Notes

- Use **spaces** (not tabs) for indentation
- Colons (`:`) separate keys from values
- Hyphens (`-`) denote list items
- Indentation sensitive

---

## Services Definition

Services are containerized components of your application. Each service definition controls how a container runs.

### Service Configuration Options

#### 1. **image vs build**

**Using Pre-built Image**:
```yaml
services:
  web:
    image: nginx:stable-alpine
    image: postgres:13
    image: my-custom-image:latest
```

**Building from Dockerfile**:
```yaml
services:
  app:
    build: ./app  # Directory containing Dockerfile
    
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod  # Custom Dockerfile name
```

#### 2. **ports: Port Mapping**

Maps host ports to container ports. Format: `HOST_PORT:CONTAINER_PORT`

```yaml
services:
  web:
    ports:
      - '8080:80'           # Single port mapping
      - '3000:3000'         # Multiple mappings
      - '0.0.0.0:5000:5000' # Explicit host binding
      
  api:
    expose:
      - '8000'  # Available to other services, not host
```

#### 3. **environment: Environment Variables**

Set configuration values inside containers.

**Direct Definition**:
```yaml
services:
  db:
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mysecret
      POSTGRES_DB: mydatabase
```

**From .env File**:
```yaml
services:
  db:
    env_file:
      - .env
```

Create `.env` file:
```
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mysecret
POSTGRES_DB=mydatabase
```

#### 4. **volumes: Data Persistence**

Mount volumes or bind mounts for data storage and configuration.

**Named Volumes**:
```yaml
volumes:
  db-data:
  app-logs:

services:
  db:
    volumes:
      - db-data:/var/lib/postgresql/data
  app:
    volumes:
      - app-logs:/app/logs
```

**Bind Mounts** (host paths):
```yaml
services:
  web:
    volumes:
      - ./html:/usr/share/nginx/html        # Read-write mount
      - ./config.yml:/app/config.yml:ro     # Read-only mount
```

#### 5. **command & entrypoint**

Override default commands defined in the Dockerfile.

```yaml
services:
  app:
    command: ["python", "manage.py", "runserver", "0.0.0.0:8000"]
    
  cache:
    entrypoint: ["redis-server", "--appendonly", "yes"]
```

#### 6. **restart Policy**

Define automatic container restart behavior.

```yaml
services:
  web:
    restart: always           # Always restart if stopped
    
  worker:
    restart: on-failure       # Restart only on non-zero exit
    
  job:
    restart: no               # Do not restart
    
  api:
    restart: unless-stopped   # Restart unless explicitly stopped
```

#### 7. **depends_on**

Control service startup order (ensures startup, not readiness).

```yaml
services:
  api:
    depends_on:
      - db          # Database must start before API
      - cache       # Cache must start before API
    environment:
      DATABASE_URL: postgresql://user:pass@db:5432/mydb
      
  db:
    image: postgres:13
    
  cache:
    image: redis:latest
```

**Note**: `depends_on` only guarantees startup order, not that services are ready. Use health checks for reliability.

#### 8. **Health Checks**

Verify service readiness.

```yaml
services:
  db:
    image: postgres:13
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
```

---

## Networks in Docker Compose

### Default Network Behavior

By default, Docker Compose:
- Creates a single network named `PROJECT_NAME_default`
- Attaches all services to this network
- Enables automatic DNS resolution by service name
- Isolates containers from other Docker networks

### Service Discovery

Services communicate using service names as hostnames:
```yaml
services:
  api:
    environment:
      DATABASE_HOST: db  # Reference by service name
  db:
    image: postgres:13
```

### Custom Networks

Define separate networks for segmentation and enhanced control.

```yaml
services:
  frontend:
    image: my-frontend
    networks:
      - frontend-net

  api-gateway:
    image: my-gateway
    networks:
      - frontend-net
      - backend-net

  backend:
    image: my-backend
    networks:
      - backend-net

  db:
    image: postgres:13
    networks:
      - backend-net

networks:
  frontend-net:
    driver: bridge
  backend-net:
    driver: bridge
```

**Benefits**:
- ✅ Network isolation between frontend and backend
- ✅ API gateway bridges both networks
- ✅ Frontend cannot access backend directly
- ✅ Enhanced security

### Network Drivers

| Driver | Use Case | Features |
|--------|----------|----------|
| **bridge** | Default, single host | Private network, NAT |
| **host** | Performance-critical | No isolation, direct access |
| **overlay** | Multi-host (Swarm/K8s) | Cross-node communication |

### External Networks

Connect to networks created outside Compose.

```yaml
networks:
  external-net:
    external: true

services:
  app:
    networks:
      - external-net
```

---

## Volumes in Docker Compose

### Named Volumes (Recommended)

Docker-managed persistent storage.

```yaml
volumes:
  db-data:
  cache-data:
  logs:

services:
  db:
    image: postgres:13
    volumes:
      - db-data:/var/lib/postgresql/data
      
  redis:
    image: redis:latest
    volumes:
      - cache-data:/data
      
  app:
    volumes:
      - logs:/app/logs
```

**Advantages**:
- ✅ Docker-managed lifecycle
- ✅ Survives container removal
- ✅ Portable and shareable
- ✅ Optimized performance

### Bind Mounts (Development)

Direct host path mounts (often for development).

```yaml
services:
  app:
    volumes:
      - ./src:/app/src           # Code directory
      - ./config:/app/config:ro  # Read-only config
```

**Use Cases**:
- Live code editing
- Configuration injection
- Development workflows

### Volume Management

Docker CLI commands for volume operations:

```bash
# List all volumes
docker volume ls

# Inspect volume details
docker volume inspect volume-name

# Remove volume (deletes data!)
docker volume rm volume-name

# Remove all unused volumes
docker volume prune
```

### Removing Volumes with Compose

```bash
# Default: keeps volumes
docker-compose down

# Remove volumes along with containers
docker-compose down -v
```

---

## Essential Commands

### docker-compose up

Start all services.

```bash
# Foreground (attach to logs)
docker-compose up

# Background (detached)
docker-compose up -d

# Rebuild and recreate services
docker-compose up -d --force-recreate --build

# Build without starting
docker-compose build
```

**Process**:
1. Parse `docker-compose.yml`
2. Create networks and volumes (if don't exist)
3. Pull/build images
4. Create and start containers
5. Stream logs (unless -d)

### docker-compose down

Stop and remove all services.

```bash
# Stop containers and remove networks (keep volumes)
docker-compose down

# Also remove volumes
docker-compose down -v

# Also remove images
docker-compose down --rmi all
```

### docker-compose ps

Check service status.

```bash
# List all services and their status
docker-compose ps

# Shows: Name, Command, State, Ports
```

### docker-compose logs

View container output.

```bash
# All service logs
docker-compose logs

# Specific service logs
docker-compose logs db
docker-compose logs web api

# Stream logs in real-time
docker-compose logs -f

# Stream specific service
docker-compose logs -f db

# Show timestamps
docker-compose logs --timestamps
```

### Additional Useful Commands

```bash
# Execute command in running container
docker-compose exec service-name command

# Restart services
docker-compose restart
docker-compose restart web db

# Stop services (without removing)
docker-compose stop

# Start stopped services
docker-compose start

# View service logs
docker-compose logs

# Validate docker-compose.yml syntax
docker-compose config
```

---

## Common Configurations

### Web Application with Database

Classic multi-tier architecture.

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - '80:80'
    volumes:
      - ./frontend/build:/usr/share/nginx/html
    depends_on:
      - api

  api:
    build: ./backend
    ports:
      - '5000:5000'
    environment:
      DATABASE_URL: postgresql://user:password@db:5432/mydb
    depends_on:
      - db

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

### Development with Hot Reload

Live code reloading for development.

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - '5000:5000'
    volumes:
      - .:/app  # Mount entire project
    command: python app.py
    # For Node.js with nodemon:
    # command: nodemon app.js
```

### Microservices with Network Segmentation

Multiple services on separate networks.

```yaml
version: '3.8'

services:
  frontend:
    image: my-frontend
    ports:
      - '3000:3000'
    networks:
      - frontend-net

  api-gateway:
    image: my-gateway
    ports:
      - '80:80'
    networks:
      - frontend-net
      - backend-net

  user-service:
    image: my-user-service
    networks:
      - backend-net

  product-service:
    image: my-product-service
    networks:
      - backend-net

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend-net

networks:
  frontend-net:
    driver: bridge
  backend-net:
    driver: bridge

volumes:
  db-data:
```

---
