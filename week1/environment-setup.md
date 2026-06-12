# Environment Setup and DevOps Toolchain

## Objective

The goal of this module was to prepare the local development environment required for the DevOps internship by installing and verifying the core tools used throughout the course.

---

# Why These Tools Matter

A DevOps workflow relies heavily on three foundational tools:

| Tool | Purpose |
|--------|----------|
| Git | Version Control System |
| Docker | Containerization Platform |
| VS Code | Development Environment |

Together they form the base of modern DevOps workflows.

---

# Git Fundamentals

## What is Git?

Git is a Distributed Version Control System (DVCS) used to:

- Track code changes
- Collaborate with teams
- Maintain project history
- Create branches for feature development
- Enable CI/CD workflows

## Important Git Concepts

### Repository

A project and its version history.

### Commit

A snapshot of changes at a specific point in time.

### Branch

An isolated line of development.

### Merge

Combining changes from multiple branches.

### Clone

Creating a local copy of a remote repository.

### Push

Uploading local commits to a remote repository.

### Pull

Fetching updates from a remote repository.

---

# Git Installation Verification

```bash
git --version
```

Example:

```bash
git version 2.x.x
```

### Configure Git

```bash
git config --global user.name "Your Name"

git config --global user.email "your@email.com"
```

### Verify Configuration

```bash
git config --global --list
```

---

# Basic Git Workflow

Initialize repository:

```bash
git init
```

Check status:

```bash
git status
```

Add files:

```bash
git add .
```

Create commit:

```bash
git commit -m "Initial Commit"
```

View history:

```bash
git log
```

---

# Docker Fundamentals

## What is Docker?

Docker is a containerization platform that packages:

- Application code
- Dependencies
- Runtime environment

into lightweight containers.

This ensures applications behave consistently across different environments.

---

# Important Docker Concepts

## Image

Blueprint used to create containers.

Example:

```bash
nginx
ubuntu
python
```

## Container

Running instance of an image.

## Dockerfile

Instructions used to build an image.

## Docker Hub

Online registry containing Docker images.

## Docker Compose

Tool for managing multi-container applications.

---

# Docker Installation Verification

Check Docker version:

```bash
docker --version
```

Check Docker engine:

```bash
docker info
```

Check Docker Compose:

```bash
docker compose version
```

---

# Running First Container

Run Nginx:

```bash
docker run --name my-nginx -p 8080:80 -d nginx
```

Check running containers:

```bash
docker ps
```

Open browser:

```
http://localhost:8080
```

Expected Result:

Nginx Welcome Page

---

# Useful Docker Commands

List containers:

```bash
docker ps -a
```

List images:

```bash
docker images
```

Pull image:

```bash
docker pull alpine
```

Stop container:

```bash
docker stop my-nginx
```

Remove container:

```bash
docker rm my-nginx
```

---

# Visual Studio Code

## What is VS Code?

VS Code is a lightweight and highly extensible code editor used for:

- Development
- Debugging
- Git integration
- Docker management
- Infrastructure as Code

---

# Essential DevOps Extensions

## Docker

Provides:

- Dockerfile support
- Container management
- Image management

## GitLens

Provides:

- Git history
- Commit tracking
- Author insights

## YAML Support

Provides:

- YAML validation
- Syntax highlighting
- Auto-completion

## Prettier

Provides:

- Automatic code formatting

---

# Installation Verification

Inside VS Code terminal:

```bash
git --version
docker --version
```

Verify:

- GitLens loaded
- Docker extension loaded
- YAML extension active
- Prettier formatting works

