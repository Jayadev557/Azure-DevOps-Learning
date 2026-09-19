# Linux Permissions

Linux permissions control **who can read, write, or execute** a file or directory.

---

# 1. Understanding `ls -l`

Example:

```text
-rw-r--r-- 1 jd jd 20 Sep 18 16:58 app.log
```

The permission section is:

```text
-rw-r--r--
```

It can be understood as:

```text
-   rw-   r--   r--
    |     |     |
   User  Group Others
```

### Meaning

* `r` → Read
* `w` → Write
* `x` → Execute
* `-` → Permission not granted

---

# 2. Numeric Permission Values

Linux uses these numeric values:

| Permission    | Value |
| ------------- | ----: |
| Read (`r`)    |     4 |
| Write (`w`)   |     2 |
| Execute (`x`) |     1 |

Examples:

```text
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2     = 6
r-x = 4 + 1     = 5
r-- = 4         = 4
```

---

# 3. Common Permission Modes

## `644`

```bash
chmod 644 app.log
```

Meaning:

```text
Owner  → read + write
Group  → read
Others → read
```

Permission:

```text
-rw-r--r--
```

This is commonly used for normal files.

---

## `755`

```bash
chmod 755 script.sh
```

Meaning:

```text
Owner  → read + write + execute
Group  → read + execute
Others → read + execute
```

Permission:

```text
-rwxr-xr-x
```

Commonly used for executable scripts and directories.

---

## `600`

```bash
chmod 600 secret.txt
```

Meaning:

```text
Owner  → read + write
Group  → no access
Others → no access
```

Permission:

```text
-rw-------
```

Useful for sensitive files that should only be accessible by the owner.

---

## `777`

```bash
chmod 777 file
```

Gives read, write and execute permissions to everyone.

```text
Owner  → rwx
Group  → rwx
Others → rwx
```

I avoid using `777` casually in production because it gives excessive access.

---

# 4. `chmod`

`chmod` changes the permission mode of a file or directory.

Example:

```bash
chmod 444 app.log
```

Now the file becomes read-only:

```text
-r--r--r--
```

When I tried:

```bash
echo "New log" >> app.log
```

I received:

```text
Permission denied
```

because the file no longer had write permission.

I restored the original permission:

```bash
chmod 644 app.log
```

---

# 5. `chown`

`chown` changes the **owner and/or group** of a file or directory.

Example:

```bash
sudo chown appuser:appuser app.log
```

This changes:

```text
Owner → appuser
Group → appuser
```

### Important Difference

```text
chmod → changes permissions
chown → changes ownership
```

A common troubleshooting mistake is confusing these two.

---

# 6. File vs Directory Permissions

Permissions behave slightly differently for files and directories.

### File

```text
r → read file
w → modify file
x → execute file
```

### Directory

```text
r → list directory contents
w → create/delete files inside directory
x → access/traverse the directory
```

For example, a user may have write permission on a file but still be unable to create a new file in the directory if the directory itself does not provide the required permission.

---

# 7. Check Directory Permissions

Use:

```bash
ls -ld /path/to/directory
```

Example:

```bash
ls -ld ~/devops-lab
```

Output:

```text
drwxrwxr-x ... /home/jd/devops-lab
```

The `d` at the beginning means it is a directory.

---

# 8. Permission Troubleshooting Flow

When I get a `Permission denied` error, I check:

```text
1. Which user is running the application?
           ↓
2. Who owns the file/directory?
           ↓
3. What permissions are configured?
           ↓
4. What permissions exist on the parent directory?
           ↓
5. Fix only the required ownership/permission
           ↓
6. Test again as the application user
```

Useful commands:

```bash
whoami
id
ls -l file
ls -ld directory
```

---

# Key Memory Trick

```text
chmod → What can they do?
chown → Who owns it?
```

---

# Interview Answer

### What is the difference between `chmod` and `chown`?

> `chmod` is used to change the permissions of a file or directory, like read, write, and execute. `chown` is used to change the owner or group. When troubleshooting permission issues, I first check ownership and permissions, then change only what is required.
