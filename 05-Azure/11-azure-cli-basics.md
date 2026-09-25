# Azure CLI Basics

## 1. What is Azure CLI?

Azure CLI is a command-line tool used to create, manage, inspect and troubleshoot Azure resources.

As a DevOps engineer, CLI is useful because we can:

* Quickly check Azure resources
* Automate repetitive tasks
* Use commands inside CI/CD pipelines
* Write scripts
* Troubleshoot production issues
* Combine commands with `--query` and output formats

### Easy Memory Trick

```text
Azure CLI = Azure management from the terminal
```

---

# 2. Check Azure CLI Version

```bash
az version
```

This confirms that Azure CLI is installed and shows the installed version.

---

# 3. Login to Azure

```bash
az login
```

After login, check the current account:

```bash
az account show -o table
```

Example:

```text
Name
Subscription
Tenant
State
```

---

# 4. List Available Subscriptions

```bash
az account list -o table
```

This is useful when we have access to multiple subscriptions.

---

# 5. Select a Subscription

```bash
az account set --subscription <subscription-id>
```

Verify:

```bash
az account show -o table
```

### Production Scenario

Suppose I have Dev, UAT and Production subscriptions.

Before running a command, I first verify the active subscription.

```text
Login
  ↓
Check subscription
  ↓
Set correct subscription
  ↓
Run Azure command
```

This avoids accidentally running commands against the wrong subscription.

---

# 6. List Resource Groups

```bash
az group list -o table
```

Get one Resource Group:

```bash
az group show \
  --name <resource-group> \
  -o table
```

Get detailed information:

```bash
az group show \
  --name <resource-group> \
  -o json
```

---

# 7. Create a Resource Group

Create a test Resource Group:

```bash
az group create \
  --name <resource-group> \
  --location <location>
```

Example structure:

```text
Resource Group
      |
      +-- Location
      +-- Provisioning State
      +-- Resources
```

Check the Resource Group:

```bash
az group show \
  --name <resource-group> \
  -o table
```

---

# 8. List Azure Resources

List all resources in the subscription:

```bash
az resource list -o table
```

List resources inside a Resource Group:

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

This is one of the most useful commands during troubleshooting.

---

# 9. Get a Specific Resource

```bash
az resource show \
  --resource-group <resource-group> \
  --name <resource-name> \
  --resource-type <resource-type> \
  -o json
```

Example structure:

```text
Resource Group
      ↓
Resource Name
      ↓
Resource Type
      ↓
Resource Details
```

---

# 10. Get Resource ID

```bash
az resource show \
  --resource-group <resource-group> \
  --name <resource-name> \
  --resource-type <resource-type> \
  --query id \
  -o tsv
```

The Resource ID is useful when working with:

* RBAC
* Azure Monitor
* Managed identities
* Networking
* Terraform
* Resource dependencies

---

# 11. Azure CLI Output Formats

Azure CLI supports different output formats.

## Table

```bash
az resource list -o table
```

Best for human-readable output.

---

## JSON

```bash
az resource list -o json
```

Useful when we need detailed structured information.

---

## TSV

```bash
az resource list -o tsv
```

Useful when we need clean output for scripts.

---

## YAML

```bash
az resource list -o yaml
```

Useful when working with YAML-based workflows.

---

# 12. Using `--query`

The `--query` option allows us to extract only the information we need.

For example:

```bash
az resource list \
  --query "[].name" \
  -o table
```

Get resource name and location:

```bash
az resource list \
  --query "[].{Name:name,Location:location}" \
  -o table
```

Get resource name and type:

```bash
az resource list \
  --query "[].{Name:name,Type:type}" \
  -o table
```

### Why `--query` is Useful

Azure may return a large JSON response.

Instead of reading everything:

```text
Large JSON
    ↓
--query
    ↓
Only required information
```

---

# 13. Filter Resources

Find resources of a particular type:

```bash
az resource list \
  --query "[?type=='Microsoft.Compute/virtualMachines'].{Name:name,Location:location}" \
  -o table
```

Find resources in a specific location:

```bash
az resource list \
  --query "[?location=='<location>'].{Name:name,Type:type}" \
  -o table
```

This is useful when troubleshooting large environments.

---

# 14. Create a Storage Account

For hands-on practice, create a small test Storage Account.

Storage account names must be globally unique and use lowercase letters/numbers.

```bash
az storage account create \
  --name <storage-account> \
  --resource-group <resource-group> \
  --location <location> \
  --sku Standard_LRS
```

Check it:

```bash
az storage account show \
  --name <storage-account> \
  --resource-group <resource-group> \
  -o table
```

Get only the provisioning state:

```bash
az storage account show \
  --name <storage-account> \
  --resource-group <resource-group> \
  --query provisioningState \
  -o tsv
```

Expected:

```text
Succeeded
```

---

# 15. List Storage Accounts

```bash
az storage account list -o table
```

List only Storage Accounts from a Resource Group:

```bash
az storage account list \
  --resource-group <resource-group> \
  -o table
```

---

# 16. Delete a Test Resource

If a resource was created only for practice, remove it after the lab.

Delete the Storage Account:

```bash
az storage account delete \
  --name <storage-account> \
  --resource-group <resource-group> \
  --yes
```

Or remove the entire test Resource Group:

```bash
az group delete \
  --name <resource-group> \
  --yes
```

---

# 17. Azure CLI Help

If you don't remember a command, use help.

Example:

```bash
az group --help
```

Storage help:

```bash
az storage --help
```

Resource help:

```bash
az resource --help
```

Specific command:

```bash
az group create --help
```

### Memory Trick

```text
Don't remember command?
        ↓
az <command> --help
```

---

# 18. Common CLI Commands to Remember

| Requirement          | Command                        |
| -------------------- | ------------------------------ |
| Login                | `az login`                     |
| Current subscription | `az account show`              |
| List subscriptions   | `az account list`              |
| Change subscription  | `az account set`               |
| List RGs             | `az group list`                |
| Show RG              | `az group show`                |
| Create RG            | `az group create`              |
| List resources       | `az resource list`             |
| Show resource        | `az resource show`             |
| Get Resource ID      | `az resource show --query id`  |
| Get help             | `az <command> --help`          |
| Activity logs        | `az monitor activity-log list` |

---

# 19. Real Production Troubleshooting Flow

Suppose someone reports:

> "The application is down."

I can start with:

```bash
az account show -o table
```

First, confirm the correct subscription.

Then:

```bash
az group show --name <resource-group> -o table
```

Check the Resource Group.

Then:

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

Check whether the required resources exist.

Then inspect the specific resource:

```bash
az resource show \
  --ids <resource-id> \
  -o json
```

Check recent Activity Logs:

```bash
az monitor activity-log list \
  --resource-group <resource-group> \
  --max-events 20 \
  -o table
```

Then continue into the specific platform:

```text
Azure Resource
      ↓
Networking
      ↓
VM / App Service / AKS
      ↓
Application
      ↓
Logs
```

---

# 20. Mini CLI Challenge

Use a test Resource Group that you manage.

### Task 1

List all Resource Groups.

```bash
az group list -o table
```

### Task 2

Create a test Resource Group.

```bash
az group create \
  --name <resource-group> \
  --location <location>
```

### Task 3

Verify its provisioning state.

```bash
az group show \
  --name <resource-group> \
  --query properties.provisioningState \
  -o tsv
```

Expected:

```text
Succeeded
```

### Task 4

Create a test Storage Account.

```bash
az storage account create \
  --name <storage-account> \
  --resource-group <resource-group> \
  --location <location> \
  --sku Standard_LRS
```

### Task 5

Get only the Storage Account name and location.

```bash
az storage account show \
  --name <storage-account> \
  --resource-group <resource-group> \
  --query "{Name:name,Location:location}" \
  -o table
```

### Task 6

Get the Resource ID.

```bash
az storage account show \
  --name <storage-account> \
  --resource-group <resource-group> \
  --query id \
  -o tsv
```

### Task 7

Delete the test resources after completing the lab.

```bash
az group delete \
  --name <resource-group> \
  --yes
```

---

# 21. Interview Answer

**Question: How do you use Azure CLI in your DevOps work?**

> "I mainly use Azure CLI for resource management, troubleshooting and automation. For example, I use it to check subscriptions, resource groups, resource states and activity logs. I also use `--query` to extract only the required information and use these commands inside scripts or CI/CD pipelines when needed."

---

# 22. Key Takeaways

```text
az login
    ↓
az account show
    ↓
az group list
    ↓
az resource list
    ↓
az resource show
    ↓
--query
    ↓
-o table / json / tsv
```

### Remember

```text
Azure CLI = Manage + Inspect + Troubleshoot + Automate
```
