# Linux Users and Groups

## 1. Check Current User

`whoami` shows the current logged-in user.

```bash
whoami
```

Example output:

```text
jd
```

---

## 2. Check User Details

`id` shows the user's UID, GID, and group memberships.

```bash
id
```

Example:

```text
uid=1000(jd) gid=1000(jd) groups=1000(jd),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),101(lxd)
```

### Important

* **UID** → User ID
* **GID** → Primary Group ID
* **Groups** → Additional groups the user belongs to

---

## 3. Check Groups

The `groups` command shows the groups associated with the current user.

```bash
groups
```

Example:

```text
jd adm cdrom sudo dip plugdev lxd
```

---

# Root and Sudo

## 4. Check Root Access

I used `sudo` to execute a command with elevated privileges.

```bash
sudo whoami
```

Output:

```text
root
```

`sudo` allows an authorized user to execute specific commands with elevated privileges.

---

# Creating Users

## 5. Create a User

I created a test application user named `appuser`.

```bash
sudo useradd appuser
```

Check the user:

```bash
id appuser
```

Example:

```text
uid=1001(appuser) gid=1001(appuser) groups=1001(appuser)
```

---

# `/etc/passwd`

Linux stores basic user account information in:

```text
/etc/passwd
```

Check a specific user:

```bash
grep appuser /etc/passwd
```

Example:

```text
appuser:x:1001:1001::/home/appuser:/bin/sh
```

The important fields are:

```text
username : password-placeholder : UID : GID : comment : home-directory : shell
```

The `x` means the password hash is stored separately, normally in `/etc/shadow`.

---

# Creating Groups

## 6. Create a Group

I created a group called `devops`.

```bash
sudo groupadd devops
```

---

## 7. Add User to Group

I added `appuser` to the `devops` group.

```bash
sudo usermod -aG devops appuser
```

Verify:

```bash
id appuser
```

Example:

```text
uid=1001(appuser) gid=1001(appuser) groups=1001(appuser),1002(devops)
```

### Why `-aG`?

* `-G` → Modify supplementary groups
* `-a` → Append the new group without removing existing supplementary groups

So:

```bash
usermod -aG devops appuser
```

means **add `appuser` to the `devops` group while keeping existing group memberships**.

---

# Why Groups Are Important in DevOps

Groups are commonly used to control access to shared resources.

For example:

```text
                    devops group
                         |
              +----------+----------+
              |                     |
           appuser               developer
              |                     |
              +----------+----------+
                         |
                    Shared files
```

Instead of giving permissions individually to every user, we can manage access through a group.

---

# Real Production Example

Suppose multiple DevOps engineers need access to application logs.

Instead of changing permissions for every user separately, I can:

1. Create a group.
2. Add required users to the group.
3. Make the group the owner of the shared directory.
4. Give the group the required permissions.

Example:

```bash
sudo groupadd devops
sudo usermod -aG devops appuser
sudo chown root:devops /opt/myapp/logs
sudo chmod 775 /opt/myapp/logs
```

This provides controlled shared access.

---

# Interview Answer

### What is the difference between UID, GID and groups?

> UID identifies a user, while GID identifies a group. A user has a primary group and can also belong to additional groups. In production, I usually use groups to provide controlled access to shared files and directories instead of giving permissions individually to every user.
