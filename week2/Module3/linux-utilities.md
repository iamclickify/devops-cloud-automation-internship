# Linux Commands for DevOps

## Overview

Linux is the foundation of modern DevOps, cloud computing, containers, CI/CD pipelines, and server administration. This module covered essential Linux command-line skills required for system management, process monitoring, software installation, networking, and automation.

---

## Text Editors

Linux administrators frequently edit configuration files, scripts, and application settings using terminal-based editors.

### Nano

A beginner-friendly text editor.

Common commands:

```bash
nano filename
```

* `Ctrl + O` → Save file
* `Ctrl + X` → Exit
* `Ctrl + K` → Cut line
* `Ctrl + U` → Paste line
* `Ctrl + W` → Search text

### Vim

A powerful editor available on almost every Linux server.

```bash
vim filename
```

Important commands:

* `i` → Insert mode
* `Esc` → Return to normal mode
* `:w` → Save
* `:q` → Quit
* `:wq` → Save and quit
* `:q!` → Quit without saving
* `dd` → Delete line
* `yy` → Copy line
* `p` → Paste
* `/word` → Search

---

## Process Management

Processes are running programs in Linux. Monitoring and controlling them is essential for troubleshooting.

### View Processes

```bash
ps aux
```

```bash
ps -ef
```

Search for a specific process:

```bash
ps aux | grep nginx
```

### Real-Time Monitoring

```bash
top
```

Useful shortcuts:

* `q` → Quit
* `P` → Sort by CPU usage
* `M` → Sort by memory usage
* `k` → Kill process

### Terminate Processes

Graceful termination:

```bash
kill PID
```

Force termination:

```bash
kill -9 PID
```

---

## Package Management

Package managers simplify software installation and updates.

### APT (Ubuntu/Debian)

Update repositories:

```bash
sudo apt update
```

Install package:

```bash
sudo apt install package_name
```

Remove package:

```bash
sudo apt remove package_name
```

Upgrade system:

```bash
sudo apt upgrade
```

Search package:

```bash
apt search package_name
```

### YUM / DNF (RHEL, CentOS, Fedora)

Install package:

```bash
sudo yum install package_name
```

or

```bash
sudo dnf install package_name
```

Update packages:

```bash
sudo yum update
```

Remove package:

```bash
sudo yum remove package_name
```

---

## Networking Commands

Networking tools help verify connectivity and troubleshoot servers.

### Ping

Check connectivity to a host:

```bash
ping google.com
```

Send limited packets:

```bash
ping -c 4 google.com
```

### Netstat

Display listening services and active connections:

```bash
sudo netstat -tulnp
```

Check specific ports:

```bash
sudo netstat -tulnp | grep :80
```

### IP Command

Display network configuration:

```bash
ip addr show
```

or

```bash
ip a
```

Display routing table:

```bash
ip route
```

or

```bash
ip r
```

---

## Redirection and Piping

These features are heavily used in shell scripting and automation.

### Output Redirection

Create/overwrite file:

```bash
ls -l > files.txt
```

Append output:

```bash
date >> files.txt
```

### Input Redirection

```bash
wc < sample.txt
```

### Piping

Pass output from one command to another:

```bash
cat sample.txt | sort
```

Filter logs:

```bash
cat logs.txt | grep ERROR
```

Count results:

```bash
cat logs.txt | grep ERROR | wc -l
```

### Error Redirection

Redirect errors:

```bash
ls missing.txt 2> error.log
```

Redirect output and errors together:

```bash
ls file.txt missing.txt &> output.log
```

---

## Commands Practiced

```bash
nano my_app.conf
vim Dockerfile

ps aux
ps -ef
top
kill PID
kill -9 PID

sudo apt update
sudo apt install htop

ping google.com
netstat -tulnp
ip addr show

ls -l > files.txt
date >> files.txt
wc < sample.txt
cat sample.txt | sort
```

---

## Key Takeaways

* Linux CLI is the primary interface for DevOps engineers.
* Nano and Vim are essential for editing configuration files and scripts.
* Process management tools help monitor and troubleshoot applications.
* Package managers automate software installation and updates.
* Networking commands assist in connectivity and server diagnostics.
* Redirection and piping are foundational concepts for shell scripting and automation.
* These commands are widely used in cloud environments, CI/CD pipelines, Docker containers, Kubernetes clusters, and production servers.
