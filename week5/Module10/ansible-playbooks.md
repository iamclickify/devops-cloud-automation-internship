# Ansible Playbooks: Advanced Concepts & Patterns

## Overview

Ansible playbooks are the core mechanism for orchestrating complex automation tasks. This guide covers advanced playbook patterns, best practices, and real-world scenarios for production-grade infrastructure automation.

---

## Table of Contents

1. [Playbook Architecture](#playbook-architecture)
2. [Advanced Task Control](#advanced-task-control)
3. [Error Handling & Recovery](#error-handling--recovery)
4. [Async & Parallel Execution](#async--parallel-execution)
5. [Blocks & Exception Handling](#blocks--exception-handling)
6. [Include & Import](#include--import)
7. [Templating & Jinja2](#templating--jinja2)
8. [Delegation & Local Actions](#delegation--local-actions)
9. [Advanced Patterns](#advanced-patterns)
10. [Playbook Testing](#playbook-testing)

---

## Playbook Architecture

### Multi-Play Structure

**Multiple plays in single playbook**:

```yaml
---
# Play 1: Configure database servers
- name: Configure database servers
  hosts: databases
  become: yes
  tasks:
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present

# Play 2: Configure web servers
- name: Configure web servers
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present

# Play 3: Test connectivity
- name: Verify connectivity
  hosts: all
  gather_facts: no
  tasks:
    - name: Ping all hosts
      ping:
```

### Play Structure Deep-Dive

```yaml
---
- name: Deploy application
  # Target configuration
  hosts: webservers
  port: 22
  remote_user: ubuntu
  become: yes
  become_user: root
  become_method: sudo
  
  # Variables
  vars:
    app_version: 1.2.3
    app_port: 8080
  vars_files:
    - vars/main.yml
    - vars/{{ environment }}.yml
  
  # Fact gathering
  gather_facts: yes
  gather_subset: all
  
  # Execution control
  serial: 1                    # One host at a time
  max_fail_percentage: 10      # Max 10% failures
  force_handlers: yes          # Run handlers on failure
  strategy: linear             # Execution strategy
  
  # Pre/post processing
  pre_tasks:
    - name: Pre-deployment checks
      debug:
        msg: "Starting deployment"
  
  roles:
    - role: webserver
      vars:
        nginx_port: "{{ app_port }}"
  
  tasks:
    - name: Main tasks
      debug:
        msg: "Deploy application"
  
  post_tasks:
    - name: Post-deployment validation
      debug:
        msg: "Deployment complete"
  
  handlers:
    - name: Restart service
      service:
        name: myapp
        state: restarted
```

### Execution Strategies

| Strategy | Behavior |
|----------|----------|
| **linear** | Default; one task per host before next task |
| **free** | Each host runs tasks independently |
| **batch** | Run tasks on batch of hosts sequentially |
| **debug** | Execute one task at a time, pause for inspection |

**Free strategy example** (faster for independent tasks):
```yaml
- name: Deploy to multiple servers
  hosts: webservers
  strategy: free
  tasks:
    - name: Long-running task
      command: /opt/slow-deployment.sh
```

---

## Advanced Task Control

### Serial Deployment

**Control how many hosts run simultaneously**:

```yaml
---
- name: Rolling deployment
  hosts: webservers
  serial: 1                    # One at a time
  tasks:
    - name: Deploy app
      copy:
        src: app.tar.gz
        dest: /opt/app/

    - name: Restart service
      service:
        name: myapp
        state: restarted

    - name: Health check
      uri:
        url: http://localhost:8080/health
        status_code: 200
```

**Percentage-based rolling**:
```yaml
serial:
  - 50%      # First 50% of hosts
  - 25%      # Then 25%
  - 100%     # Remaining hosts
```

### Run Once

**Execute task on single host only**:

```yaml
- name: Initialize database
  mysql_db:
    name: myapp
    state: present
  run_once: yes
  delegate_to: db1.example.com
```

### Order Execution

**Control task ordering**:

```yaml
---
- name: Complex orchestration
  hosts: all
  order: reverse_sorted       # Options: inventory, reverse_inventory, sorted, reverse_sorted, shuffle
  tasks:
    - name: Task order matters here
      debug:
        msg: "Host: {{ inventory_hostname }}"
```

---

## Error Handling & Recovery

### Failed When

**Custom failure conditions**:

```yaml
- name: Run command
  command: /opt/check.sh
  register: result
  failed_when:
    - result.rc != 0
    - "'ERROR' in result.stdout"
  ignore_errors: yes
```

### Changed When

**Control what constitutes a change**:

```yaml
- name: Check system status
  shell: systemctl status myapp
  register: result
  changed_when: "'inactive' in result.stdout"
```

### Rescue & Always

**Error recovery blocks**:

```yaml
- name: Deploy with recovery
  block:
    - name: Deploy application
      copy:
        src: app.tar.gz
        dest: /opt/app/
      register: deployment
    
    - name: Start service
      service:
        name: myapp
        state: started
  
  rescue:
    - name: Handle deployment failure
      debug:
        msg: "Deployment failed, attempting rollback"
    
    - name: Rollback to previous version
      copy:
        src: app-previous.tar.gz
        dest: /opt/app/
    
    - name: Restart with old version
      service:
        name: myapp
        state: restarted
  
  always:
    - name: Cleanup
      file:
        path: /tmp/deploy_*
        state: absent
```

### Fail Safe

**Control failure behavior**:

```yaml
- name: Graceful failure
  block:
    - name: Try operation
      command: /opt/optional-check.sh
      ignore_errors: yes
  
  rescue:
    - name: Handle gracefully
      set_fact:
        operation_result: "failed"
  
  always:
    - name: Log result
      debug:
        msg: "Operation result: {{ operation_result }}"
```

---

## Async & Parallel Execution

### Asynchronous Tasks

**Long-running tasks without blocking**:

```yaml
---
- name: Parallel deployment
  hosts: webservers
  tasks:
    # Start 5 deployments in parallel
    - name: Deploy application
      command: /opt/deploy.sh
      async: 3600                  # Timeout: 3600 seconds
      poll: 0                       # Don't wait (fire and forget)
      register: deployment_job
    
    # Check job status
    - name: Check deployment status
      async_status:
        jid: "{{ deployment_job.ansible_job_id }}"
      register: job_result
      until: job_result.finished
      retries: 30                   # Check up to 30 times
      delay: 10                     # Wait 10 seconds between checks
```

### Poll Modes

**Poll=0 (Fire and Forget)**:
```yaml
- name: Start background process
  command: python /opt/long_task.py
  async: 1800
  poll: 0
  register: bg_job
```

**Poll>0 (Check Periodically)**:
```yaml
- name: Task with periodic checks
  command: /opt/deployment.sh
  async: 3600
  poll: 30               # Check every 30 seconds
  register: result
```

### Wait For

**Wait for conditions**:

```yaml
- name: Deploy and wait for readiness
  tasks:
    - name: Deploy container
      docker_container:
        name: myapp
        image: myapp:latest
        state: started
    
    - name: Wait for app to be healthy
      uri:
        url: http://localhost:8080/health
        status_code: 200
      register: result
      until: result.status == 200
      retries: 30
      delay: 5
```

---

## Blocks & Exception Handling

### Block Basics

**Group related tasks**:

```yaml
- name: Web server configuration
  block:
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - nginx
        - curl
    
    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx
    
    - name: Start service
      service:
        name: nginx
        state: started
  
  when: ansible_os_family == "Debian"
```

### Block with Exception Handling

```yaml
- name: Complex deployment
  block:
    - name: Validate prerequisites
      assert:
        that:
          - ansible_memtotal_mb >= 2048
          - ansible_processor_vcpus >= 2
        fail_msg: "Insufficient resources"
    
    - name: Deploy application
      include_role:
        name: app_deploy
        tasks_from: deploy.yml
    
    - name: Run tests
      command: /opt/tests/run.sh
      register: tests
      failed_when: tests.rc != 0
  
  rescue:
    - name: Notify on failure
      mail:
        host: smtp.example.com
        to: devops@example.com
        subject: "Deployment failed on {{ inventory_hostname }}"
        body: "Error: {{ ansible_failed_result.msg }}"
      ignore_errors: yes
    
    - name: Attempt recovery
      block:
        - name: Run recovery script
          command: /opt/recovery.sh
      rescue:
        - name: Last resort cleanup
          file:
            path: /opt/app
            state: absent
  
  always:
    - name: Final cleanup
      debug:
        msg: "Deployment playbook finished"
```

---

## Include & Import

### Import vs Include

| Feature | import_tasks | include_tasks |
|---------|--------------|---------------|
| **Timing** | Pre-processing | Runtime |
| **Static** | Yes (can use variables) | Dynamic |
| **Conditional** | Limited | Full support |
| **Performance** | Faster | Slightly slower |

### Import Playbooks

```yaml
---
- name: Master playbook
  import_playbook: playbooks/prerequisites.yml
  
- name: Import conditional
  import_playbook: playbooks/{{ environment }}.yml
```

### Include Tasks

**Include reusable task lists**:

```yaml
---
- name: Deploy application
  hosts: webservers
  tasks:
    - name: Include common tasks
      include_tasks: tasks/common.yml
    
    - name: Include env-specific tasks
      include_tasks: "tasks/{{ environment }}.yml"
      vars:
        env_var: production
```

### Include Roles

**Include role inline**:

```yaml
- name: Configure servers
  hosts: all
  tasks:
    - name: Run monitoring role
      include_role:
        name: monitoring
        tasks_from: main.yml
      vars:
        monitoring_port: 9090
```

---

## Templating & Jinja2

### Template Basics

**Jinja2 expressions in playbooks**:

```yaml
---
- name: Configuration management
  hosts: all
  vars:
    app_name: myapp
    environment: production
    replicas: 3
  
  tasks:
    - name: Create config
      debug:
        msg: |
          App: {{ app_name }}
          Env: {{ environment }}
          Replicas: {{ replicas }}
```

### Conditional Expressions

**if/elif/else in templates**:

```yaml
- name: Configure based on OS
  template:
    src: app.conf.j2
    dest: /etc/app.conf
  vars:
    package_manager: "{% if ansible_os_family == 'Debian' %}apt{% elif ansible_os_family == 'RedHat' %}yum{% endif %}"
```

### Loops in Templates

**For loops in Jinja2**:

```yaml
- name: Generate hosts file
  template:
    src: hosts.j2
    dest: /etc/hosts
  vars:
    servers:
      - name: web1
        ip: 192.168.1.10
      - name: web2
        ip: 192.168.1.11
```

**Template file (hosts.j2)**:
```
localhost 127.0.0.1
{% for server in servers %}
{{ server.ip }} {{ server.name }}.example.com
{% endfor %}
```

### Filters

**Built-in Jinja2 filters**:

```yaml
- name: Use filters
  debug:
    msg: |
      Uppercase: {{ app_name | upper }}
      Lowercase: {{ "MYAPP" | lower }}
      Length: {{ my_list | length }}
      Join: {{ my_list | join(',') }}
      Default: {{ undefined_var | default('default_value') }}
      Select: {{ servers | selectattr('env', 'equalto', 'prod') | list }}
```

---

## Delegation & Local Actions

### Delegation

**Run task on different host**:

```yaml
---
- name: Deploy with delegation
  hosts: webservers
  tasks:
    - name: Deploy application
      copy:
        src: app.tar.gz
        dest: /opt/app/
    
    - name: Notify load balancer
      command: curl -X POST http://lb.example.com/reload
      delegate_to: localhost
    
    - name: Update DNS
      route53:
        zone: example.com
        record: app.example.com
        value: "{{ inventory_hostname }}"
      delegate_to: 127.0.0.1
```

### Local Actions

**Execute on control node**:

```yaml
- name: Deployment with notifications
  hosts: webservers
  tasks:
    - name: Deploy
      copy:
        src: app.tar.gz
        dest: /opt/app/
    
    - name: Notify via webhook (local)
      local_action:
        module: uri
        url: "{{ webhook_url }}"
        method: POST
        body_format: json
        body:
          event: deployment_started
          host: "{{ inventory_hostname }}"
```

### Delegated Facts

**Gather facts on different host**:

```yaml
- name: Gather database facts
  setup:
    gather_subset: all
  delegate_to: db.example.com
  register: db_facts
  
- name: Use facts from delegated host
  debug:
    msg: "DB version: {{ db_facts.ansible_facts.ansible_version }}"
```

---

## Advanced Patterns

### Handler Idempotency

**Handlers only run when needed**:

```yaml
---
- name: Configuration management
  hosts: webservers
  tasks:
    - name: Deploy config
      copy:
        src: app.conf
        dest: /etc/app.conf
        backup: yes
      notify: Restart application
    
    - name: Deploy another config
      copy:
        src: logging.conf
        dest: /etc/logging.conf
      notify: Restart application
  
  handlers:
    - name: Restart application
      service:
        name: myapp
        state: restarted
      listen: "Restart application"  # Listen for multiple notifications
```

### Facts as Variables

**Register and reuse facts**:

```yaml
- name: Complex automation
  hosts: all
  tasks:
    - name: Check service status
      command: systemctl status myapp
      register: service_status
      failed_when: false
      changed_when: false
    
    - name: Act on status
      debug:
        msg: "Service is running"
      when: service_status.rc == 0
    
    - name: Start if not running
      service:
        name: myapp
        state: started
      when: service_status.rc != 0
```

### Conditional Based on Previous Task

```yaml
- name: Deployment workflow
  hosts: webservers
  tasks:
    - name: Check current version
      command: cat /opt/app/VERSION
      register: current_version
      changed_when: false
    
    - name: Deploy new version
      copy:
        src: "app-{{ version }}.tar.gz"
        dest: /opt/app/
      when: current_version.stdout != version
      register: deployment
    
    - name: Restart only if deployed
      service:
        name: myapp
        state: restarted
      when: deployment.changed
```

### Idempotent Scripting

**Making bash scripts idempotent**:

```yaml
- name: Idempotent operations
  hosts: all
  tasks:
    - name: Add line to file only once
      lineinfile:
        path: /etc/app.conf
        line: "config_key = value"
        state: present
    
    - name: Create user if not exists
      user:
        name: appuser
        state: present
        shell: /bin/bash
    
    - name: Install package (idempotent)
      apt:
        name: nginx
        state: present
```

---

## Playbook Testing

### Syntax Checking

```bash
# Check YAML syntax
ansible-playbook playbook.yml --syntax-check

# Check for specific errors
ansible-playbook playbook.yml -i inventory --syntax-check -vv
```

### Dry Run (Check Mode)

```bash
# Preview changes without applying
ansible-playbook playbook.yml -i inventory --check

# Diff mode shows changes
ansible-playbook playbook.yml -i inventory --check --diff
```

### Verbosity Levels

```bash
# Standard output
ansible-playbook playbook.yml

# Verbose (task results)
ansible-playbook playbook.yml -v

# Very verbose (all tasks)
ansible-playbook playbook.yml -vv

# Debug (variable values)
ansible-playbook playbook.yml -vvv

# Extreme debugging
ansible-playbook playbook.yml -vvvv
```

### Testing Patterns

**Test playbook structure**:

```yaml
---
- name: Test playbook
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Validate variables
      assert:
        that:
          - app_name is defined
          - app_port is defined
          - app_port | int >= 1024 and app_port | int <= 65535
        fail_msg: "Invalid configuration"
    
    - name: Test connectivity
      wait_for:
        host: "{{ item }}"
        port: 22
        timeout: 5
      loop: "{{ groups['webservers'] }}"
      ignore_errors: yes
    
    - name: Validate prerequisites
      command: /opt/validate.sh
      register: validation
      failed_when: validation.rc != 0
```

---

## Best Practices

### Playbook Organization

✅ **Modular design** - Use roles and includes  
✅ **Clear naming** - Descriptive task names  
✅ **Comments** - Explain complex logic  
✅ **Error handling** - Anticipate failures  
✅ **Idempotency** - Safe to run multiple times  

### Performance

✅ **Minimal fact gathering** - Only needed facts  
✅ **Batch execution** - Parallel where possible  
✅ **Async tasks** - Long-running operations  
✅ **Caching** - Reuse computed values  
✅ **Variable scope** - Limit variable pollution  

### Security

✅ **Vault encryption** - Secure sensitive data  
✅ **Limited privileges** - Principle of least privilege  
✅ **Audit logging** - Track changes  
✅ **Secure transfer** - SFTP, SSH keys  
✅ **Input validation** - Verify user input  

### Maintainability

✅ **Version control** - All playbooks in Git  
✅ **Documentation** - README and comments  
✅ **Testing** - Validate before production  
✅ **CI/CD integration** - Automated deployment  
✅ **Change management** - Approval process  

---

## Quick Reference

### Essential Commands

```bash
# Run playbook
ansible-playbook site.yml -i inventory

# Check syntax
ansible-playbook site.yml --syntax-check

# Dry run
ansible-playbook site.yml --check --diff

# Run specific tags
ansible-playbook site.yml --tags "deploy"

# Skip specific tags
ansible-playbook site.yml --skip-tags "tests"

# Verbose output
ansible-playbook site.yml -vvv

# Run with extra variables
ansible-playbook site.yml -e "env=prod version=1.2.3"

# List tasks
ansible-playbook site.yml --list-tasks

# Step through playbook
ansible-playbook site.yml --step
```

### Playbook Template

```yaml
---
- name: Application deployment
  hosts: webservers
  become: yes
  gather_facts: yes
  
  vars:
    app_name: myapp
    app_version: 1.0.0
  
  vars_files:
    - vars/main.yml
  
  pre_tasks:
    - name: Pre-deployment checks
      assert:
        that:
          - ansible_memory_mb.real.total > 2048
        fail_msg: "Insufficient memory"
  
  roles:
    - role: common
    - role: "{{ app_name }}"
  
  tasks:
    - name: Deploy application
      block:
        - name: Download app
          get_url:
            url: "{{ app_url }}/{{ app_name }}-{{ app_version }}.tar.gz"
            dest: /tmp/
        
        - name: Extract app
          unarchive:
            src: "/tmp/{{ app_name }}-{{ app_version }}.tar.gz"
            dest: /opt/
      
      rescue:
        - name: Handle deployment failure
          debug:
            msg: "Deployment failed"
      
      always:
        - name: Cleanup
          file:
            path: /tmp/{{ app_name }}-*
            state: absent
  
  post_tasks:
    - name: Verify deployment
      uri:
        url: http://localhost:8080/health
        status_code: 200
  
  handlers:
    - name: Restart application
      service:
        name: "{{ app_name }}"
        state: restarted
```

---

## Summary

**Advanced Ansible playbooks** enable:

✅ **Complex orchestration** - Multi-play coordination  
✅ **Error recovery** - Graceful failure handling  
✅ **Parallel execution** - Async tasks for speed  
✅ **Flexible templating** - Dynamic configuration  
✅ **Delegation** - Execute on specific hosts  
✅ **Idempotency** - Safe repeated execution  

**Key techniques**:
- **Serial deployment**: Rolling updates
- **Block exception handling**: Try/rescue/always
- **Async/poll**: Background task execution
- **Include/import**: Code reuse
- **Delegation**: Task execution control
- **Templating**: Dynamic file generation

---
