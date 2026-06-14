# Shell Scripting Basics for DevOps

## Overview

Shell scripting is one of the most important skills in DevOps. It allows engineers to automate repetitive tasks, manage servers, create deployment workflows, perform backups, monitor systems, and integrate tools within CI/CD pipelines.

Common DevOps use cases:

* Deployment automation
* CI/CD workflows
* System administration
* Log analysis
* Configuration management
* Infrastructure automation
* Monitoring and reporting

---

# Shebang and Script Execution

## Shebang Line

The first line of every shell script defines which interpreter should execute the script.

```bash
#!/bin/bash
```

### Why It Matters

* Specifies the shell interpreter.
* Improves portability.
* Prevents compatibility issues.
* Ensures scripts run consistently.

---

## Making Scripts Executable

```bash
chmod +x script.sh
```

Run the script:

```bash
./script.sh
```

Or execute directly using Bash:

```bash
bash script.sh
```

---

# Script Debugging

## Syntax Check

```bash
bash -n script.sh
```

Checks for syntax errors without executing.

---

## Verbose Debugging

```bash
bash -x script.sh
```

Displays each command before execution.

Enable inside a script:

```bash
set -x
```

Disable:

```bash
set +x
```

---

## Using Echo Statements

```bash
echo "Current value: $VAR"
```

Useful for troubleshooting script execution.

---

# Variables

Variables store data for reuse throughout the script.

## Variable Assignment

```bash
NAME="Shubham"
PORT=8080
```

**Important:** No spaces around `=`.

---

## Accessing Variables

```bash
echo $NAME
echo ${PORT}
```

Example:

```bash
APP_NAME="my-app"

echo "Deploying $APP_NAME"
```

---

## Quoting Variables

Recommended:

```bash
echo "$FILE_PATH"
```

Prevents issues with spaces and special characters.

---

# Special Variables

| Variable | Description                    |
| -------- | ------------------------------ |
| `$0`     | Script name                    |
| `$1`     | First argument                 |
| `$2`     | Second argument                |
| `$#`     | Number of arguments            |
| `$@`     | All arguments                  |
| `$*`     | All arguments as single string |
| `$?`     | Exit status of last command    |

Example:

```bash
./script.sh hello world
```

```bash
echo $1
echo $2
```

Output:

```text
hello
world
```

---

# Command Substitution

Command substitution captures command output.

## Preferred Syntax

```bash
$(command)
```

Example:

```bash
CURRENT_DATE=$(date)
USER=$(whoami)
CURRENT_DIR=$(pwd)
```

---

## Examples

```bash
echo "Today is $(date)"
```

```bash
BACKUP_DIR="backup_$(date +%Y%m%d)"
```

---

# Conditional Statements

Used for decision making.

---

## if Statement

```bash
if [ condition ]; then
    commands
fi
```

Example:

```bash
if [ -f config.txt ]; then
    echo "File exists"
fi
```

---

## if-else Statement

```bash
if [ condition ]; then
    commands
else
    commands
fi
```

Example:

```bash
if [ "$(id -u)" -eq 0 ]; then
    echo "Root User"
else
    echo "Normal User"
fi
```

---

## if-elif-else Statement

```bash
if [ condition1 ]; then
    commands
elif [ condition2 ]; then
    commands
else
    commands
fi
```

Example:

```bash
if [ "$ENV" = "production" ]; then
    echo "Production Environment"
elif [ "$ENV" = "staging" ]; then
    echo "Staging Environment"
else
    echo "Development Environment"
fi
```

---

# Common Test Operators

## File Checks

| Operator | Meaning          |
| -------- | ---------------- |
| `-e`     | File exists      |
| `-f`     | Regular file     |
| `-d`     | Directory exists |
| `-r`     | Readable         |
| `-w`     | Writable         |
| `-x`     | Executable       |

Example:

```bash
if [ -d logs ]; then
    echo "Directory exists"
fi
```

---

## String Comparisons

```bash
=      Equal
!=     Not Equal
-z     Empty String
-n     Non-empty String
```

Example:

```bash
if [ "$USER" = "root" ]; then
    echo "Administrator"
fi
```

---

## Numeric Comparisons

| Operator | Meaning               |
| -------- | --------------------- |
| `-eq`    | Equal                 |
| `-ne`    | Not Equal             |
| `-gt`    | Greater Than          |
| `-ge`    | Greater Than or Equal |
| `-lt`    | Less Than             |
| `-le`    | Less Than or Equal    |

Example:

```bash
if [ "$COUNT" -gt 10 ]; then
    echo "Limit exceeded"
fi
```

---

# DevOps Example

Environment-based deployment configuration:

```bash
#!/bin/bash

ENV=${1:-development}

if [ "$ENV" = "production" ]; then
    DB_HOST="prod-db"
    LOG_LEVEL="INFO"

elif [ "$ENV" = "staging" ]; then
    DB_HOST="stage-db"
    LOG_LEVEL="DEBUG"

else
    DB_HOST="localhost"
    LOG_LEVEL="TRACE"
fi

echo "Database: $DB_HOST"
echo "Log Level: $LOG_LEVEL"
```

Run:

```bash
./deploy.sh production
```

---

# Key Takeaways

* Use `#!/bin/bash` in every script.
* Make scripts executable using `chmod +x`.
* Use variables to create reusable scripts.
* Use command substitution `$(command)` to capture output.
* Use conditional statements for decision making.
* Debug scripts using `bash -x`.
* Shell scripting is a core DevOps skill used in automation, CI/CD, deployments, monitoring, and infrastructure management.
