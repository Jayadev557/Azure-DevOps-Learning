# Production Permission Troubleshooting Lab

## Scenario

Suppose an application is running as a dedicated Linux user called `appuser`.

The application needs to write logs under:

```text
/opt/myapp/app.log
```

But the application is getting:

```text
Permission denied
```

I will troubleshoot this like a real production issue.

---

# 1. Create Application Directory

```bash
sudo mkdir -p /opt/myapp
```

Create the log file:

```bash
sudo touch /opt/myapp/app.log
```

Check ownership and permissions:

```bash
ls -l /opt/myapp/app.log
```

Example:

```text
-rw-r--r-- 1 root root ... app.log
```

The file is owned by:

```text
Owner → root
Group → root
```

---

# 2. Simulate Application Writing to the Log

The application is running as `appuser`.

I simulate this using:

```bash
sudo -u appuser bash -c 'echo "Application started" >> /opt/myapp/app.log'
```

Result:

```text
Permission denied
```

---

# 3. Identify the Root Cause

First, I check the file:

```bash
ls -l /opt/myapp/app.log
```

Example:

```text
-rw-r--r-- 1 root root ... app.log
```

The application is running as:

```text
appuser
```

But the file belongs to:

```text
root:root
```

The `appuser` does not have write permission on the file.

So the problem is incorrect ownership/permissions.

---

# 4. Fix the Ownership

Since this application should own its log file, I change the ownership:

```bash
sudo chown appuser:appuser /opt/myapp/app.log
```

Check again:

```bash
ls -l /opt/myapp/app.log
```

Now:

```text
-rw-r--r-- 1 appuser appuser ... app.log
```

---

# 5. Test Again

Run the application simulation again:

```bash
sudo -u appuser bash -c 'echo "Application started" >> /opt/myapp/app.log'
```

This time the command succeeds.

Check the log:

```bash
cat /opt/myapp/app.log
```

The application can now write successfully.

---

# Second Scenario: Directory Permission

Now create a logs directory:

```bash
sudo mkdir -p /opt/myapp/logs
```

Create a log file:

```bash
sudo touch /opt/myapp/logs/app.log
```

Give ownership of the file to `appuser`:

```bash
sudo chown appuser:appuser /opt/myapp/logs/app.log
```

Check the directory:

```bash
ls -ld /opt/myapp/logs
```

Example:

```text
drwxr-xr-x 2 root root ... /opt/myapp/logs
```

---

# 6. Try Creating a New File

Run:

```bash
sudo -u appuser touch /opt/myapp/logs/test.log
```

Result:

```text
Permission denied
```

Why?

The existing `app.log` belongs to `appuser`, but the **directory itself** belongs to `root`.

The directory does not give `appuser` the required write permission.

---

# 7. Fix Directory Ownership

Change the directory ownership:

```bash
sudo chown appuser:appuser /opt/myapp/logs
```

Now test again:

```bash
sudo -u appuser touch /opt/myapp/logs/test.log
```

This time it succeeds.

Check:

```bash
ls -l /opt/myapp/logs
```

Example:

```text
-rw-r--r-- 1 appuser appuser ... app.log
-rw-rw-r-- 1 appuser appuser ... test.log
```

---

# Important Learning

A file can have the correct ownership but the application can still get `Permission denied` if the **parent directory permissions** are incorrect.

So during permission troubleshooting, I check both:

```text
File permissions
        +
Directory permissions
```

---

# Production Troubleshooting Flow

When an application gets `Permission denied`, I follow this flow:

```text
Application
    |
    ↓
Which user is running it?
    |
    ↓
Check file owner/group
    |
    ↓
Check file permissions
    |
    ↓
Check parent directory permissions
    |
    ↓
Fix ownership or permissions
    |
    ↓
Retest as application user
```

Useful commands:

```bash
whoami
id
ls -l /path/to/file
ls -ld /path/to/directory
```

---

# Least Privilege

I should not blindly use:

```bash
chmod 777
```

just to solve a permission issue.

Instead, I give the application only the access it actually needs.

For example:

```text
Application user → appuser
Log ownership    → appuser:appuser
Required access  → write to application log directory
```

This follows the **least-privilege** principle.

---

# Interview Answer

### How would you troubleshoot a `Permission denied` issue in production?

> First, I check which user the application is running as. Then I check the file and directory ownership and permissions using `ls -l` and `ls -ld`. If the ownership is incorrect, I use `chown`; if the permission mode is incorrect, I use `chmod`. Finally, I test again as the application user instead of directly using root.

---

# Real Example From This Lab

The application was running as `appuser`, but the log file was owned by `root`.

I checked the user, ownership, and permissions and found that `appuser` did not have write access.

I fixed it using:

```bash
sudo chown appuser:appuser /opt/myapp/app.log
```

After that, I tested again as `appuser`, and the application was able to write to the log successfully.

---

# Key Takeaways

```text
chmod → Change permissions
chown → Change ownership

File permissions → Control access to the file
Directory permissions → Control access inside the directory

root → Administrative user
appuser → Dedicated application user

Best practice → Give only the required permissions
```
