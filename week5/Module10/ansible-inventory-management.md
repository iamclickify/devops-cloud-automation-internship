# Ansible Inventory Management

## Overview

Ansible inventory is the foundation of infrastructure automation, defining all managed hosts and their properties. Advanced inventory management enables dynamic host discovery, group hierarchies, variable organization, and flexible deployment strategies across complex infrastructure.

---

## Table of Contents

1. [Inventory Fundamentals](#inventory-fundamentals)
2. [Inventory Formats](#inventory-formats)
3. [Hosts & Groups](#hosts--groups)
4. [Variables & Precedence](#variables--precedence)
5. [Group Variables & Host Variables](#group-variables--host-variables)
6. [Dynamic Inventory](#dynamic-inventory)
7. [Inventory Plugins](#inventory-plugins)
8. [Ansible Patterns](#ansible-patterns)
9. [Best Practices](#best-practices)

---

## Inventory Fundamentals

### What is Inventory?

**Inventory** is a catalog of all infrastructure that Ansible can manage - servers, cloud instances, network devices, containers, etc.

### Inventory Purposes

✅ **Host discovery** - Know all machines to manage  
✅ **Host grouping** - Organize by role, environment, location  
✅ **Host variables** - Store per-host configuration  
✅ **Group variables** - Share configuration across hosts  
✅ **Credentials** - Store connection information  
✅ **Dynamic membership** - Automatic group assignment  

### Inventory Location

**Default locations** (in order):
1. `/etc/ansible/hosts` - System default
2. `.ansible.cfg` configured location
3. Command-line specified with `-i`

**Specify inventory**:
```bash
# Use specific file
ansible-playbook site.yml -i inventory/production.ini

# Use directory (all files)
ansible-playbook site.yml -i inventory/

# Multiple inventories
ansible-playbook site.yml -i inventory/static.ini -i inventory/dynamic.py
```

### Inventory Structure

```
inventory/
├── production/
│   ├── hosts.ini          # Static hosts
│   ├── hosts.yml          # YAML format
│   ├── group_vars/        # Group variables
│   │   ├── webservers.yml
│   │   ├── databases.yml
│   │   └── all.yml
│   └── host_vars/         # Host variables
│       ├── web1.example.com.yml
│       └── web2.example.com.yml
├── staging/
│   └── hosts.ini
└── dynamic.py             # Dynamic inventory script
```

---

## Inventory Formats

### INI Format

**Traditional, simple format**:

```ini
[webservers]
web1.example.com ansible_host=192.168.1.10
web2.example.com ansible_host=192.168.1.11 ansible_port=2222
web3.example.com

[databases]
db1.example.com ansible_host=192.168.1.20
db2.example.com ansible_host=192.168.1.21

[loadbalancers]
lb1.example.com

# Ungrouped hosts
orphan.example.com

# Group of groups
[production:children]
webservers
databases
loadbalancers

# Group variables
[webservers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/web_key.pem
nginx_port=80
```

### YAML Format

**More readable, structured format**:

```yaml
---
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          ansible_host: 192.168.1.10
          ansible_user: ubuntu
        web2.example.com:
          ansible_host: 192.168.1.11
          ansible_port: 2222
          ansible_user: ubuntu
      vars:
        nginx_port: 80
        app_env: production
    
    databases:
      hosts:
        db1.example.com:
          ansible_host: 192.168.1.20
        db2.example.com:
          ansible_host: 192.168.1.21
      vars:
        database_port: 5432
    
    loadbalancers:
      hosts:
        lb1.example.com:
      vars:
        lb_type: haproxy
    
    production:
      children:
        - webservers
        - databases
        - loadbalancers
  
  vars:
    ansible_user: ubuntu
    ansible_ssh_private_key_file: ~/.ssh/default_key.pem
```

### Format Comparison

| Feature | INI | YAML |
|---------|-----|------|
| **Readability** | Simple | Structured |
| **Nested groups** | Limited | Full support |
| **Comments** | Supported | Supported |
| **Complex data** | Basic | Full YAML |
| **Learning curve** | Flat | Steeper |

---

## Hosts & Groups

### Host Definitions

**Single host**:
```ini
web1.example.com
```

**Host with variables**:
```ini
web1.example.com ansible_host=192.168.1.10 ansible_port=2222 ansible_user=ubuntu
```

**YAML format**:
```yaml
webservers:
  hosts:
    web1.example.com:
      ansible_host: 192.168.1.10
      ansible_port: 2222
      ansible_user: ubuntu
```

### Group Definitions

**Simple group**:
```ini
[webservers]
web1.example.com
web2.example.com
web3.example.com
```

**Group with variables**:
```ini
[webservers:vars]
ansible_user=ubuntu
app_port=8080
```

### Group of Groups

**Parent groups (meta-groups)**:

```ini
[webservers]
web1.example.com
web2.example.com

[databases]
db1.example.com
db2.example.com

[production:children]
webservers
databases
```

**YAML format**:
```yaml
all:
  children:
    production:
      children:
        - webservers
        - databases
    staging:
      children:
        - webservers
        - databases
```

### Special Groups

**all** - All hosts in inventory
```bash
ansible all -i inventory.ini -m ping
```

**ungrouped** - Hosts not in any group
```bash
ansible ungrouped -i inventory.ini -m ping
```

**localhost** - Control node
```ini
localhost ansible_connection=local
```

---

## Variables & Precedence

### Variable Sources (Priority Order)

1. **Extra variables** (-e flag) - Highest priority
2. **Task variables** - Variables set in task
3. **Block variables** - Variables in block
4. **Play variables** - Variables in play vars
5. **Host variables** - host_vars/hostname.yml
6. **Group variables** - group_vars/groupname.yml
7. **Inventory variables** - Defined in inventory
8. **Role defaults** - roles/role/defaults/main.yml - Lowest priority

```
Extra (-e) > Task > Block > Play > Host vars > Group vars > Inventory > Role defaults
```

### Variable Examples

**Inventory variables**:
```ini
[webservers]
web1 ansible_host=192.168.1.10 nginx_port=80
web2 ansible_host=192.168.1.11 nginx_port=8080
```

**Task variables**:
```yaml
- name: Deploy
  hosts: webservers
  tasks:
    - name: Configure nginx
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      vars:
        nginx_port: 80
```

**Extra variables**:
```bash
ansible-playbook site.yml -e "env=prod version=1.2.3"
```

---

## Group Variables & Host Variables

### Group Variables Directory Structure

```
group_vars/
├── all.yml                    # Applied to all hosts
├── webservers.yml             # Applied to webservers group
├── webservers/                # Multi-file for webservers
│   ├── main.yml
│   ├── packages.yml
│   └── security.yml
├── production.yml             # Multi-level group
└── staging.yml
```

### Group Variables Example

**group_vars/webservers.yml**:
```yaml
---
# Common web server configuration
nginx_version: "1.21.0"
nginx_port: 80
app_user: www-data
app_group: www-data

# List variable
required_packages:
  - nginx
  - curl
  - git
  - python3-pip

# Dictionary variable
ssl_config:
  enabled: true
  certificate_path: /etc/ssl/certs/server.crt
  key_path: /etc/ssl/private/server.key
```

**group_vars/production.yml**:
```yaml
---
environment: production
debug_mode: false
log_level: error
backup_enabled: true
replicas: 3
```

**group_vars/all.yml**:
```yaml
---
# Applied to all hosts
ansible_user: ubuntu
ansible_ssh_private_key_file: ~/.ssh/default_key.pem
ntp_server: time.example.com
syslog_server: 192.168.1.30
```

### Host Variables Directory Structure

```
host_vars/
├── web1.example.com.yml       # Single host
├── db1.example.com.yml
└── production/
    ├── web1.example.com.yml   # Host in subdirectory
    └── db1.example.com.yml
```

### Host Variables Example

**host_vars/web1.example.com.yml**:
```yaml
---
# Host-specific configuration
ansible_host: 192.168.1.10
nginx_port: 80
server_name: web1-prod
datacenter: us-east-1a
```

**host_vars/db1.example.com.yml**:
```yaml
---
ansible_host: 192.168.1.20
database_port: 5432
database_name: production_db
backup_schedule: "0 2 * * *"  # 2 AM daily
replication_enabled: true
```

### Variable Organization Best Practice

```
inventory/
├── hosts.yml                  # Inventory definition
├── group_vars/
│   ├── all.yml               # Global defaults
│   ├── webservers.yml        # Web server config
│   ├── databases.yml         # Database config
│   ├── production.yml        # Production env
│   └── staging.yml           # Staging env
└── host_vars/
    ├── web1.example.com.yml  # Host 1 overrides
    ├── web2.example.com.yml  # Host 2 overrides
    ├── db1.example.com.yml   # Host 3 overrides
    └── db2.example.com.yml   # Host 4 overrides
```

---

## Dynamic Inventory

### What is Dynamic Inventory?

**Automatically populate inventory from external sources** instead of static files

### Benefits

✅ **Auto-discovery** - Discover instances dynamically  
✅ **No manual updates** - Always in sync  
✅ **Cloud integration** - AWS, Azure, GCP native  
✅ **Scale** - Handles hundreds/thousands of hosts  
✅ **Real-time** - Reflects infrastructure changes  

### Dynamic Inventory Script

**Python example** (outputs JSON):

```python
#!/usr/bin/env python3

import json
import sys

def get_inventory():
    inventory = {
        "webservers": {
            "hosts": ["192.168.1.10", "192.168.1.11"],
            "vars": {
                "ansible_user": "ubuntu",
                "nginx_port": 80
            }
        },
        "databases": {
            "hosts": ["192.168.1.20", "192.168.1.21"],
            "vars": {
                "ansible_user": "postgres",
                "database_port": 5432
            }
        },
        "_meta": {
            "hostvars": {
                "192.168.1.10": {
                    "datacenter": "us-east-1a"
                },
                "192.168.1.11": {
                    "datacenter": "us-east-1b"
                },
                "192.168.1.20": {
                    "database_role": "primary"
                },
                "192.168.1.21": {
                    "database_role": "replica"
                }
            }
        }
    }
    return inventory

if __name__ == "__main__":
    print(json.dumps(get_inventory(), indent=2))
```

**Use dynamic inventory**:
```bash
chmod +x inventory.py
ansible-playbook site.yml -i inventory.py
```

### AWS Dynamic Inventory

**Using aws_ec2 plugin** (built-in):

```yaml
# inventory/aws_ec2.yml
plugin: aws_ec2
regions:
  - us-east-1
  - us-west-2
keyed_groups:
  - prefix: tag
    key: tags
  - prefix: aws_region
    key: placement.region_name
  - prefix: instance_type
    key: instance_type
compose:
  ansible_host: public_ip_address
filters:
  tag:Environment: production
  instance-state-name: running
```

**Use AWS inventory**:
```bash
ansible-playbook site.yml -i inventory/aws_ec2.yml
```

### Terraform Dynamic Inventory

**Generate from Terraform state**:

```python
#!/usr/bin/env python3

import json
import subprocess

def terraform_inventory():
    # Get Terraform outputs
    result = subprocess.run(
        ['terraform', 'output', '-json'],
        capture_output=True,
        text=True
    )
    
    tf_output = json.loads(result.stdout)
    
    inventory = {
        "webservers": {
            "hosts": tf_output.get("web_ips", [])
        },
        "databases": {
            "hosts": tf_output.get("db_ips", [])
        },
        "_meta": {
            "hostvars": {}
        }
    }
    
    return inventory

if __name__ == "__main__":
    print(json.dumps(terraform_inventory(), indent=2))
```

---

## Inventory Plugins

### Built-in Plugins

| Plugin | Purpose | Format |
|--------|---------|--------|
| **ini** | INI file format | .ini |
| **yaml** | YAML file format | .yml, .yaml |
| **aws_ec2** | AWS EC2 instances | Dynamic |
| **azure_rm** | Azure VMs | Dynamic |
| **gcp_compute** | Google Compute Engine | Dynamic |
| **docker** | Docker containers | Dynamic |
| **kubernetes** | Kubernetes pods | Dynamic |
| **constructed** | Build groups from variables | Dynamic |

### Using Plugins

**aws_ec2 plugin** (in ansible.cfg):
```ini
[inventory]
enable_plugins = aws_ec2, yaml, ini
```

**Plugin configuration**:
```yaml
# inventory/aws.yml
plugin: aws_ec2
aws_profile: default
regions:
  - us-east-1
keyed_groups:
  - prefix: tag_Environment
    key: tags.Environment
compose:
  ansible_host: public_ip_address
```

### Custom Plugins

**Create custom plugin** (plugins/inventory/custom.py):

```python
from ansible.plugins.inventory import BaseInventoryPlugin

class InventoryModule(BaseInventoryPlugin):
    NAME = 'custom'
    
    def parse(self, inventory, loader, path, cache=True):
        super(InventoryModule, self).parse(inventory, loader, path, cache)
        
        # Add hosts
        self.inventory.add_host('host1.example.com')
        self.inventory.add_host('host2.example.com')
        
        # Add groups
        self.inventory.add_group('webservers')
        self.inventory.add_host('host1.example.com', group='webservers')
        
        # Add variables
        self.inventory.set_variable('host1.example.com', 'var_name', 'value')
```

---

## Ansible Patterns

### Host Patterns

**Target specific hosts/groups**:

```bash
# All hosts
ansible all -i inventory.ini

# Specific group
ansible webservers -i inventory.ini

# Multiple groups
ansible webservers,databases -i inventory.ini

# Group minus hosts
ansible webservers,!web2.example.com -i inventory.ini

# Host by regex
ansible "~web[0-9]+" -i inventory.ini

# Group of groups
ansible production -i inventory.ini

# Specific host
ansible web1.example.com -i inventory.ini

# Limit percentage
ansible all -i inventory.ini -l "50%"

# Limit by index
ansible all -i inventory.ini -l "0:10"  # First 10
```

### Using Patterns in Playbooks

```yaml
---
- name: Deploy all
  hosts: all
  tasks:
    - name: Task for all
      debug:
        msg: "Running on all hosts"

- name: Deploy production
  hosts: production
  tasks:
    - name: Prod-only task
      debug:
        msg: "Running in production"

- name: Deploy specific hosts
  hosts: "web1.example.com,web2.example.com"
  tasks:
    - name: Specific hosts only
      debug:
        msg: "Running on specific hosts"

- name: Deploy excluding hosts
  hosts: "all:!staging"
  tasks:
    - name: Skip staging
      debug:
        msg: "Running everywhere except staging"
```

---

## Best Practices

### Inventory Organization

✅ **Environment separation** - Separate prod/staging/dev  
✅ **Group by function** - webservers, databases, etc.  
✅ **Use group_vars** - Share config across groups  
✅ **Use host_vars** - Host-specific overrides  
✅ **Version control** - Keep in Git  
✅ **Document groups** - Comments explaining purpose  

### Inventory Structure Example

```
inventory/
├── README.md                          # Inventory documentation
├── production/
│   ├── hosts.yml                      # Production hosts
│   ├── group_vars/
│   │   ├── all.yml                    # Global vars
│   │   ├── webservers.yml             # Webserver group vars
│   │   └── databases.yml              # Database group vars
│   └── host_vars/
│       ├── web1.prod.example.com.yml  # Host overrides
│       └── db1.prod.example.com.yml
├── staging/
│   ├── hosts.yml
│   ├── group_vars/
│   └── host_vars/
├── development/
│   ├── hosts.yml
│   ├── group_vars/
│   └── host_vars/
└── dynamic.py                         # Dynamic inventory script
```

### Variable Management

✅ **Clear naming** - Use descriptive variable names  
✅ **Consistent format** - Use YAML consistently  
✅ **Defaults in roles** - Default values in defaults/  
✅ **Override in groups** - group_vars override defaults  
✅ **Override per-host** - host_vars for exceptions  
✅ **Document variables** - Comment complex values  

### Security

✅ **Use vault** - Encrypt sensitive variables
```bash
ansible-vault encrypt group_vars/all.yml
```

✅ **SSH keys** - Use key-based auth, not passwords  
✅ **Limit inventory** - Restrict read access  
✅ **No secrets in code** - Use vault or external secrets  
✅ **Audit changes** - Version control with history  

### Performance

✅ **Minimize fact gathering** - Only needed facts  
✅ **Cache facts** - Reuse gathered information  
✅ **Efficient groups** - Smart group membership  
✅ **Batch operations** - Process in parallel  
✅ **Monitor inventory size** - Large inventories slow down  

---

## Quick Reference

### Common Inventory Commands

```bash
# List all hosts
ansible-inventory -i inventory.ini --list

# List specific group
ansible-inventory -i inventory.ini --graph webservers

# Show inventory tree
ansible-inventory -i inventory.ini --graph

# Show host details
ansible-inventory -i inventory.ini --host web1.example.com

# Validate inventory
ansible-inventory -i inventory.ini --list > /dev/null && echo "Valid"

# Check specific host
ansible -i inventory.ini web1.example.com -m setup

# List hosts in group
ansible -i inventory.ini --list-hosts webservers
```

### Inventory File Template

**INI Format** (inventory.ini):
```ini
# Global variables
[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/default.pem

# Web servers
[webservers]
web1.example.com
web2.example.com

[webservers:vars]
nginx_port=80
app_user=www-data

# Databases
[databases]
db1.example.com ansible_host=192.168.1.20
db2.example.com ansible_host=192.168.1.21

[databases:vars]
database_port=5432

# Groups
[production:children]
webservers
databases

[production:vars]
environment=production
```

**YAML Format** (inventory.yml):
```yaml
---
all:
  vars:
    ansible_user: ubuntu
    ansible_ssh_private_key_file: ~/.ssh/default.pem
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
      vars:
        nginx_port: 80
    databases:
      hosts:
        db1.example.com:
          ansible_host: 192.168.1.20
      vars:
        database_port: 5432
    production:
      children:
        - webservers
        - databases
      vars:
        environment: production
```

---

## Summary

**Ansible inventory** provides:

✅ **Host management** - Centralized infrastructure catalog  
✅ **Organization** - Group hosts by role, environment, location  
✅ **Configuration** - Store host/group variables  
✅ **Flexibility** - Static or dynamic sources  
✅ **Scalability** - Manage thousands of hosts  
✅ **Integration** - Cloud provider plugins  

**Key concepts**:
- **Hosts**: Individual machines to manage
- **Groups**: Organize hosts by function
- **Variables**: Configuration data (precedence matters)
- **Inventory files**: INI or YAML format
- **Dynamic inventory**: Discover hosts automatically
- **Patterns**: Target specific hosts/groups

---
