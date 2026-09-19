# Linux Files and Logs

## 1. `mkdir`

Creates a new directory.

```bash
mkdir devops-lab
```

Example:

```bash
mkdir backup
```

---

## 2. `touch`

Creates an empty file.

```bash
touch app.log
```

It can also update the timestamp of an existing file.

---

## 3. `cp`

Copies a file or directory.

Example:

```bash
cp app.log backup/
```

This copies `app.log` into the `backup` directory.

---

## 4. `mv`

Moves or renames a file or directory.

Rename:

```bash
mv app.log application.log
```

Move:

```bash
mv application.log backup/
```

---

## 5. `rm`

Removes a file.

```bash
rm app.log
```

**Important:** `rm` permanently removes the file, so use it carefully on production servers.

---

# Working With Application Logs

I created a realistic application log file:

```bash
touch app.log
```

Example log content:

```text
2026-09-18 10:00:01 INFO Application Started
2026-09-18 10:00:05 INFO Database connection successful
2026-09-18 10:01:12 INFO User login successful
2026-09-18 10:02:15 ERROR Database connection timeout
2026-09-18 10:02:20 ERROR Payment service unavailable
2026-09-18 10:03:10 INFO Application restarted
2026-09-18 10:03:15 INFO Database connection successful
2026-09-18 10:04:20 ERROR Payment service unavailable
```

---

# 6. `grep`

Used to search for specific text inside a file.

Find all errors:

```bash
grep "ERROR" app.log
```

Find payment-related messages:

```bash
grep "Payment" app.log
```

This is very useful when troubleshooting application issues from logs.

---

# 7. `head`

Shows the beginning of a file.

```bash
head app.log
```

Show only the first 3 lines:

```bash
head -3 app.log
```

---

# 8. `tail`

Shows the last lines of a file.

```bash
tail app.log
```

Show only the last 3 lines:

```bash
tail -3 app.log
```

---

# 9. `tail -f`

Continuously monitors a log file as new entries are added.

```bash
tail -f app.log
```

This is commonly used for real-time application troubleshooting.

Example:

```text
2026-09-18 10:05:10 INFO New request received
2026-09-18 10:05:12 ERROR Database connection timeout
```

Press `Ctrl + C` to stop monitoring.

---

# Real Production Troubleshooting Example

Suppose an application is running slowly and users are reporting payment failures.

I would first check the latest logs:

```bash
tail -f app.log
```

Then search specifically for errors:

```bash
grep "ERROR" app.log
```

If I find:

```text
ERROR Payment service unavailable
```

I would investigate the payment service and its dependencies further.

---

# Useful Log Commands

| Command                  | Purpose                   |
| ------------------------ | ------------------------- |
| `grep "ERROR" app.log`   | Find errors               |
| `grep "Payment" app.log` | Find payment-related logs |
| `head app.log`           | Show beginning of log     |
| `tail app.log`           | Show latest log entries   |
| `tail -f app.log`        | Monitor logs in real time |

---

# Interview Answer

### How do you troubleshoot an application using Linux logs?

> First, I check the latest logs using `tail` or `tail -f`. Then I use `grep` to search for specific errors or keywords. For example, if users report payment failures, I search for `Payment` or `ERROR` in the application log and then investigate the related service or dependency.
