# Ansible Fundamentals

## Overview

Ansible is an agentless, open-source automation and configuration management platform that simplifies IT operations through declarative automation. It enables infrastructure provisioning, application deployment, and configuration management at scale without requiring agents on target machines.

---

## Table of Contents

1. [What is Ansible](#what-is-ansible)
2. [Core Concepts](#core-concepts)
3. [Architecture](#architecture)
4. [Inventory](#inventory)
5. [Playbooks](#playbooks)
6. [Modules](#modules)
7. [Variables & Facts](#variables--facts)
8. [Handlers & Conditionals](#handlers--conditionals)
9. [Roles](#roles)
10. [Best Practices](#best-practices)

---

## What is Ansible

### Definition

**Ansible** is an infrastructure automation tool that uses simple YAML-based playbooks to configure systems, deploy software, and orchestrate complex IT tasks across any number of servers.

### Key Characteristics

✅ **Agentless** - No software required on target machines (SSH-based)  
✅ **Idempotent** - Safe to run multiple times  
✅ **Declarative** - Describe desired state, not steps  
✅ **Simple syntax** - YAML-based, human-readable  
✅ **Agentless** - Uses standard protocols (SSH, WinRM)  
✅ **Extensible** - Custom modules and plugins  
✅ **Cross-platform** - Works on Linux, Windows, cloud, network devices  

### Why Ansible?

| Use Case | Benefit |
|----------|---------|
| **Configuration Management** | Ensure consistent system configuration |
| **Application Deployment** | Automate application rollout |
| **Provisioning** | Automatically set up infrastructure |
| **Orchestration** | Coordinate complex multi-system changes |
| **Cloud Management** | Work across AWS, Azure, GCP, etc. |
| **Network Automation** | Manage network devices |

---

## Core Concepts

### Control Node

**Machine where Ansible runs**

Requirements:
- Python 2.7+ or Python 3.5+
- SSH client (for Linux/Unix targets)
- Ansible package installed

```bash
# Install Ansible
pip install ansible

# Verify installation
ansible --version
```

### Managed Nodes

**Target machines to be automated**

Requirements:
- ✅ SSH access (Linux/Unix)
- ✅ WinRM access (Windows)
- ✅ Python 2.6+ or Python 3.5+ (for most modules)
- ✅ Sudo access (for privileged operations)

**No Ansible software needed!**

### Plays & Playbooks

**Play**: Single configuration unit applying to specific hosts

**Playbook**: Collection of plays (YAML file)

```yaml
---
# This is a playbook with one play
- name: Configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
```

### Tasks & Modules

**Task**: Individual unit of work using a module

**Module**: Ansible's building block; reusable code doing specific work

```yaml
- name: Install nginx
  apt:                    # Module name
    name: nginx           # Module argument
    state: present        # Module argument
```

### Handlers

**Special tasks triggered by other tasks**

```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
  notify: Restart nginx   # Trigger handler

handlers:
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

---

## Architecture

### Ansible Execution Flow

```
┌────────────┐
│  Control   │
│   Node     │
│            │
│ • Ansible  │
│ • Python   │
│ • Modules  │
└─────┬──────┘
      │ SSH/WinRM
      │
┌─────┴──────────────────────────────────┐
│                                         │
│    Managed Nodes (No agent needed)     │
│                                         │
│  ┌──────────────┐  ┌──────────────┐   │
│  │   Server 1   │  │   Server 2   │   │
│  │  • Linux     │  │  • Windows   │   │
│  │  • Python    │  │  • Python    │   │
│  └──────────────┘  └──────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

### Connection Methods

| Method | Protocol | Use Case |
|--------|----------|----------|
| **ssh** | SSH | Linux/Unix remote execution |
| **local** | Local | Run on control node |
| **winrm** | WinRM | Windows remote execution |
| **docker** | Docker | Run in containers |
| **raw** | Minimal | Basic commands |

---

## Inventory

### What is Inventory?

**File listing all hosts/machines** Ansible can manage

### Inventory Formats

**INI Format**:
```ini
[webservers]
web1.example.com
web2.example.com ansible_user=ubuntu

[databases]
db1.example.com ansible_host=192.168.1.10
db2.example.com ansible_host=192.168.1.11

[databases:vars]
ansible_user=admin
ansible_port=22
```

**YAML Format**:
```yaml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
          ansible_user: ubuntu
    databases:
      hosts:
        db1.example.com:
          ansible_host: 192.168.1.10
      vars:
        ansible_user: admin
        ansible_port: 22
```

### Inventory Variables

**Host Variables**:
```ini
web1.example.com ansible_user=ubuntu ansible_port=2222
```

**Group Variables**:
```ini
[webservers:vars]
ansible_user=ubuntu
ansible_port=22
nginx_port=80
```

**Special Variables**:
- `ansible_host` - IP/hostname
- `ansible_port` - SSH port
- `ansible_user` - Remote user
- `ansible_password` - Password (use vault!)
- `ansible_become` - Enable privilege escalation
- `ansible_become_method` - sudo, su, etc.

### Dynamic Inventory

**Inventory from external sources**:

```python
#!/usr/bin/env python
import json

inventory = {
    "webservers": {
        "hosts": ["192.168.1.1", "192.168.1.2"],
        "vars": {
            "ansible_user": "ubuntu"
        }
    },
    "_meta": {
        "hostvars": {}
    }
}

print(json.dumps(inventory))
```

---

## Playbooks

### Playbook Structure

```yaml
---
- name: Deploy web application
  hosts: webservers
  become: yes
  gather_facts: yes
  vars:
    app_port: 8080
    app_user: appuser
  
  pre_tasks:
    - name: Pre-deployment task
      debug:
        msg: "Starting deployment"
  
  tasks:
    - name: Install dependencies
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - python3
        - python3-pip
    
    - name: Clone repository
      git:
        repo: https://github.com/example/app.git
        dest: /opt/app
        version: main
    
    - name: Install Python packages
      pip:
        requirements: /opt/app/requirements.txt
        executable: pip3
  
  post_tasks:
    - name: Post-deployment task
      debug:
        msg: "Deployment complete"
  
  handlers:
    - name: Restart service
      service:
        name: myapp
        state: restarted
```

### Playbook Sections

| Section | Purpose | When Runs |
|---------|---------|-----------|
| **pre_tasks** | Initial setup | Before main tasks |
| **tasks** | Main work | After pre_tasks |
| **post_tasks** | Cleanup | After tasks |
| **handlers** | Triggered actions | When notified |

### Common Playbook Directives

```yaml
- name: Example play
  hosts: webservers           # Target hosts
  become: yes                 # Run as root
  become_user: www-data       # Run as specific user
  gather_facts: yes           # Collect host facts
  serial: 1                   # Run one host at a time
  max_fail_percentage: 10     # Max failure %
  vars:                       # Play variables
    my_var: value
  tags:                       # Label tasks
    - deploy
    - important
```

---

## Modules

### What are Modules?

**Modules** are Ansible's core tools - reusable scripts doing specific work

### Common Modules

| Module | Purpose |
|--------|---------|
| **apt/yum** | Install packages |
| **service** | Manage services |
| **copy** | Copy files |
| **template** | Deploy template files |
| **user** | Manage users |
| **file** | Manage files/directories |
| **command/shell** | Execute commands |
| **debug** | Output information |
| **git** | Clone repositories |
| **docker_container** | Manage containers |
| **aws_ec2** | Manage AWS resources |
| **lineinfile** | Modify file lines |
| **block** | Group tasks |

### Module Examples

**Package Installation**:
```yaml
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - git
```

**Service Management**:
```yaml
- name: Start and enable nginx
  service:
    name: nginx
    state: started
    enabled: yes
```

**File Operations**:
```yaml
- name: Create directory
  file:
    path: /opt/app
    state: directory
    owner: appuser
    group: appuser
    mode: '0755'
```

**Execute Commands**:
```yaml
- name: Run custom script
  shell: /opt/scripts/deploy.sh
  args:
    executable: /bin/bash
```

**Template Deployment**:
```yaml
- name: Deploy config file
  template:
    src: nginx.conf.j2      # Source template
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'
  notify: Restart nginx
```

---

## Variables & Facts

### Variable Types

**Play Variables**:
```yaml
- name: Example
  hosts: all
  vars:
    app_name: myapp
    app_port: 8080
  tasks:
    - name: Print var
      debug:
        msg: "App {{ app_name }} runs on {{ app_port }}"
```

**Variables File** (vars/main.yml):
```yaml
---
app_name: myapp
app_port: 8080
database_host: db.example.com
```

**Load Variables**:
```yaml
- name: Load variables
  hosts: all
  vars_files:
    - vars/main.yml
    - vars/{{ ansible_os_family }}.yml
  tasks:
    - debug:
        msg: "{{ app_name }}"
```

**Command-Line Variables**:
```bash
ansible-playbook playbook.yml -e "app_name=myapp app_port=8080"
```

### Facts

**System information** gathered from hosts

```yaml
- name: Show facts
  hosts: all
  tasks:
    - name: Print IP address
      debug:
        msg: "IP: {{ ansible_default_ipv4.address }}"
    
    - name: Print OS
      debug:
        msg: "OS: {{ ansible_os_family }}"
```

**Common Facts**:
- `ansible_os_family` - OS type (Debian, RedHat, etc.)
- `ansible_distribution` - Distribution name (Ubuntu, CentOS)
- `ansible_default_ipv4.address` - IP address
- `ansible_hostname` - Hostname
- `ansible_processor_vcpus` - CPU count
- `ansible_memtotal_mb` - Total memory

**Disable fact gathering**:
```yaml
- name: Fast playbook
  hosts: all
  gather_facts: no
  tasks:
    - debug:
        msg: "No facts needed"
```

---

## Handlers & Conditionals

### Handlers

**Tasks triggered by notifications**

```yaml
- name: Install and configure nginx
  hosts: webservers
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
      notify: Restart nginx
    
    - name: Deploy config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx
  
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

**Handler Run Order**:
- Handlers run at end of play
- Each handler runs once (deduplicated)
- Execute in order defined

### Conditionals

**when Statement** - Execute task conditionally

```yaml
- name: Install if Debian
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"

- name: Install if not installed
  apt:
    name: nginx
    state: present
  when: '"nginx" not in ansible_facts.packages'

- name: Install if variable set
  apt:
    name: "{{ package }}"
    state: present
  when: package is defined

- name: Multiple conditions (AND)
  service:
    name: nginx
    state: restarted
  when:
    - ansible_os_family == "Debian"
    - nginx_installed | bool
```

### Loops

**Repeat tasks multiple times**

```yaml
# Simple loop
- name: Install packages
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - git

# Loop over list variable
- name: Create users
  user:
    name: "{{ item }}"
    state: present
  loop: "{{ users }}"

# Loop with index
- name: Configure services
  service:
    name: "{{ item.name }}"
    port: "{{ item.port }}"
    enabled: yes
  loop:
    - { name: nginx, port: 80 }
    - { name: postgresql, port: 5432 }
```

---

## Roles

### What are Roles?

**Roles** organize related tasks, files, templates, and variables into reusable packages

### Role Structure

```
roles/
├── webserver/
│   ├── tasks/
│   │   └── main.yml          # Tasks
│   ├── handlers/
│   │   └── main.yml          # Handlers
│   ├── templates/
│   │   └── nginx.conf.j2     # Jinja2 templates
│   ├── files/
│   │   └── app.conf          # Static files
│   ├── vars/
│   │   └── main.yml          # Role variables
│   ├── defaults/
│   │   └── main.yml          # Default variables
│   ├── meta/
│   │   └── main.yml          # Role metadata
│   └── README.md             # Documentation
```

### Using Roles

```yaml
---
- name: Deploy web application
  hosts: webservers
  roles:
    - role: webserver
      vars:
        nginx_port: 8080
    - role: monitoring
```

### Role Best Practices

✅ **Single responsibility** - One role per function  
✅ **Reusable** - Work across projects  
✅ **Well documented** - README and examples  
✅ **Parameterized** - Use variables for flexibility  
✅ **Tested** - Validate with different configurations  

---

## Best Practices

### Playbook Organization

✅ **Use roles** - Structure complex playbooks  
✅ **Clear naming** - Descriptive task/role names  
✅ **Comments** - Explain complex logic  
✅ **Variables** - Externalize configuration  
✅ **Tags** - Allow selective execution  

### Idempotency

✅ **Design for idempotency** - Safe to run repeatedly  
✅ **Use state parameters** - present, absent, etc.  
✅ **Check before creating** - Avoid unnecessary changes  

### Security

✅ **Use vault** - Encrypt sensitive data
```bash
ansible-vault encrypt vars/secrets.yml
ansible-playbook playbook.yml --ask-vault-pass
```

✅ **Avoid passwords** - Use SSH keys  
✅ **Limit access** - Restrict user permissions  
✅ **Audit changes** - Log all modifications  

### Performance

✅ **Parallel execution** - Use `forks`  
✅ **Gather facts selectively** - Only when needed  
✅ **Use include** - Load playbooks conditionally  
✅ **Cache facts** - Reuse system information  

### Testing

✅ **Dry-run** - Check before applying
```bash
ansible-playbook playbook.yml --check
```

✅ **Syntax check** - Validate YAML
```bash
ansible-playbook playbook.yml --syntax-check
```

✅ **Test in staging** - Before production  

---

## Quick Reference

### Common Commands

```bash
# Ping hosts
ansible all -i inventory.ini -m ping

# Gather facts
ansible webservers -i inventory.ini -m setup

# Run playbook
ansible-playbook playbook.yml -i inventory.ini

# Dry run
ansible-playbook playbook.yml -i inventory.ini --check

# Run specific tags
ansible-playbook playbook.yml --tags "deploy"

# Run with extra variables
ansible-playbook playbook.yml -e "env=prod"

# Show verbose output
ansible-playbook playbook.yml -v
```

### Playbook Template

```yaml
---
- name: Configure servers
  hosts: all
  become: yes
  gather_facts: yes
  vars:
    app_name: myapp
  
  tasks:
    - name: Update packages
      apt:
        update_cache: yes
    
    - name: Install dependencies
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - git
        - curl
  
  handlers:
    - name: Restart service
      service:
        name: myapp
        state: restarted
```

---

## Summary

**Ansible** provides:

✅ **Agentless automation** - SSH-based, simple setup  
✅ **Declarative configuration** - YAML-based, human-readable  
✅ **Idempotent operations** - Safe to run repeatedly  
✅ **Scalable** - Manage hundreds/thousands of servers  
✅ **Flexible** - Works everywhere: Linux, Windows, cloud, network  
✅ **Extensible** - Custom modules and plugins  

**Core concepts**:
- **Playbooks**: Automation files in YAML
- **Inventory**: List of target machines
- **Modules**: Building blocks for tasks
- **Variables**: Configuration parameters
- **Handlers**: Triggered actions
- **Roles**: Reusable automation packages

---
