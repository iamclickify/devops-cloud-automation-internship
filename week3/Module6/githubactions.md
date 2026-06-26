# GitHub Actions: CI/CD Automation

## Overview

GitHub Actions is a powerful CI/CD automation platform integrated directly into GitHub. It enables you to automate build, test, and deployment processes through declarative YAML workflows triggered by repository events.

---

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Workflows & Events](#workflows--events)
3. [Jobs, Steps & Actions](#jobs-steps--actions)
4. [YAML Syntax](#yaml-syntax)
5. [GitHub Marketplace](#github-marketplace)
6. [Runners](#runners)
7. [Best Practices](#best-practices)

---

## Core Concepts

### What is GitHub Actions?

**Definition**: An automation platform that runs workflows in response to GitHub events, enabling CI/CD without external tools.

**Key Characteristics**:
- ✅ Integrated into GitHub (no separate platform)
- ✅ Event-driven automation
- ✅ Defined as code (YAML)
- ✅ Pre-built actions available
- ✅ Scalable execution environment
- ✅ Free for public repositories

### GitHub Actions Hierarchy

```
Workflow (YAML file in .github/workflows/)
├── Job 1 (runs on runner)
│   ├── Step 1 (run or uses)
│   ├── Step 2 (run or uses)
│   └── Step 3 (run or uses)
├── Job 2 (depends on Job 1)
│   ├── Step 1 (action)
│   └── Step 2 (shell command)
└── Job 3 (parallel with Job 1)
    └── Step 1 (shell command)
```

---

## Workflows & Events

### What is a Workflow?

**Definition**: A configurable automated process defined in YAML that runs in response to GitHub events.

**Characteristics**:
- Located in `.github/workflows/` directory
- One YAML file = one workflow
- Contains one or more jobs
- Event-driven execution
- Repository-scoped automation

### Events & Triggers

**Definition**: An event is any activity in your repository that can trigger a workflow.

#### Common Triggering Events

| Event | Trigger Condition | Use Case |
|-------|-------------------|----------|
| **push** | Code pushed to branch/tag | Run CI on every commit |
| **pull_request** | PR opened/updated/closed | Validate PRs before merge |
| **schedule** | Cron schedule | Nightly builds, periodic tasks |
| **workflow_dispatch** | Manual trigger from UI | On-demand jobs |
| **release** | Release published/created | Deploy on release |
| **issue_comment** | Comment on issue/PR | Automated reactions |
| **create** | Branch/tag created | Setup workflows |
| **delete** | Branch/tag deleted | Cleanup workflows |

### Event Filtering

Narrow down when workflows run using filters:

```yaml
on:
  push:
    branches:
      - main
      - develop
    paths:
      - 'src/**'
      - 'package.json'
    
  pull_request:
    branches:
      - main
```

**Filters**:
- `branches`: Specific branch names
- `paths`: Only trigger if specific files modified
- `tags`: Specific tag patterns

### Example Triggers

**On push to main**:
```yaml
on:
  push:
    branches:
      - main
```

**On pull request to main**:
```yaml
on:
  pull_request:
    branches:
      - main
```

**On schedule (nightly)**:
```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Every day at midnight UTC
```

**Manual trigger**:
```yaml
on:
  workflow_dispatch:
```

**Multiple triggers**:
```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * 0'
```

---

## Jobs, Steps & Actions

### Jobs

**Definition**: A set of steps executed sequentially on the same runner.

**Characteristics**:
- Run on isolated runners
- Execute in parallel by default
- Can be ordered using `needs`
- Each has unique identifier
- Can run on different OS

```yaml
jobs:
  build:                    # Job ID
    runs-on: ubuntu-latest  # Runner type
    steps:
      - name: Step 1
        run: echo "Building..."
  
  test:
    runs-on: ubuntu-latest
    needs: build            # Depends on 'build' job
    steps:
      - name: Step 1
        run: echo "Testing..."
```

### Steps

**Definition**: Individual tasks within a job executed sequentially.

**Characteristics**:
- Run in order defined
- Share workspace/environment
- Can be shell commands or actions
- Can have conditions
- Can use env variables

**Two Types of Steps**:

| Type | Keyword | Purpose |
|------|---------|---------|
| **Command** | `run` | Execute shell commands |
| **Action** | `uses` | Run pre-built/custom actions |

```yaml
steps:
  # Command step
  - name: Install dependencies
    run: npm install
  
  # Action step
  - name: Setup Node.js
    uses: actions/setup-node@v4
    with:
      node-version: '20'
  
  # Conditional step
  - name: Deploy
    if: github.ref == 'refs/heads/main'
    run: npm run deploy
```

### Actions

**Definition**: Reusable components that perform complex tasks, encapsulating logic and configuration.

**Types of Actions**:

| Type | Environment | Use Case |
|------|-------------|----------|
| **JavaScript** | Runner (native) | Fast execution |
| **Docker** | Container | Isolated environment |
| **Composite** | Multiple steps | Combine existing steps |

**Action Format**: `owner/repo@version`

```yaml
uses: actions/checkout@v4           # Specific tag
uses: actions/setup-node@main       # Branch (not recommended)
uses: actions/checkout@a1b2c3d      # Commit SHA (most stable)
```

### Job Dependencies

Control execution order with `needs`:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: npm run build

  test:
    runs-on: ubuntu-latest
    needs: build              # Waits for build to complete
    steps:
      - run: npm test

  deploy:
    runs-on: ubuntu-latest
    needs: [build, test]      # Waits for both
    steps:
      - run: npm run deploy
```

---

## YAML Syntax

### Basic Structure

```yaml
name: Workflow Name               # Displayed in GitHub UI

on: [push, pull_request]         # Triggers

jobs:
  job-id:                        # Job identifier
    runs-on: ubuntu-latest       # Runner type
    steps:
      - name: Step name         # Step description
        run: command here        # Shell command
```

### Key Components

| Component | Purpose | Example |
|-----------|---------|---------|
| **name** | Workflow display name | `name: CI Pipeline` |
| **on** | Events triggering workflow | `on: [push, pull_request]` |
| **jobs** | Container for all jobs | `jobs:` |
| **runs-on** | Runner/OS for job | `runs-on: ubuntu-latest` |
| **steps** | Tasks in job | `steps:` |
| **uses** | Action to run | `uses: actions/checkout@v4` |
| **run** | Shell command | `run: npm test` |
| **with** | Action inputs | `with: {node-version: '20'}` |
| **env** | Environment variables | `env: {KEY: value}` |
| **if** | Conditional execution | `if: github.ref == 'refs/heads/main'` |

### YAML Best Practices

✅ Use **spaces** (not tabs) for indentation  
✅ Consistent 2-space indentation  
✅ Pin actions to versions (`@v4`, not `@main`)  
✅ Quote strings with special characters  
✅ Use comments for clarity  
✅ Avoid hardcoding secrets  

### Common YAML Pitfalls

❌ Tab indentation (use spaces)  
❌ Inconsistent indentation levels  
❌ Typos in keywords (`on` not `on:`)  
❌ Incorrect list syntax (missing hyphens)  
❌ Unpinned action versions  

### Expressions & Contexts

Use `${{ }}` syntax for dynamic values:

```yaml
if: github.ref == 'refs/heads/main'
env:
  BUILD_ID: ${{ github.run_id }}
  COMMIT: ${{ github.sha }}
```

**Common Contexts**:
- `github.*`: Workflow/repository info
- `env.*`: Environment variables
- `runner.*`: Runner info (OS, arch)
- `secrets.*`: Repository secrets
- `matrix.*`: Matrix build values

### Example Workflow

```yaml
name: Node.js CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Build project
        run: npm run build
```

---

## GitHub Marketplace

### What is GitHub Marketplace?

**Definition**: Central repository of pre-built GitHub Actions created by GitHub, partners, and community.

**Access**: `github.com/marketplace?type=actions`

### Benefits of Using Actions

✅ Save development time  
✅ Leverage tested, reliable code  
✅ Reduce complexity  
✅ Promote standardization  
✅ Active community support  

### Essential Actions

| Action | Purpose | Example |
|--------|---------|---------|
| **actions/checkout** | Clone repository | `uses: actions/checkout@v4` |
| **actions/setup-node** | Setup Node.js | `uses: actions/setup-node@v4` |
| **actions/setup-python** | Setup Python | `uses: actions/setup-python@v5` |
| **actions/setup-java** | Setup Java | `uses: actions/setup-java@v4` |
| **docker/login-action** | Docker registry login | `uses: docker/login-action@v3` |
| **docker/build-push-action** | Build/push Docker | `uses: docker/build-push-action@v5` |
| **aws-actions/configure-aws-credentials** | AWS auth | `uses: aws-actions/configure-aws-credentials@v4` |
| **hashicorp/setup-terraform** | Setup Terraform | `uses: hashicorp/setup-terraform@v3` |
| **actions/upload-artifact** | Upload files | `uses: actions/upload-artifact@v4` |

### Action Inputs

Actions accept inputs via `with`:

```yaml
- name: Set up Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'
    architecture: 'x64'
```

### Action Versioning

**Always pin to specific version**:

```yaml
# ✅ Good: Specific version tag
uses: actions/checkout@v4

# ✅ Good: Commit SHA
uses: actions/checkout@a1b2c3d4e5f6

# ❌ Avoid: Branch (can break)
uses: actions/checkout@main
```

### Example: Multi-Action Workflow

```yaml
name: Full CI/CD Pipeline

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: pip install -r requirements.txt
      
      - name: Run tests
        run: pytest
      
      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: myapp:latest
      
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY }}
          aws-region: us-east-1
      
      - name: Deploy to AWS
        run: aws s3 sync ./build s3://my-bucket/
      
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: build/
```

---

## Runners

### What is a Runner?

**Definition**: A machine that executes workflow jobs.

**Types**:

| Type | Management | Cost | Control | Use Case |
|------|-----------|------|---------|----------|
| **GitHub-hosted** | GitHub | Free tier | Limited | Most projects |
| **Self-hosted** | You | Variable | Full | Private resources |

### GitHub-Hosted Runners

**Available OS**:
- `ubuntu-latest` (Ubuntu Linux)
- `windows-latest` (Windows Server)
- `macos-latest` (macOS)

**Specific versions**:
- `ubuntu-22.04`, `ubuntu-20.04`
- `windows-2022`, `windows-2019`
- `macos-13`, `macos-12`

**Features**:
- ✅ No setup required
- ✅ Always available
- ✅ Scalable
- ✅ Pre-installed tools
- ❌ Ephemeral (reset after job)
- ❌ Limited customization
- ❌ No private network access

### Self-Hosted Runners

**Features**:
- ✅ Full control
- ✅ Custom software/tools
- ✅ Private network access
- ✅ Cost-effective at scale
- ❌ Requires setup/maintenance
- ❌ Your responsibility to maintain

**When to Use**:
- Need private network access
- Require specific hardware
- Have high volume (cost-effective)
- Need custom tools installed
- Have strict security requirements

### Specifying Runners

```yaml
jobs:
  linux-job:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running on Linux"
  
  windows-job:
    runs-on: windows-latest
    steps:
      - run: echo "Running on Windows"
  
  macos-job:
    runs-on: macos-latest
    steps:
      - run: echo "Running on macOS"
  
  self-hosted-job:
    runs-on: [self-hosted, linux]
    steps:
      - run: echo "Running on self-hosted Linux"
```

---

## Secrets & Security

### GitHub Secrets

Store sensitive information securely:

**How to Add**:
1. Go to repository Settings
2. Click "Secrets and variables" → "Actions"
3. Click "New repository secret"
4. Add secret name and value

**Usage in Workflows**:
```yaml
- name: Configure AWS
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY }}

- name: Login to Docker
  run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }}
```

**Best Practices**:
✅ Never hardcode secrets  
✅ Use repository/organization secrets  
✅ Rotate secrets regularly  
✅ Limit secret access to necessary workflows  
✅ Use OIDC for cloud provider authentication  

---

## Best Practices

### Workflow Design

✅ Keep workflows focused (one responsibility)  
✅ Use descriptive names for workflows, jobs, steps  
✅ Add comments explaining complex logic  
✅ Break large jobs into smaller steps  
✅ Use matrix builds for multiple configurations  

### Performance

✅ Cache dependencies (npm, pip, Maven)  
✅ Parallelize independent jobs  
✅ Use conditional steps to skip unnecessary work  
✅ Minimize runner startup time  
✅ Use appropriate runner for task  

### Reliability

✅ Pin action versions to tags/SHAs  
✅ Test workflows on multiple branches  
✅ Implement proper error handling  
✅ Use status checks (required for PRs)  
✅ Monitor workflow execution times  

### Security

✅ Store secrets in GitHub Secrets  
✅ Use minimal permissions (least privilege)  
✅ Scan dependencies for vulnerabilities  
✅ Scan container images  
✅ Audit workflow access logs  
✅ Use OIDC for cloud authentication  

### Maintainability

✅ Document workflow purpose  
✅ Keep YAML files organized  
✅ Use reusable composite actions  
✅ Version control all workflows  
✅ Use conventional commit messages  

---

## Common Workflow Patterns

### Basic CI Pipeline

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm test
```

### Multi-Language Matrix

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x, 21.x]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

### Conditional Deployment

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to production
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: npm run deploy
```

### Docker Build & Push

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/myapp:latest
```

---
### Essential Keywords

| Keyword | Level | Purpose |
|---------|-------|---------|
| `name` | Workflow | Display name |
| `on` | Workflow | Triggers |
| `jobs` | Workflow | Job container |
| `runs-on` | Job | Runner type |
| `steps` | Job | Steps container |
| `uses` | Step | Action reference |
| `run` | Step | Shell command |
| `with` | Step | Action inputs |
| `env` | Any | Environment vars |
| `if` | Step/Job | Condition |

### Useful Links

- **Docs**: `docs.github.com/en/actions`
- **Marketplace**: `github.com/marketplace?type=actions`
- **Syntax**: `docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions`

---

## Summary

**GitHub Actions** provides integrated CI/CD automation:

- **Workflows**: YAML-defined automation in `.github/workflows/`
- **Events**: Repository activities that trigger workflows
- **Jobs**: Parallel or sequential execution units
- **Steps**: Individual tasks (commands or actions)
- **Actions**: Reusable components from Marketplace
- **Runners**: GitHub-hosted or self-hosted execution environments

**Key Advantages**:
- No external platform needed
- Version control integration
- Free for public repositories
- Extensive Marketplace
- Powerful automation capabilities

**Core Workflow Structure**:
```yaml
name: Workflow
on: [push, pull_request]
jobs:
  job-id:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm test
```
