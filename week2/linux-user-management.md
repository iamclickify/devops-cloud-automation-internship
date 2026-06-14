# User & Group Management Basics

## Overview

Linux is a multi-user operating system where every file, process, and service belongs to a user and a group. User and group management helps enforce security, access control, and collaboration.

---

## Important System Files

| File          | Purpose                  |
| ------------- | ------------------------ |
| `/etc/passwd` | User account information |
| `/etc/shadow` | Encrypted passwords      |
| `/etc/group`  | Group information        |

---

## User Management Commands

### Create User

```bash
sudo useradd username
```

Create user with home directory:

```bash
sudo useradd -m username
```

### Set Password

```bash
sudo passwd username
```

### Delete User

```bash
sudo userdel username
```

Delete user with home directory:

```bash
sudo userdel -r username
```

### Switch User

```bash
su - username
```

### Check Current User

```bash
whoami
```

### View Logged-In Users

```bash
who
```

---

## Group Management Commands

### Create Group

```bash
sudo groupadd developers
```

### Delete Group

```bash
sudo groupdel developers
```

### Add User to Group

```bash
sudo usermod -aG developers username
```

### View User Groups

```bash
groups username
```

---

# File Ownership & Permissions

Every file and directory has:

* Owner (User)
* Group
* Permissions

View ownership and permissions:

```bash
ls -l
```

Example:

```bash
-rw-r--r-- 1 shubham developers notes.txt
```

---

## Permission Types

| Permission | Symbol | Value |
| ---------- | ------ | ----- |
| Read       | r      | 4     |
| Write      | w      | 2     |
| Execute    | x      | 1     |

---

## Permission Categories

| Category   | Description   |
| ---------- | ------------- |
| User (u)   | File owner    |
| Group (g)  | Group owner   |
| Others (o) | Everyone else |

Example:

```bash
rwxr-xr--
```

* User → rwx
* Group → r-x
* Others → r--

Numeric form:

```bash
754
```

---

## Common Permission Values

| Value | Meaning                         |
| ----- | ------------------------------- |
| 777   | Full access                     |
| 755   | Owner full, others read/execute |
| 744   | Owner full, others read         |
| 700   | Owner only                      |
| 770   | User & group full access        |
| 644   | Standard file permission        |

---

## Change Permissions

```bash
chmod 755 script.sh
```

Recursive:

```bash
chmod -R 770 project/
```

---

## Change Ownership

Change file owner:

```bash
sudo chown user file.txt
```

Example:

```bash
sudo chown shubham notes.txt
```

Change group ownership:

```bash
sudo chgrp developers project/
```

Recursive:

```bash
sudo chgrp -R developers project/
```

---

# Real-World DevOps Example

Project Directory:

```bash
/srv/projects/my_project
```

Create team group:

```bash
sudo groupadd developers
```

Assign group ownership:

```bash
sudo chgrp -R developers /srv/projects/my_project
```

Set permissions:

```bash
sudo chmod -R 770 /srv/projects/my_project
```

Add team members:

```bash
sudo usermod -aG developers alice
sudo usermod -aG developers bob
```

Result:

* Developers can collaborate securely
* Shared access is managed through groups
* Unauthorized users cannot access project files

---

## Quick Command Cheat Sheet

```bash
whoami
who

sudo useradd -m username
sudo passwd username
sudo userdel -r username

sudo groupadd groupname
sudo groupdel groupname

sudo usermod -aG groupname username
groups username

ls -l

chmod 755 file
chmod -R 770 directory

chown user file
chgrp group file
```

---

## DevOps Relevance

* Manage Linux server users
* Configure application access
* Secure shared project directories
* Manage CI/CD service accounts
* Implement least-privilege security practices
* Essential for cloud and infrastructure administration
