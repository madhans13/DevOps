# 🧩 Linux Text Processing Commands — Complete Guide

A comprehensive guide to mastering Linux text manipulation commands for DevOps, system administration, and technical interviews.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)](https://www.linux.org/)
[![Shell](https://img.shields.io/badge/Shell-Bash-green.svg)](https://www.gnu.org/software/bash/)

---

## 📚 Table of Contents

- [cat - Concatenate and Display](#-1️⃣-cat--concatenate-and-display-files)
- [tac - Reverse cat](#-2️⃣-tac--reverse-cat)
- [head - View Beginning](#-3️⃣-head--view-beginning-of-file)
- [tail - View End](#-4️⃣-tail--view-end-of-file)
- [cut - Extract Fields](#-5️⃣-cut--extract-specific-fields-or-columns)
- [sort - Sort Lines](#-6️⃣-sort--sort-lines)
- [uniq - Remove Duplicates](#-7️⃣-uniq--remove-or-count-duplicates)
- [tr - Translate Characters](#-8️⃣-tr--translate-replace-or-delete-characters)
- [grep - Search Patterns](#-9️⃣-grep--search-text-by-pattern)
- [awk - Field Processing](#-🔟-awk--field-based-data-extractor)
- [sed - Stream Editor](#-1️⃣1️⃣-sed--stream-editor)
- [find - Search for Files](#-1️⃣2️⃣-find--search-for-files)
- [locate - Fast File Search](#-1️⃣3️⃣-locate--fast-file-search)
- [File Operations](#-1️⃣4️⃣-file-operations)
- [Real-World Examples](#-real-world-devops-examples)
- [Command Chaining](#-command-chaining--pipelines)
- [Quick Reference](#-quick-reference-card)

---

## 🧩 1️⃣ `cat` – Concatenate and Display Files

**Purpose**: Display, combine, or create text files.

### Syntax
```bash
cat [options] [file...]
```

### Common Options

| Option | Description |
|--------|-------------|
| `-n` | Number all output lines |
| `-b` | Number non-empty lines only |
| `-s` | Suppress repeated empty lines |
| `-E` | Show `$` at line ends |
| `-A` | Show all non-printing characters |

### Examples

```bash
# Display file contents
cat file.txt

# Merge two files
cat file1.txt file2.txt > merge.txt

# Show line numbers
cat -n file.txt

# Create a new file (Ctrl+D to save)
cat > notes.txt

# Append to existing file
cat >> existing.txt

# Display multiple files
cat file1.txt file2.txt file3.txt

# Show line endings
cat -E file.txt
```

### 💡 Real-World DevOps Use

```bash
# View log files
cat /var/log/nginx/access.log

# Filter failed requests
cat /var/log/nginx/access.log | grep "404"

# Combine config files
cat config1.conf config2.conf > merged.conf

# View Docker logs
cat /var/lib/docker/containers/*/container.log
```

---

## 🧩 2️⃣ `tac` – Reverse `cat`

Displays file contents from **last line to first**.

### Syntax
```bash
tac file.txt
```

### Use Cases

```bash
# View most recent logs first
tac /var/log/syslog

# Reverse order of any file
tac data.txt > reversed.txt

# Combined with grep
tac app.log | grep -m 5 "ERROR"  # Find last 5 errors
```

💡 **Useful for**: Viewing the most recent logs first (similar to `tail -r`).

---

## 🧩 3️⃣ `head` – View Beginning of File

**Purpose**: Show the first N lines of a file.

### Syntax
```bash
head [options] [file]
```

### Options

| Option | Description |
|--------|-------------|
| `-n N` | Show first N lines |
| `-c N` | Show first N bytes |
| `-q` | Quiet mode (no headers) |
| `-v` | Verbose mode (show filename) |

### Examples

```bash
# First 10 lines (default)
head file.txt

# First 20 lines
head -n 20 file.txt
head -20 file.txt  # Shorthand

# First 100 bytes
head -c 100 file.txt

# Multiple files
head file1.txt file2.txt

# Preview config files
head /etc/nginx/nginx.conf
```

### 💡 Use Case
Quickly preview config or log file headers without opening the entire file.

```bash
# Check CSV headers
head -1 data.csv

# Preview multiple config files
head -n 5 /etc/*.conf
```

---

## 🧩 4️⃣ `tail` – View End of File

**Purpose**: Show last N lines or follow updates in real-time.

### Syntax
```bash
tail [options] [file]
```

### Options

| Option | Description |
|--------|-------------|
| `-n N` | Show last N lines |
| `-f` | Follow file changes in real-time |
| `-F` | Follow with retry (if file rotates) |
| `-c N` | Show last N bytes |
| `--pid=PID` | Stop following when process dies |

### Examples

```bash
# Last 10 lines (default)
tail file.txt

# Last 20 lines
tail -n 20 syslog.log

# Follow log in real-time
tail -f /var/log/nginx/access.log

# Follow and show last 50 lines
tail -n 50 -f app.log

# Multiple files
tail -f /var/log/nginx/*.log
```

### 💡 DevOps Trick: Follow logs in real-time while deploying

```bash
# Monitor errors during deployment
tail -f /var/log/nginx/error.log | grep "fail"

# Follow multiple logs
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Follow with grep filter
tail -f app.log | grep --line-buffered "ERROR"

# Stop following when process stops
tail -f --pid=1234 app.log
```

---

## 🧩 5️⃣ `cut` – Extract Specific Fields or Columns

**Purpose**: Slice parts of each line based on delimiters or character positions.

### Syntax
```bash
cut OPTION... [FILE]
```

### Options

| Option | Description |
|--------|-------------|
| `-d` | Specify delimiter (default: TAB) |
| `-f` | Field numbers (e.g., `-f1,3` or `-f1-5`) |
| `-c` | Character positions |
| `-b` | Byte positions |
| `--complement` | Invert selection |

### Understanding Delimiters

**The delimiter (`-d`) tells `cut` where to split each line into fields.**

#### Example with `/etc/passwd`

```
root:x:0:0:root:/root:/bin/bash
```

The delimiter is `:` (colon). Fields are:
- Field 1: `root` (username)
- Field 2: `x` (password placeholder)
- Field 3: `0` (UID)
- Field 7: `/bin/bash` (shell)

```bash
# Extract usernames (field 1)
cut -d':' -f1 /etc/passwd
# Output: root, daemon, bin, sys...

# Extract shells (field 7)
cut -d':' -f7 /etc/passwd
# Output: /bin/bash, /usr/sbin/nologin...

# Multiple fields
cut -d':' -f1,3,7 /etc/passwd
# Output: root:0:/bin/bash
```

#### Example with CSV

```csv
name,email,age
madhan,madhan@gmail.com,22
karthik,karthik@yahoo.com,23
```

```bash
# Extract 2nd column (emails)
cut -d',' -f2 data.csv
# Output:
# email
# madhan@gmail.com
# karthik@yahoo.com

# Extract multiple columns
cut -d',' -f1,3 data.csv
# Output:
# name,age
# madhan,22
```

#### Character Mode (No Delimiter)

```bash
# First 5 characters of each line
cut -c1-5 names.txt
# Input:  "Madhan Kumar"
# Output: "Madha"

# Characters 1-10 and 15-20
cut -c1-10,15-20 file.txt

# Everything from character 5 onwards
cut -c5- file.txt
```

### Delimiter vs Character Mode Summary

| Mode | Option | Meaning | Example | Output |
|------|--------|---------|---------|--------|
| Field mode | `-d':' -f1` | Split by delimiter `:`, print field 1 | `/etc/passwd` | `root`, `daemon` |
| Field mode | `-d',' -f2` | Split by comma, print field 2 | `data.csv` | emails |
| Character mode | `-c1-5` | Print first 5 characters | `names.txt` | `Madha` |

### 💡 DevOps Examples

```bash
# Extract usernames from passwd
cut -d':' -f1 /etc/passwd | sort

# Extract IP addresses from log
cut -d' ' -f1 access.log | sort | uniq -c

# Parse CSV data
cut -d',' -f2,3 sales.csv

# Extract process IDs
ps aux | cut -c16-24

# Get specific columns from space-separated data
df -h | tail -n +2 | cut -d' ' -f1,5
```

---

## 🧩 6️⃣ `sort` – Sort Lines

**Purpose**: Sort lines alphabetically or numerically.

### Syntax
```bash
sort [options] [file]
```

### Options

| Option | Description |
|--------|-------------|
| `-r` | Reverse order |
| `-n` | Numerical sort |
| `-u` | Unique sort (remove duplicates) |
| `-t` | Specify delimiter |
| `-k` | Sort by specific field |
| `-h` | Human-readable numbers (K, M, G) |
| `-M` | Month name sort |
| `-R` | Random sort |

### Examples

```bash
# Basic alphabetical sort
sort names.txt

# Reverse sort
sort -r names.txt

# Numerical sort
sort -n numbers.txt

# Remove duplicates while sorting
sort -u names.txt

# Sort by specific column (CSV)
sort -t',' -k2 data.csv

# Sort by second field numerically
sort -t':' -k2 -n data.txt

# Human-readable size sort
du -h | sort -h

# Sort by multiple keys
sort -t',' -k2,2 -k3,3n data.csv
```

### 💡 Real-World Use

```bash
# Sort log entries
sort access.log

# Sort usernames
cut -d':' -f1 /etc/passwd | sort

# Find largest files
du -h /var/log/* | sort -h

# Sort IPs by frequency
cut -d' ' -f1 access.log | sort | uniq -c | sort -rn

# Sort processes by memory
ps aux | sort -k4 -rn | head -10
```

---

## 🧩 7️⃣ `uniq` – Remove or Count Duplicates

**Usually used with `sort`** because `uniq` only removes consecutive duplicates.

### Syntax
```bash
uniq [options] [file]
```

### Options

| Option | Description |
|--------|-------------|
| `-c` | Show count of occurrences |
| `-d` | Show only duplicates |
| `-u` | Show only unique lines |
| `-i` | Case-insensitive |
| `-f N` | Skip first N fields |

### Examples

```bash
# Remove consecutive duplicates
sort names.txt | uniq

# Count duplicates
sort names.txt | uniq -c

# Show only duplicates
sort names.txt | uniq -d

# Show only unique (non-duplicated) lines
sort names.txt | uniq -u

# Case-insensitive
sort names.txt | uniq -i

# Count and sort by frequency
sort data.txt | uniq -c | sort -rn
```

### 💡 DevOps Use

```bash
# Check for duplicate IPs in log
awk '{print $1}' access.log | sort | uniq -c | sort -rn

# Find most common error messages
grep "ERROR" app.log | sort | uniq -c | sort -rn

# Count unique visitors
cut -d' ' -f1 access.log | sort | uniq | wc -l

# Find duplicate usernames (shouldn't happen!)
cut -d':' -f1 /etc/passwd | sort | uniq -d
```

---

## 🧩 8️⃣ `tr` – Translate, Replace, or Delete Characters

**Purpose**: Character-based transformations.

### Syntax
```bash
tr [OPTION] SET1 [SET2]
```

### Options

| Option | Description |
|--------|-------------|
| `-d` | Delete characters |
| `-s` | Squeeze repeated characters |
| `-c` | Complement (invert) SET1 |

### Examples

```bash
# Convert to uppercase
echo "linux" | tr 'a-z' 'A-Z'
# Output: LINUX

# Convert to lowercase
echo "DEVOPS" | tr 'A-Z' 'a-z'
# Output: devops

# Delete vowels
echo "hello world" | tr -d 'aeiou'
# Output: hll wrld

# Replace spaces with newlines
cat file.txt | tr ' ' '\n'

# Squeeze repeated spaces
echo "hello    world" | tr -s ' '
# Output: hello world

# Delete digits
echo "abc123def456" | tr -d '0-9'
# Output: abcdef

# ROT13 encoding
echo "hello" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

### 💡 DevOps Tricks

```bash
# Normalize filenames (lowercase)
ls | tr '[:upper:]' '[:lower:]'

# Remove Windows line endings
cat file.txt | tr -d '\r' > unix_file.txt

# Convert tabs to spaces
cat file.txt | tr '\t' ' '

# Remove all non-alphanumeric
echo "hello@world!" | tr -cd '[:alnum:]'
# Output: helloworld

# Count words by converting spaces to newlines
cat file.txt | tr ' ' '\n' | wc -l
```

---

## 🧩 9️⃣ `grep` – Search Text by Pattern

**Purpose**: Find matching lines using regular expressions.

### Syntax
```bash
grep [options] pattern [file]
```

### Common Options

| Option | Description |
|--------|-------------|
| `-i` | Case-insensitive |
| `-r` or `-R` | Recursive search |
| `-v` | Invert match (show non-matching) |
| `-n` | Show line numbers |
| `-c` | Count matching lines |
| `-l` | Show only filenames |
| `-A N` | Show N lines after match |
| `-B N` | Show N lines before match |
| `-C N` | Show N lines around match |
| `-w` | Match whole word |
| `-E` | Extended regex (or use `egrep`) |
| `--color` | Highlight matches |

### Examples

```bash
# Basic search
grep "error" log.txt

# Case-insensitive
grep -i "failed" /var/log/*

# Invert match (lines without pattern)
grep -v "INFO" log.txt

# Show line numbers
grep -n "ERROR" app.log

# Recursive search
grep -r "nginx" /etc/

# Show context (2 lines before and after)
grep -C 2 "ERROR" log.txt

# Count matches
grep -c "success" results.txt

# Multiple patterns
grep -E "error|warning|critical" log.txt
# Or: egrep "error|warning|critical" log.txt

# Whole word match
grep -w "log" file.txt  # Won't match "login" or "catalog"

# Show only filenames
grep -l "TODO" *.py
```

### Regular Expressions

```bash
# Start of line
grep "^Error" log.txt

# End of line
grep "failed$" log.txt

# Any character
grep "h.t" file.txt  # Matches: hat, hit, hot

# Zero or more
grep "go*d" file.txt  # Matches: gd, god, good, goood

# One or more
grep -E "go+d" file.txt  # Matches: god, good, goood (not gd)

# Optional
grep -E "colou?r" file.txt  # Matches: color, colour

# Character class
grep "[0-9]" file.txt  # Lines with digits
grep "[a-z]" file.txt  # Lines with lowercase

# Negated character class
grep "[^0-9]" file.txt  # Lines without digits
```

### 💡 DevOps Examples

```bash
# Find errors with context
grep -A2 "ERROR" /var/log/syslog

# Search in compressed logs
zgrep "error" /var/log/syslog.*.gz

# Find IPs in logs
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log

# Exclude certain patterns
grep "error" log.txt | grep -v "timeout"

# Find configuration files
grep -r "database.host" /etc/

# Count 404 errors
grep "404" access.log | wc -l

# Real-time error monitoring
tail -f app.log | grep --line-buffered "ERROR"
```

---

## 🧩 🔟 `awk` – Field-Based Data Extractor

**Purpose**: Process and extract structured text (CSV, logs, tables). Works line-by-line, field-by-field.

### Syntax
```bash
awk 'pattern { action }' file
awk -F'delimiter' 'pattern { action }' file
```

### Built-in Variables

| Variable | Description |
|----------|-------------|
| `$0` | Entire line |
| `$1, $2, $3...` | Fields 1, 2, 3, etc. |
| `NF` | Number of fields |
| `NR` | Line number (row number) |
| `FS` | Field separator (default: space) |
| `OFS` | Output field separator |

### Basic Examples

```bash
# Print first and third fields
awk '{print $1, $3}' data.txt

# Use custom delimiter
awk -F: '{print $1}' /etc/passwd

# Print with custom output separator
awk -F: -v OFS="|" '{print $1, $3}' /etc/passwd

# Print line numbers
awk '{print NR, $0}' file.txt

# Print last field
awk '{print $NF}' file.txt

# Print everything except first field
awk '{$1=""; print $0}' file.txt
```

### Pattern Matching

```bash
# Lines containing "error"
awk '/error/ {print}' log.txt

# Lines NOT containing "info"
awk '!/info/ {print}' log.txt

# Specific field matches
awk '$3 == "ERROR" {print $0}' log.txt

# Numeric comparison
awk '$3 > 100 {print $1, $3}' data.txt

# Range of lines
awk 'NR==10,NR==20 {print}' file.txt
```

### Calculations

```bash
# Sum of second field
awk '{sum+=$2} END {print sum}' salary.txt

# Average
awk '{sum+=$2; count++} END {print sum/count}' data.txt

# Count lines
awk 'END {print NR}' file.txt

# Find maximum
awk 'BEGIN {max=0} {if($2>max) max=$2} END {print max}' data.txt
```

### Advanced Examples

```bash
# Print if more than 5 fields
awk 'NF > 5 {print $0}' file.txt

# Conditional formatting
awk '{if($3>100) print $1, "HIGH"; else print $1, "LOW"}' data.txt

# Multiple actions
awk '{print $1; print $2}' file.txt

# Skip header line
awk 'NR>1 {print $0}' data.csv

# Custom formatting
awk '{printf "%-10s %5d\n", $1, $2}' data.txt
```

### 💡 DevOps Examples

```bash
# Extract memory usage from ps
ps aux | awk '{print $1, $3}' | sort -k2 -rn | head

# Sum disk usage
df -h | awk 'NR>1 {sum+=$3} END {print sum}'

# Parse access logs
awk '{print $1}' access.log | sort | uniq -c | sort -rn

# Extract specific time range
awk '$4 ~ /10:00:/ {print $0}' access.log

# Calculate average response time
awk '{sum+=$NF; count++} END {print sum/count}' response_times.log

# Find long-running processes
ps aux | awk '$10 ~ /:[0-9]{2}:[0-9]{2}/ {print $2, $11}'

# CSV processing
awk -F',' '{sum+=$3} END {print "Total:", sum}' sales.csv

# Generate report
awk -F: '{users[$3]++} END {for(u in users) print u, users[u]}' /etc/passwd
```

---

## 🧩 1️⃣1️⃣ `sed` – Stream Editor

**Purpose**: Modify text using substitution patterns and stream operations.

### Syntax
```bash
sed [options] 'command' file
```

### Understanding `s/old/new/g`

Let's break down the substitution command:

```
s/error/issue/g
```

| Part | Meaning |
|------|---------|
| `s` | **Substitute** (the action sed will perform) |
| `error` | **Pattern** (the text you want to find) |
| `issue` | **Replacement** text |
| `g` | **Global** flag (replace all occurrences on each line) |

### The `g` Flag Explained

**Without `g`** (only first occurrence per line):
```bash
# Input:  error: failed to connect. error: timeout.
sed 's/error/issue/' file.txt
# Output: issue: failed to connect. error: timeout.
#         ^^^^^^ only first one changed
```

**With `g`** (all occurrences):
```bash
# Input:  error: failed to connect. error: timeout.
sed 's/error/issue/g' file.txt
# Output: issue: failed to connect. issue: timeout.
#         ^^^^^^                     ^^^^^^ both changed
```

### Common Flags

| Flag | Meaning |
|------|---------|
| `g` | Replace **all** matches on the line |
| `1`, `2`, `3`... | Replace only the **Nth** occurrence |
| `i` | Case-**insensitive** search |
| `p` | **Print** the line if substitution was made |
| `w file` | **Write** changed lines to file |

### Basic Substitution

```bash
# Simple replacement
sed 's/error/issue/' file.txt

# Global replacement
sed 's/error/issue/g' file.txt

# In-place edit (modify file directly)
sed -i 's/error/issue/g' file.txt

# Create backup before editing
sed -i.bak 's/error/issue/g' file.txt

# Case-insensitive
sed 's/error/issue/gi' file.txt
# Matches: error, Error, ERROR, eRRor

# Replace only 2nd occurrence
sed 's/error/issue/2' file.txt
```

### Deletion Commands

```bash
# Delete lines containing pattern
sed '/pattern/d' file.txt

# Delete blank lines
sed '/^$/d' file.txt

# Delete lines 5-10
sed '5,10d' file.txt

# Delete last line
sed '$d' file.txt

# Delete first line
sed '1d' file.txt
```

### Advanced Patterns

```bash
# Replace only on specific lines
sed '10,20s/old/new/g' file.txt

# Replace using regex
sed 's/[0-9]\+/NUMBER/g' file.txt

# Multiple commands
sed -e 's/foo/bar/g' -e 's/old/new/g' file.txt
# Or: sed 's/foo/bar/g; s/old/new/g' file.txt

# Replace with special characters
sed 's/\/usr\/local/\/opt/g' file.txt
# Better: use different delimiter
sed 's|/usr/local|/opt|g' file.txt

# Append text after match
sed '/pattern/a\New line of text' file.txt

# Insert text before match
sed '/pattern/i\New line of text' file.txt

# Change entire line
sed '/pattern/c\Replacement line' file.txt
```

### Using Different Delimiters

```bash
# Standard (using /)
sed 's/\/usr\/local\/bin/\/opt\/bin/g'

# Using | as delimiter (cleaner)
sed 's|/usr/local/bin|/opt/bin|g'

# Using @ as delimiter
sed 's@/usr/local/bin@/opt/bin@g'
```

### Print and Conditional Operations

```bash
# Print only changed lines
sed -n 's/error/issue/gp' file.txt

# Print specific lines
sed -n '10,20p' file.txt

# Print lines matching pattern
sed -n '/pattern/p' file.txt

# Print and substitute
sed -n 's/error/issue/gp' file.txt
```

### 💡 DevOps Examples

```bash
# Update configuration files
sed -i 's/DB_NAME=.*/DB_NAME=prod_db/' .env
sed -i 's/^#Port 22/Port 2222/' /etc/ssh/sshd_config

# Replace IP addresses
sed 's/192\.168\.1\.[0-9]\+/10.0.0.1/g' config.txt

# Remove comments
sed '/^#/d' config.conf

# Add line after match
sed '/\[mysqld\]/a max_connections = 500' my.cnf

# Replace multiple spaces with single space
sed 's/  \+/ /g' file.txt

# Remove trailing whitespace
sed 's/[[:space:]]*$//' file.txt

# Convert DOS to Unix line endings
sed -i 's/\r$//' file.txt

# Extract specific sections
sed -n '/START/,/END/p' file.txt

# Batch rename in files
for file in *.conf; do
    sed -i 's/old_server/new_server/g' "$file"
done

# Update version numbers
sed -i 's/version="1\.0\.0"/version="2.0.0"/' package.json

# Comment out lines
sed 's/^/# /' file.txt

# Uncomment lines
sed 's/^# //' file.txt
```

### Memory Trick

```
s → substitute
g → global (all occurrences)

's/old/new/g' = "replace ALL occurrences of old with new"
's/old/new/'  = "replace FIRST occurrence only"
's/old/new/2' = "replace SECOND occurrence only"
```

---

## 🧩 1️⃣2️⃣ `find` – Search for Files

**Purpose**: Search for files and directories based on various criteria and perform actions on results.

### Syntax
```bash
find [path] [options] [expression] [action]
```

### Basic Search by Name

```bash
# Find by exact name
find /path -name "filename.txt"

# Case-insensitive search
find /path -iname "filename.txt"

# Find with wildcard
find /path -name "*.log"
find /path -name "file*"

# Find in current directory
find . -name "*.py"

# Find in multiple directories
find /var/log /tmp -name "*.log"
```

### Search by Type

| Option | Type |
|--------|------|
| `-type f` | Regular files |
| `-type d` | Directories |
| `-type l` | Symbolic links |
| `-type b` | Block devices |
| `-type c` | Character devices |

```bash
# Find only files
find /path -type f -name "*.txt"

# Find only directories
find /path -type d -name "logs"

# Find symbolic links
find /path -type l
```

### Search by Size

| Option | Meaning |
|--------|---------|
| `-size +100M` | Larger than 100MB |
| `-size -10M` | Smaller than 10MB |
| `-size 50M` | Exactly 50MB |
| `c` | bytes |
| `k` | kilobytes |
| `M` | megabytes |
| `G` | gigabytes |

```bash
# Files larger than 100MB
find /var/log -type f -size +100M

# Files smaller than 1KB
find /tmp -type f -size -1k

# Empty files
find /path -type f -empty

# Empty directories
find /path -type d -empty
```

### Search by Time

| Option | Meaning |
|--------|---------|
| `-mtime n` | Modified n days ago |
| `-mtime +n` | Modified more than n days ago |
| `-mtime -n` | Modified less than n days ago |
| `-atime n` | Accessed n days ago |
| `-ctime n` | Changed n days ago |
| `-mmin n` | Modified n minutes ago |

```bash
# Modified in last 7 days
find /path -type f -mtime -7

# Modified more than 30 days ago
find /var/log -type f -mtime +30

# Modified in last 60 minutes
find /tmp -type f -mmin -60

# Accessed today
find /path -type f -atime 0

# Not accessed in 90 days
find /home -type f -atime +90
```

### Search by Permissions

```bash
# Find files with specific permissions
find /path -type f -perm 644

# Find files with at least these permissions
find /path -type f -perm -644

# Find executable files
find /path -type f -perm /111

# Find files writable by others
find /path -type f -perm /002

# Find SUID files
find / -type f -perm -4000

# Find SGID files
find / -type f -perm -2000
```

### Search by Owner

```bash
# Find by owner
find /path -user madhan

# Find by group
find /path -group developers

# Find files not owned by specific user
find /path ! -user root

# Find files with no owner
find /path -nouser

# Find files with no group
find /path -nogroup
```

### Combining Conditions

```bash
# AND condition (default)
find /path -type f -name "*.log" -size +10M

# OR condition
find /path -type f \( -name "*.log" -o -name "*.txt" \)

# NOT condition
find /path -type f ! -name "*.log"

# Complex conditions
find /var/log -type f -name "*.log" -size +50M -mtime +30
```

### Actions on Found Files

| Action | Description |
|--------|-------------|
| `-print` | Print file path (default) |
| `-delete` | Delete found files |
| `-exec` | Execute command on each file |
| `-ok` | Execute with confirmation |
| `-ls` | List in `ls -l` format |

```bash
# Delete found files
find /tmp -type f -name "*.tmp" -delete

# Execute command on each file
find /path -type f -name "*.log" -exec gzip {} \;

# Execute command with confirmation
find /path -type f -name "*.old" -ok rm {} \;

# Execute command on multiple files at once
find /path -type f -name "*.txt" -exec grep "pattern" {} +

# List details of found files
find /path -type f -name "*.conf" -ls

# Copy found files
find /source -type f -name "*.pdf" -exec cp {} /backup/ \;

# Change permissions
find /path -type f -name "*.sh" -exec chmod +x {} \;

# Change ownership
find /var/www -type f -exec chown www-data:www-data {} \;
```

### Advanced Examples

```bash
# Find and count files
find /path -type f -name "*.log" | wc -l

# Find files and show sizes
find /var/log -type f -name "*.log" -exec du -h {} \; | sort -h

# Find and archive
find /path -type f -name "*.txt" -exec tar -czf backup.tar.gz {} +

# Find duplicate files
find /path -type f -exec md5sum {} \; | sort | uniq -w32 -D

# Find files modified today
find /var/log -type f -mtime 0 -ls

# Find large files and sort
find /home -type f -size +100M -exec ls -lh {} \; | sort -k5 -h

# Find files by extension and move them
find /downloads -type f -name "*.jpg" -exec mv {} /pictures/ \;

# Find and replace content
find /path -type f -name "*.conf" -exec sed -i 's/old/new/g' {} \;

# Find broken symbolic links
find /path -type l ! -exec test -e {} \; -print

# Find files with specific content
find /path -type f -name "*.txt" -exec grep -l "pattern" {} \;
```

### Performance Tips

```bash
# Limit search depth
find /path -maxdepth 2 -name "*.log"
find /path -mindepth 1 -maxdepth 3 -type f

# Exclude directories
find /path -type f -name "*.log" -not -path "*/node_modules/*"

# Use -prune to skip directories
find /path -path "*/cache/*" -prune -o -type f -name "*.log" -print

# Faster than -exec for single operations
find /path -type f -name "*.log" -delete
# vs slower:
find /path -type f -name "*.log" -exec rm {} \;
```

### 💡 Real-World DevOps Use Cases

```bash
# Clean old log files
find /var/log -type f -name "*.log" -mtime +30 -delete

# Find and compress old logs
find /var/log -type f -name "*.log" -mtime +7 -exec gzip {} \;

# Find configuration files
find /etc -type f -name "*.conf"

# Security audit: find world-writable files
find / -type f -perm -002 -ls

# Find SUID/SGID files (potential security risk)
find / -type f \( -perm -4000 -o -perm -2000 \) -ls

# Find large files consuming disk space
find /var -type f -size +500M -exec ls -lh {} \;

# Find files modified in last hour (debugging)
find /var/log -type f -mmin -60

# Backup specific files
find /home -type f -name "*.doc" -exec cp {} /backup/ \;

# Find and remove empty directories
find /tmp -type d -empty -delete

# Find core dumps
find / -type f -name "core.*" -o -name "core"

# Find all shell scripts
find /usr/local/bin -type f -name "*.sh"

# Clean docker logs
find /var/lib/docker/containers -name "*.log" -mtime +7 -delete

# Find files owned by deleted users
find /home -nouser -ls
```

---

## 🧩 1️⃣3️⃣ `locate` – Fast File Search

**Purpose**: Quickly find files by name using a pre-built database.

### Syntax
```bash
locate [options] pattern
```

### How It Works

- Uses a database (`/var/lib/mlocate/mlocate.db`)
- Database updated daily via cron (or manually with `updatedb`)
- Much faster than `find` for name-based searches
- Doesn't search in real-time

### Basic Usage

```bash
# Find files by name
locate filename.txt

# Case-insensitive search
locate -i filename.txt

# Limit results
locate -n 10 pattern

# Show only existing files (check if still exists)
locate -e filename

# Count matches
locate -c "*.log"

# Show statistics
locate -S
```

### Examples

```bash
# Find all Python files
locate "*.py"

# Find nginx configs
locate nginx.conf

# Find in specific directory
locate "/etc/*.conf"

# Use regex
locate -r "\.log$"

# Case-insensitive
locate -i README

# Count matching files
locate -c "*.jpg"
```

### Update Database

```bash
# Update database manually
sudo updatedb

# Update specific paths only
sudo updatedb --localpaths='/home /var'

# Exclude paths
sudo updatedb --prunepaths='/tmp /var/cache'
```

### `locate` vs `find`

| Feature | `locate` | `find` |
|---------|----------|--------|
| **Speed** | Very fast | Slower |
| **Real-time** | No (uses database) | Yes (searches live) |
| **Search by** | Name only | Name, size, time, permissions, etc. |
| **Actions** | None | Can execute commands |
| **New files** | Not found until DB update | Found immediately |
| **Best for** | Quick name lookups | Complex searches |

### 💡 When to Use Which?

**Use `locate` when:**
- ✅ You know the filename
- ✅ You need results quickly
- ✅ You don't care if file is very recent
- ✅ Simple name-based search

**Use `find` when:**
- ✅ You need real-time search
- ✅ You need to search by size, time, permissions
- ✅ You need to perform actions on found files
- ✅ You need complex search criteria

```bash
# Quick filename lookup
locate nginx.conf

# Complex search with actions
find /etc -name "*.conf" -type f -mtime -7 -exec grep "ssl" {} \;
```

---

## 🧩 1️⃣4️⃣ File Operations

### Creating Files and Directories

```bash
# Create empty file
touch file.txt
touch file1.txt file2.txt file3.txt

# Create file with content
echo "Hello World" > file.txt
cat > file.txt << EOF
Line 1
Line 2
EOF

# Create directory
mkdir directory_name

# Create nested directories
mkdir -p path/to/nested/directory

# Create multiple directories
mkdir dir1 dir2 dir3

# Create with specific permissions
mkdir -m 755 public_folder
```

### Copying Files

```bash
# Copy file
cp source.txt destination.txt

# Copy to directory
cp file.txt /path/to/directory/

# Copy multiple files
cp file1.txt file2.txt /destination/

# Copy directory recursively
cp -r source_dir/ destination_dir/

# Copy with attributes preserved
cp -p file.txt backup.txt

# Interactive copy (ask before overwrite)
cp -i file.txt existing.txt

# Verbose copy
cp -v file.txt backup.txt

# Copy only if source is newer
cp -u source.txt destination.txt

# Create backup of destination
cp -b file.txt existing.txt
```

### Moving and Renaming

```bash
# Rename file
mv oldname.txt newname.txt

# Move file
mv file.txt /path/to/directory/

# Move multiple files
mv file1.txt file2.txt /destination/

# Move directory
mv old_dir/ new_location/

# Interactive move
mv -i file.txt destination.txt

# Don't overwrite existing
mv -n file.txt existing.txt

# Create backup if exists
mv -b file.txt existing.txt

# Verbose move
mv -v source destination
```

### Deleting Files

```bash
# Remove file
rm file.txt

# Remove multiple files
rm file1.txt file2.txt file3.txt

# Interactive removal (ask before delete)
rm -i file.txt

# Force removal (no prompt)
rm -f file.txt

# Remove directory and contents
rm -r directory/

# Force remove directory
rm -rf directory/

# Verbose deletion
rm -v file.txt

# Remove empty directory
rmdir empty_directory/

# Remove nested empty directories
rmdir -p path/to/empty/dirs/
```

### Viewing Files

```bash
# Display entire file
cat file.txt

# Display with line numbers
cat -n file.txt

# View file page by page
less file.txt
more file.txt

# View with search capability
less file.txt  # Press / to search

# View first lines
head file.txt
head -n 20 file.txt

# View last lines
tail file.txt
tail -n 50 file.txt

# Follow file updates
tail -f logfile.log
```

### File Information

```bash
# File type
file filename.txt

# Detailed information
stat filename.txt

# Show file size
ls -lh file.txt
du -h file.txt

# Count lines, words, characters
wc file.txt
wc -l file.txt  # Just lines
wc -w file.txt  # Just words
wc -c file.txt  # Just bytes

# Show disk usage
du -sh directory/
du -h --max-depth=1 directory/

# Check file exists
test -f file.txt && echo "Exists" || echo "Not found"
[ -f file.txt ] && echo "Exists"
```

### Comparing Files

```bash
# Compare two files
diff file1.txt file2.txt

# Side-by-side comparison
diff -y file1.txt file2.txt

# Unified format (like git diff)
diff -u file1.txt file2.txt

# Compare directories
diff -r dir1/ dir2/

# Ignore whitespace
diff -w file1.txt file2.txt

# Show only if files differ
diff -q file1.txt file2.txt

# Compare using cmp
cmp file1.txt file2.txt
```

### Linking Files

```bash
# Create hard link
ln original.txt hardlink.txt

# Create symbolic (soft) link
ln -s /path/to/original.txt symlink.txt

# Create symbolic link to directory
ln -s /path/to/directory/ link_name

# Force link creation (overwrite if exists)
ln -sf original.txt link.txt

# Show where symbolic link points
readlink symlink.txt
readlink -f symlink.txt  # Show absolute path
```

### Archiving and Compression

```bash
# Create tar archive
tar -cf archive.tar files/

# Create compressed tar.gz
tar -czf archive.tar.gz files/

# Create compressed tar.bz2
tar -cjf archive.tar.bz2 files/

# Extract tar archive
tar -xf archive.tar

# Extract tar.gz
tar -xzf archive.tar.gz

# Extract to specific directory
tar -xzf archive.tar.gz -C /destination/

# View contents without extracting
tar -tzf archive.tar.gz

# Compress with gzip
gzip file.txt  # Creates file.txt.gz

# Decompress gzip
gunzip file.txt.gz

# Compress with bzip2
bzip2 file.txt

# Decompress bzip2
bunzip2 file.txt.bz2

# Create zip archive
zip archive.zip file1.txt file2.txt

# Create zip recursively
zip -r archive.zip directory/

# Extract zip
unzip archive.zip

# Extract zip to directory
unzip archive.zip -d /destination/
```

### File Permissions

```bash
# Change permissions (numeric)
chmod 755 file.sh
chmod 644 file.txt

# Change permissions (symbolic)
chmod u+x file.sh      # Add execute for user
chmod g-w file.txt     # Remove write from group
chmod o=r file.txt     # Set others to read only
chmod a+r file.txt     # Add read for all

# Change recursively
chmod -R 755 directory/

# Change ownership
chown user:group file.txt
chown user file.txt
chown :group file.txt

# Change ownership recursively
chown -R user:group directory/

# Change group only
chgrp groupname file.txt
```

### Searching Within Files

```bash
# Search for pattern
grep "pattern" file.txt

# Case-insensitive search
grep -i "pattern" file.txt

# Recursive search in directory
grep -r "pattern" /path/

# Show line numbers
grep -n "pattern" file.txt

# Show files containing pattern
grep -l "pattern" *.txt

# Count matches
grep -c "pattern" file.txt

# Show context (lines around match)
grep -C 2 "pattern" file.txt
```

### Bulk Operations

```bash
# Rename multiple files (add prefix)
for file in *.txt; do
    mv "$file" "backup_$file"
done

# Change extension for all files
for file in *.txt; do
    mv "$file" "${file%.txt}.md"
done

# Copy all files with pattern
cp *.log /backup/

# Delete all files with pattern
rm *.tmp

# Find and delete old files
find /tmp -type f -mtime +30 -delete

# Bulk permission change
find /var/www -type f -exec chmod 644 {} \;
find /var/www -type d -exec chmod 755 {} \;

# Mass rename with sed
for file in *; do
    newname=$(echo "$file" | sed 's/old/new/g')
    mv "$file" "$newname"
done
```

### 💡 DevOps File Operations

```bash
# Backup configuration files
cp -p /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak.$(date +%F)

# Clean old backups
find /backups -type f -name "*.bak" -mtime +30 -delete

# Deploy application files
cp -r /builds/app/* /var/www/html/
chown -R www-data:www-data /var/www/html/
chmod -R 755 /var/www/html/

# Create deployment archive
tar -czf deploy-$(date +%F).tar.gz app/

# Extract and deploy
tar -xzf deploy.tar.gz -C /opt/app/

# Find and replace in multiple files
find /etc/app -type f -name "*.conf" -exec sed -i 's/old_host/new_host/g' {} \;

# Copy with progress
rsync -avh --progress source/ destination/

# Mirror directories
rsync -av --delete source/ destination/

# Sync to remote server
rsync -avz -e ssh local/ user@remote:/path/

# Find large log files and compress
find /var/log -type f -size +100M -name "*.log" -exec gzip {} \;

# Clean temporary files
find /tmp -type f -atime +7 -delete
find /tmp -type d -empty -delete

# Safe file deletion (move to trash)
mkdir -p ~/.trash
mv unwanted_file.txt ~/.trash/

# Disk cleanup script
#!/bin/bash
echo "Cleaning old logs..."
find /var/log -name "*.log.*" -mtime +30 -delete
echo "Cleaning temp files..."
find /tmp -type f -atime +7 -delete
echo "Cleaning package cache..."
apt-get clean
echo "Done!"
```

---

## 🚀 Real-World DevOps Examples

### 1. Log Analysis Pipeline

```bash
# Find top 10 IPs accessing your server
cat access.log \
  | awk '{print $1}' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -10

# Find 404 errors with context
grep "404" access.log \
  | awk '{print $1, $7}' \
  | sort \
  | uniq -c \
  | sort -rn

# Calculate average response time
awk '{sum+=$NF; count++} END {print "Average:", sum/count, "ms"}' response_times.log

# Find all error logs in last 7 days
find /var/log -type f -name "*.log" -mtime -7 -exec grep -l "ERROR" {} \;
```

### 2. System Monitoring

```bash
# Find top memory-consuming processes
ps aux \
  | awk '{print $4, $11}' \
  | sort -rn \
  | head -10

# Monitor specific service logs
tail -f /var/log/nginx/error.log \
  | grep --line-buffered "fail" \
  | while read line; do
      echo "[ALERT] $line"
      # Send notification
    done

# Find large files consuming disk
find /var -type f -size +500M -exec ls -lh {} \; | sort -k5 -h
```

### 3. Configuration Management

```bash
# Bulk update database configs
find /etc/app/ -name "*.conf" -exec \
  sed -i 's/db.old.com/db.new.com/g' {} \;

# Update environment variables
sed -i 's/ENV=dev/ENV=prod/g' .env
sed -i 's/DEBUG=true/DEBUG=false/g' .env

# Generate host list from /etc/hosts
grep -v "^#" /etc/hosts \
  | awk '{print $2}' \
  | sort -u

# Backup configs before changes
find /etc/nginx -name "*.conf" -exec cp {} {}.bak \;
```

### 4. Data Processing

```bash
# CSV: Extract specific columns and calculate sum
awk -F',' 'NR>1 {sum+=$3} END {print "Total Sales:", sum}' sales.csv

# Process JSON logs (basic extraction)
grep "error" app.json \
  | sed 's/.*"message":"\([^"]*\)".*/\1/' \
  | sort \
  | uniq -c

# Clean data: remove duplicates and empty lines
sort data.txt | uniq | sed '/^$/d' > cleaned.txt
```

### 5. Security Auditing

```bash
# Find failed SSH login attempts
grep "Failed password" /var/log/auth.log \
  | awk '{print $(NF-3)}' \
  | sort \
  | uniq -c \
  | sort -rn

# Extract and analyze sudo usage
grep "sudo:" /var/log/auth.log \
  | awk '{print $5, $6, $14}' \
  | sort \
  | uniq -c

# Find world-writable files (security risk)
find / -type f -perm -002 -ls 2>/dev/null

# Find SUID files
find / -type f -perm -4000 -ls 2>/dev/null
```

### 6. Deployment Automation

```bash
# Clean and deploy application
#!/bin/bash
echo "Cleaning old builds..."
find /builds -type f -name "*.tar.gz" -mtime +7 -delete

echo "Deploying application..."
tar -xzf /builds/app-latest.tar.gz -C /opt/app/

echo "Setting permissions..."
chown -R app:app /opt/app/
find /opt/app -type f -exec chmod 644 {} \;
find /opt/app -type d -exec chmod 755 {} \;

echo "Restarting service..."
systemctl restart myapp

echo "Deployment complete!"
```

### 7. Log Rotation and Cleanup

```bash
# Compress old logs
find /var/log -type f -name "*.log" -mtime +7 -exec gzip {} \;

# Delete very old compressed logs
find /var/log -type f -name "*.log.gz" -mtime +30 -delete

# Archive logs by month
#!/bin/bash
MONTH=$(date -d "last month" +%Y-%m)
find /var/log/app -name "*.log" -type f -mtime +30 \
  -exec tar -czf /backups/logs-${MONTH}.tar.gz {} + \
  -delete
```

### 8. Database Backup Scripts

```bash
# Automated MySQL backup
#!/bin/bash
BACKUP_DIR="/backups/mysql"
DATE=$(date +%F)

# Create backup
mysqldump -u root -p"$PASSWORD" --all-databases | gzip > "$BACKUP_DIR/backup-$DATE.sql.gz"

# Keep only last 7 days
find "$BACKUP_DIR" -name "backup-*.sql.gz" -mtime +7 -delete

# Log the backup
echo "$(date): Backup completed" >> /var/log/backup.log
```

---

## 🔗 Command Chaining & Pipelines

### Common Patterns

```bash
# Pattern: Extract → Sort → Count → Top N
command | awk '{print $1}' | sort | uniq -c | sort -rn | head -10

# Pattern: Search → Filter → Transform
grep "pattern" file | sed 's/old/new/g' | awk '{print $2}'

# Pattern: Multiple file processing
cat file1 file2 file3 | sort | uniq

# Pattern: Conditional processing
grep "error" log.txt | while read line; do
    # Process each line
    echo "Processing: $line"
done
```

### Combining Multiple Commands

```bash
# Complex log analysis
cat access.log \
  | grep -v "bot" \
  | awk '{print $1, $7}' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -20 \
  | awk '{printf "%-6s %s %s\n", $1, $2, $3}'

# Multi-stage data processing
cat sales.csv \
  | tail -n +2 \
  | cut -d',' -f2,3 \
  | sed 's/,/ /g' \
  | awk '{sum+=$2} END {print "Total:", sum}'

# Nested filtering
ps aux \
  | grep -v grep \
  | grep "nginx" \
  | awk '{print $2, $11}' \
  | sort -k2
```

---

## 📋 Quick Reference Card

### One-Liners Cheat Sheet

```bash
# FILE VIEWING
cat file.txt                    # Display file
tac file.txt                    # Reverse display
head -n 20 file.txt            # First 20 lines
tail -n 20 file.txt            # Last 20 lines
tail -f file.txt               # Follow in real-time

# EXTRACTION
cut -d':' -f1 /etc/passwd      # Extract field
cut -c1-10 file.txt            # Extract characters
awk '{print $1, $3}' file      # Extract columns
grep "pattern" file            # Search pattern

# TRANSFORMATION
tr 'a-z' 'A-Z'                 # Convert case
sed 's/old/new/g' file         # Replace text
sort file.txt                  # Sort lines
uniq -c                        # Count duplicates

# COMBINATION
cat file | grep "error" | wc -l                    # Count errors
cut -d',' -f1 file.csv | sort | uniq              # Unique values
tail -f log | grep --line-buffered "ERROR"        # Live monitoring
awk '{print $1}' log | sort | uniq -c | sort -rn  # Top IPs
```

---

## 🎯 Summary Table

| Command | Key Use | Common Options | Example |
|---------|---------|----------------|---------|
| `cat` | Display or merge files | `-n`, `-s`, `-E` | `cat file.txt` |
| `tac` | Reverse display | - | `tac log.txt` |
| `head` | Show file start | `-n`, `-c` | `head -20 file.txt` |
| `tail` | Show file end | `-n`, `-f` | `tail -f app.log` |
| `cut` | Extract columns | `-d`, `-f`, `-c` | `cut -d':' -f1 /etc/passwd` |
| `sort` | Sort lines | `-r`, `-n`, `-u` | `sort -n numbers.txt` |
| `uniq` | Remove duplicates | `-c`, `-d`, `-u` | `sort file \| uniq -c` |
| `tr` | Character transform | `-d`, `-s` | `tr 'a-z' 'A-Z'` |
| `grep` | Search patterns | `-i`, `-r`, `-v` | `grep -i "error" log` |
| `awk` | Field-based extraction | `-F` | `awk '{print $1}'` |
| `sed` | Stream-based editing | `-i`, `-n` | `sed 's/old/new/g'` |
| `find` | Search for files | `-name`, `-type`, `-size` | `find / -name "*.log"` |
| `locate` | Fast name-based search | `-i`, `-n` | `locate nginx.conf` |

### File Operations

| Command | Purpose | Example |
|---------|---------|---------|
| `touch` | Create file | `touch file.txt` |
| `mkdir` | Create directory | `mkdir -p path/to/dir` |
| `cp` | Copy | `cp -r source/ dest/` |
| `mv` | Move/rename | `mv old.txt new.txt` |
| `rm` | Remove | `rm -rf directory/` |
| `ln` | Create link | `ln -s target link` |
| `tar` | Archive | `tar -czf file.tar.gz dir/` |
| `chmod` | Change permissions | `chmod 755 script.sh` |
| `chown` | Change owner | `chown user:group file` |

---

## 🧠 Interview Quick Tips

### Common Questions & Answers

**Q: What's the difference between `cat` and `tac`?**  
**A:** `cat` displays from first to last line, `tac` displays from last to first line. `tac` is useful for viewing recent log entries first.

**Q: When should you use `head` vs `tail`?**  
**A:** Use `head` to preview the beginning of a file (like headers or config starts). Use `tail` to view the end (like recent log entries) or follow live updates with `tail -f`.

**Q: Explain the delimiter in `cut -d':' -f1`**  
**A:** The `-d` specifies the delimiter (`:` in this case). `cut` splits each line at this delimiter and `-f1` extracts the first field. For `/etc/passwd`, this extracts usernames.

**Q: Why use `sort` before `uniq`?**  
**A:** `uniq` only removes consecutive duplicate lines. `sort` arranges all duplicates together, allowing `uniq` to properly identify and remove them.

**Q: What does `sed 's/old/new/g'` do?**  
**A:** It's a substitution command. `s` means substitute, `old` is the pattern to find, `new` is the replacement, and `g` means global (replace all occurrences on each line, not just the first).

**Q: What's the difference between `find` and `locate`?**  
**A:** `find` searches the filesystem in real-time and can search by name, size, time, permissions, and execute actions. `locate` searches a pre-built database (updated daily) and is much faster but only searches by name and doesn't show newly created files until the database is updated.

**Q: How do you find files larger than 100MB?**  
**A:** `find /path -type f -size +100M` - the `+` means greater than, `-` means less than.

**Q: How do you safely delete files in production?**  
**A:** Use `find` with careful conditions, test with `-print` first, then use `-delete`. Or use `mv` to move files to a trash directory first: `mv file ~/.trash/` before permanent deletion.

**Q: What's the difference between hard and soft links?**  
**A:** Hard links point to the same inode (same file, different name), work only on same filesystem, and file remains if original deleted. Soft links (symbolic) are shortcuts that store the path, can cross filesystems, but break if original is deleted.

---

## 🔍 Performance Tips

### Best Practices

1. **Use the right tool for the job**
   - Simple display: `cat`
   - Pattern search: `grep`
   - Field extraction: `awk` or `cut`
   - Text transformation: `sed`

2. **Avoid unnecessary cats**
   ```bash
   # Bad (useless use of cat)
   cat file.txt | grep "pattern"
   
   # Good
   grep "pattern" file.txt
   ```

3. **Optimize pipelines**
   ```bash
   # Filter early in the pipeline
   grep "pattern" huge_file.txt | awk '{print $1}' | sort
   
   # Not: cat huge_file.txt | awk '{print $1}' | sort | grep "pattern"
   ```

4. **Use appropriate options**
   ```bash
   # For large files, limit output early
   head -1000 large.log | grep "error"
   
   # Use -m to stop after N matches
   grep -m 10 "pattern" file.txt
   ```

5. **In-place editing**
   ```bash
   # Create backup automatically
   sed -i.bak 's/old/new/g' file.txt
   ```

---

## 🛠️ Advanced Techniques

### 1. Process Substitution

```bash
# Compare outputs of two commands
diff <(sort file1.txt) <(sort file2.txt)

# Multiple inputs to awk
awk '{print $1}' <(grep "error" log1) <(grep "error" log2)
```

### 2. Command Substitution

```bash
# Use output as argument
grep "error" $(find /var/log -name "*.log")

# With backticks (older syntax)
grep "error" `find /var/log -name "*.log"`
```

### 3. Here Documents

```bash
# Multi-line input to command
cat << EOF > config.txt
server {
    listen 80;
    server_name example.com;
}
EOF

# With sed
sed 's/old/new/g' << EOF
line 1 with old text
line 2 with old text
EOF
```

### 4. While Loops with Read

```bash
# Process each line
cat users.txt | while read user; do
    echo "Processing: $user"
    id "$user"
done

# With multiple fields
cat data.csv | while IFS=',' read name email age; do
    echo "Name: $name, Email: $email"
done
```

### 5. Arrays and Loops

```bash
# Store grep results in array
mapfile -t errors < <(grep "ERROR" log.txt)

# Process array
for error in "${errors[@]}"; do
    echo "Found: $error"
done
```

---

## 🎓 Practice Exercises

### Beginner Level

1. Display the first 15 lines of `/etc/passwd`
2. Show the last 25 lines of `/var/log/syslog`
3. Extract all usernames from `/etc/passwd`
4. Count how many users exist on the system
5. Convert a text file to all uppercase

**Solutions:**
```bash
# 1
head -15 /etc/passwd

# 2
tail -25 /var/log/syslog

# 3
cut -d':' -f1 /etc/passwd

# 4
cut -d':' -f1 /etc/passwd | wc -l

# 5
cat file.txt | tr 'a-z' 'A-Z'
```

### Intermediate Level

1. Find top 10 most common words in a text file
2. Extract all IP addresses from an access log
3. Count unique visitors to your website
4. Find and replace all occurrences of a word in multiple files
5. Display lines 20-30 from a file

**Solutions:**
```bash
# 1
cat file.txt | tr ' ' '\n' | sort | uniq -c | sort -rn | head -10

# 2
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' access.log

# 3
awk '{print $1}' access.log | sort | uniq | wc -l

# 4
find . -name "*.txt" -exec sed -i 's/old/new/g' {} \;

# 5
sed -n '20,30p' file.txt
# Or: head -30 file.txt | tail -11
```

### Advanced Level

1. Create a script to monitor failed SSH login attempts
2. Parse CSV and generate summary report
3. Real-time log monitoring with alerts
4. Extract and analyze HTTP response codes
5. Find duplicate files based on content

**Solutions:**
```bash
# 1
#!/bin/bash
tail -f /var/log/auth.log | grep --line-buffered "Failed password" | \
while read line; do
    ip=$(echo "$line" | awk '{print $(NF-3)}')
    echo "[ALERT] Failed login from: $ip"
done

# 2
#!/bin/bash
echo "Sales Summary Report"
echo "===================="
awk -F',' 'NR>1 {
    sum+=$3; 
    count++; 
    if($3>max) max=$3
} 
END {
    print "Total Sales:", sum
    print "Average:", sum/count
    print "Maximum:", max
}' sales.csv

# 3
#!/bin/bash
tail -f /var/log/nginx/error.log | grep --line-buffered "error" | \
while read line; do
    echo "[$(date)] ERROR: $line" >> alerts.log
    # Send notification (e.g., email, Slack)
done

# 4
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# 5
find . -type f -exec md5sum {} \; | sort | uniq -w32 -D --all-repeated=separate
```

---

## 📚 Additional Resources

### Official Documentation
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- [GNU Grep Manual](https://www.gnu.org/software/grep/manual/)
- [GNU Sed Manual](https://www.gnu.org/software/sed/manual/)
- [GNU Awk Manual](https://www.gnu.org/software/gawk/manual/)

### Learning Resources
- [The Linux Command Line (Book)](http://linuxcommand.org/tlcl.php)
- [Bash Guide for Beginners](https://tldp.org/LDP/Bash-Beginners-Guide/html/)
- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)
- [Regular-Expressions.info](https://www.regular-expressions.info/)

### Online Tools
- [Regex101](https://regex101.com/) - Test regular expressions
- [Explain Shell](https://explainshell.com/) - Understand shell commands
- [ShellCheck](https://www.shellcheck.net/) - Lint your scripts

---

## 💡 Pro Tips

1. **Use `man` pages**: Always check `man command` for detailed options
2. **Test with sample data**: Never run destructive commands on production first
3. **Use `-i.bak` with sed**: Always create backups when editing in-place
4. **Quote variables**: Use `"$var"` to prevent word splitting issues
5. **Use `set -e` in scripts**: Exit on first error
6. **Comment your pipelines**: Complex one-liners should be documented
7. **Learn regular expressions**: They make grep, sed, and awk much more powerful
8. **Practice daily**: Use these commands in daily tasks to build muscle memory

---

## 🤝 Contributing

Found an error or have a suggestion? Contributions are welcome!

- Report issues
- Suggest improvements
- Add more examples
- Share your use cases

---

## 📝 License

This guide is provided under the MIT License. Feel free to use, share, and modify!

---

## ⭐ Quick Command Reference

```bash
# Display
cat file.txt                              # Show content
tac file.txt                              # Reverse show
head -n 20 file.txt                       # First 20 lines
tail -n 20 file.txt                       # Last 20 lines
tail -f file.txt                          # Follow updates

# Extract
cut -d':' -f1 /etc/passwd                # Field extraction
cut -c1-10 file.txt                       # Character extraction
awk '{print $1, $3}' file                 # Column extraction
grep "pattern" file                       # Pattern matching
grep -E "pattern1|pattern2" file          # Multiple patterns

# Transform
tr 'a-z' 'A-Z' < file.txt                # Case conversion
sed 's/old/new/g' file.txt               # Text replacement
sed -i 's/old/new/g' file.txt            # In-place edit
sort file.txt                             # Sort lines
sort -u file.txt                          # Sort + unique
uniq -c                                   # Count occurrences

# Combine (Common Patterns)
cat file | grep "error" | wc -l                           # Count matches
cut -d':' -f1 /etc/passwd | sort                         # Extract + sort
awk '{print $1}' log | sort | uniq -c | sort -rn         # Frequency analysis
tail -f log | grep --line-buffered "ERROR"               # Live monitoring
sed 's/old/new/g' file | grep "pattern" | awk '{print}'  # Chain operations
```

---

**Master Linux Text Processing! 🚀**

*Last Updated: November 2025*
