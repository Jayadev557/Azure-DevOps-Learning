# Day 1 – Linux Fundamentals

## Objective

Build a strong Linux foundation for DevOps and production troubleshooting.

---

## Topics Covered

### 1. Basic Linux Commands

Learned:

```bash
pwd
ls
ls -a
ls -l
ls -la
whoami
uname -a
```

Covered:

* Current working directory
* Files and directories
* Hidden files
* File details
* Current user
* Linux system information
* Linux path symbols

---

### 2. Files and Application Logs

Learned:

```bash
mkdir
touch
cp
mv
rm
grep
head
tail
tail -f
```

Practiced:

* Creating files and directories
* Copying and moving files
* Removing files
* Searching application logs
* Checking latest log entries
* Monitoring logs in real time

---

### 3. Linux Users and Groups

Learned:

```bash
whoami
id
groups
sudo
useradd
groupadd
usermod -aG
```

Practiced:

* Creating `appuser`
* Creating `devops` group
* Adding `appuser` to the `devops` group
* Understanding UID and GID
* Checking `/etc/passwd`
* Understanding sudo/root access

---

### 4. Linux Permissions

Learned:

```bash
chmod
chown
ls -l
ls -ld
```

Covered:

* Read, write and execute permissions
* Numeric permissions
* `600`
* `644`
* `755`
* Why `777` should be avoided in production
* File vs directory permissions
* Ownership vs permissions

Key memory:

```text
chmod → What can they do?
chown → Who owns it?
```

---

### 5. Production Permission Troubleshooting

Built a real-world scenario where:

```text
Application user → appuser
Log owner        → root
Result           → Permission denied
```

Troubleshooting flow:

```text
Identify application user
        ↓
Check file owner/group
        ↓
Check file permissions
        ↓
Check parent directory permissions
        ↓
Fix ownership/permissions
        ↓
Retest as application user
```

Used:

```bash
sudo -u appuser
chown
chmod
ls -l
ls -ld
```

---

## Important DevOps Learnings

### Log Troubleshooting

```bash
tail -f app.log
grep "ERROR" app.log
```

### Permission Troubleshooting

```bash
whoami
id
ls -l app.log
ls -ld /path/to/directory
```

### Ownership

```bash
sudo chown appuser:appuser app.log
```

### Permission

```bash
chmod 644 app.log
```

---

## Day-1 Interview Questions

### 1. What commands do you use for basic Linux troubleshooting?

> I commonly use `pwd`, `ls -la`, `whoami`, `id`, `tail`, and `grep`. I use them to check the current location, files, users, logs, and application errors.

### 2. How do you troubleshoot `Permission denied`?

> First, I check which user the application is running as. Then I check file and directory ownership and permissions. Based on the issue, I use `chown` or `chmod` and finally retest as the application user.

### 3. What is the difference between `chmod` and `chown`?

> `chmod` changes permissions, while `chown` changes ownership. I check both when troubleshooting Linux permission issues.

### 4. Why do you avoid `chmod 777` in production?

> Because it gives read, write, and execute access to everyone. Instead, I give only the permissions required by the application or user.

---

## Day-1 Status

```text
Linux Basics              ✅
Files & Logs              ✅
Users & Groups            ✅
Permissions               ✅
Production Lab            ✅
Interview Practice        ✅
Documentation             ✅
```
Conflict resolution
```
