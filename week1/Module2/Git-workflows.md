# Git Workflows and Branching Strategies

## Overview

Git workflows define how developers collaborate using Git branches. They provide a structured way to develop features, fix bugs, review code, and release software.

Two of the most popular workflows are:

1. Gitflow
2. GitHub Flow

---

# Gitflow Workflow

Gitflow is a structured branching strategy designed for projects with scheduled releases and multiple developers.

## Main Branches

### main

- Production-ready code
- Always stable
- Contains released versions

### develop

- Integration branch
- New features are merged here
- Acts as the working branch

---

## Supporting Branches

### Feature Branch

Used for developing new features.

Branch From:
```bash
develop
```

Merge Into:
```bash
develop
```

Example:

```bash
feature/user-authentication
feature/payment-module
```

---

### Release Branch

Used to prepare a production release.

Branch From:

```bash
develop
```

Merge Into:

```bash
main
develop
```

Example:

```bash
release/1.0.0
release/2.0.0
```

Purpose:

- Final testing
- Documentation updates
- Bug fixes

---

### Hotfix Branch

Used for urgent production fixes.

Branch From:

```bash
main
```

Merge Into:

```bash
main
develop
```

Example:

```bash
hotfix/login-error
hotfix/payment-failure
```

Purpose:

- Quickly fix production issues

---

## Gitflow Diagram

main
│
├──── release/1.0
│
├──── hotfix/login-bug
│
develop
│
├──── feature/auth
├──── feature/cart
└──── feature/payment

---

## Advantages of Gitflow

- Clear release management
- Better organization
- Suitable for large teams
- Supports long-term maintenance

## Disadvantages

- Complex workflow
- Too many branches
- Slower development cycle
- Not ideal for Continuous Delivery

---

# GitHub Flow

GitHub Flow is a lightweight workflow designed for Continuous Integration and Continuous Deployment (CI/CD).

Only one permanent branch exists:

```bash
main
```

All work happens through short-lived feature branches.

---

## Workflow

### Step 1

Create a branch from main

```bash
git checkout -b add-login-feature
```

### Step 2

Make changes and commit

```bash
git add .
git commit -m "Added login page"
```

### Step 3

Push branch

```bash
git push origin add-login-feature
```

### Step 4

Create Pull Request

PR is used for:

- Code Review
- Discussion
- Automated Testing

### Step 5

Merge into main

After approval and successful tests.

### Step 6

Deploy automatically

CI/CD pipeline deploys application.

---

# Pull Requests (PR)

Pull Requests are the core of GitHub Flow.

Purpose:

- Code review
- Team discussion
- Automated testing
- Quality assurance

Typical Flow:

Feature Branch
↓
Pull Request
↓
Code Review
↓
Tests Pass
↓
Merge to main
↓
Deploy

---

# Gitflow vs GitHub Flow

| Feature | Gitflow | GitHub Flow |
|----------|----------|-------------|
| Complexity | High | Low |
| Main Branches | main + develop | main |
| Release Branches | Yes | No |
| Hotfix Branches | Yes | No |
| CI/CD Friendly | Moderate | Excellent |
| Release Frequency | Scheduled | Continuous |
| Best For | Large Projects | Modern DevOps |

---

# DevOps Perspective

Modern DevOps teams usually prefer GitHub Flow because:

- Simpler workflow
- Faster deployments
- Better CI/CD integration
- Continuous delivery friendly

Gitflow is still useful for:

- Enterprise projects
- Large teams
- Products with fixed release schedules

---

# Key Takeaways

- Gitflow uses multiple branches for structured release management.
- GitHub Flow uses one main branch and short-lived feature branches.
- Pull Requests are essential for collaboration and code review.
- GitHub Flow is the preferred workflow in most modern DevOps environments.
- CI/CD pipelines commonly trigger after Pull Requests are merged into main.
