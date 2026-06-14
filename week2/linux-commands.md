# Linux Commands & File Management

## Overview

Linux command-line skills are essential for DevOps engineers because most cloud servers, Docker containers, Kubernetes nodes, and CI/CD systems run on Linux.

This module covers:

- File viewing
- File management
- Wildcards (Globbing)
- Permissions
- File searching
- Practical workspace operations

---

# 1. Viewing File Content

## cat

Displays entire file contents.

```bash
cat file.txt
```

Display multiple files:

```bash
cat file1.txt file2.txt
```

Create a file:

```bash
cat > notes.txt
```

Append content:

```bash
cat extra.txt >> notes.txt
```

Useful options:

```bash
cat -n file.txt
cat -b file.txt
cat -s file.txt
```

### Real Use Cases

- View configuration files
- Merge logs
- Create quick text files

---

## less

View large files page by page.

```bash
less large.log
```

Useful for:

- Server logs
- Application logs
- Large configuration files

---

## head

Display first 10 lines.

```bash
head file.txt
```

Specify lines:

```bash
head -20 file.txt
```

---

## tail

Display last 10 lines.

```bash
tail file.txt
```

Monitor logs live:

```bash
tail -f application.log
```

Very common in DevOps.

---

# 2. Wildcards (Globbing)

Wildcards help perform operations on multiple files.

---

## *

Matches zero or more characters.

Examples:

```bash
ls *.txt
cp *.jpg backup/
rm *.tmp
```

Examples:

```text
report.txt
notes.txt
data.txt
```

Matched by:

```bash
*.txt
```

---

## ?

Matches exactly one character.

Example:

```bash
ls file?.txt
```

Matches:

```text
file1.txt
fileA.txt
```

Does not match:

```text
file10.txt
```

---

## []

Matches one character from a set.

Example:

```bash
ls [abc]*
```

Matches:

```text
apple.txt
banana.txt
cat.txt
```

---

## [!]

Negates the selection.

Example:

```bash
ls [!abc]*
```

Lists files not starting with a, b, or c.

---

# 3. File & Directory Management

## pwd

Print current working directory.

```bash
pwd
```

---

## ls

List files and directories.

```bash
ls
ls -l
ls -a
```

---

## cd

Change directory.

```bash
cd folder
cd ..
cd ~
```

---

## mkdir

Create directories.

```bash
mkdir project
mkdir src docs data
```

---

## touch

Create empty files.

```bash
touch app.py
touch config.yaml
```

---

## cp

Copy files or directories.

Copy file:

```bash
cp file1 file2
```

Copy directory:

```bash
cp -r src src_backup
```

---

## mv

Move or rename files.

Move:

```bash
mv data docs/
```

Rename:

```bash
mv src_backup old_src
```

---

## rm

Delete files.

```bash
rm file.txt
```

Delete directories:

```bash
rm -r old_src
```

---

# 4. Permissions & Ownership

Linux permissions use:

```text
r = Read
w = Write
x = Execute
```

Example:

```text
-rwxr-xr--
```

---

## chmod

Change permissions.

```bash
chmod 755 script.sh
chmod 644 file.txt
```

Common values:

| Permission | Meaning |
|------------|----------|
| 777 | Full access |
| 755 | Owner full access |
| 644 | Owner RW, others Read |

---

## chown

Change owner.

```bash
sudo chown user file.txt
```

Example:

```bash
sudo chown ubuntu config.yaml
```

---

## chgrp

Change group ownership.

```bash
sudo chgrp developers app.py
```

---

# 5. Finding Files

## find

Search files and directories.

Find text files:

```bash
find . -name "*.txt"
```

Find configuration files:

```bash
find /etc -name "*.conf"
```

---

## locate

Fast file search using database.

```bash
locate nginx.conf
```

Update database:

```bash
sudo updatedb
```

---

# Practical Lab Summary

Created:

```text
devops_practice/
├── docs/
│   └── data/
└── src/
    ├── main.py
    └── config.yaml
```

Commands practiced:

```bash
pwd
cd
ls
mkdir
touch
cp
mv
rm
```

---

# DevOps Relevance

These commands are used daily in:

- Linux Servers
- AWS EC2
- Docker Containers
- Kubernetes Nodes
- CI/CD Pipelines
- Log Analysis
- Infrastructure Automation

---

# Key Takeaways

- Linux is the backbone of DevOps.
- Learn navigation and file management first.
- `tail -f` is heavily used for log monitoring.
- `find` is one of the most important Linux commands.
- Understand permissions (`chmod`) and ownership (`chown`).
- Wildcards save significant time when managing large numbers of files.
