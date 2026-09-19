# Linux Basic Commands

## 1. `pwd`

`pwd` stands for **Print Working Directory**.

It shows the current directory where I am working.

```bash
pwd
```

Example output:

```text
/home/jd
```

---

## 2. `ls`

`ls` lists files and directories in the current directory.

```bash
ls
```

---

## 3. `ls -a`

Shows all files and directories, including hidden files.

```bash
ls -a
```

Hidden Linux files usually start with `.`.

Examples:

```text
.bashrc
.ssh
.profile
```

---

## 4. `ls -l`

Shows files in long format with details such as:

* Permissions
* Owner
* Group
* File size
* Date/time
* File name

```bash
ls -l
```

Example:

```text
-rw-r--r-- 1 jd jd 20 Sep 18 16:58 app.log
```

---

## 5. `ls -la`

Shows hidden files with detailed information.

```bash
ls -la
```

This is commonly useful when troubleshooting hidden configuration files and permissions.

---

## 6. `whoami`

Shows the currently logged-in user.

```bash
whoami
```

Example:

```text
jd
```

---

## 7. `uname -a`

Displays Linux system and kernel information.

```bash
uname -a
```

Useful when checking the operating system, kernel version, and system architecture.

---

# Important Linux Path Symbols

| Symbol | Meaning                       |
| ------ | ----------------------------- |
| `.`    | Current directory             |
| `..`   | Parent directory              |
| `~`    | Current user's home directory |
| `/`    | Root of the filesystem        |

Example:

```bash
cd ~/devops-lab
```

For user `jd`, this means:

```text
/home/jd/devops-lab
```

---

# Practical Example

During troubleshooting, I can use:

```bash
pwd
ls -la
whoami
uname -a
```

This quickly tells me:

1. Where I am
2. What files exist
3. Which user I am using
4. Basic system information

---

# Interview Answer

### What Linux commands do you commonly use for basic troubleshooting?

> I commonly use `pwd` to check my current directory, `ls -la` to check files and permissions, `whoami` to identify the current user, and `uname -a` to check system and kernel information. These are usually my first commands when I start troubleshooting a Linux server.
