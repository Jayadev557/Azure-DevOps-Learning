# Azure Resource Group

## What is a Resource Group?

A Resource Group is a **logical container** used to organize and manage related Azure resources.

For example, an application environment may have:

```text
Resource Group
      │
      ├── Virtual Network
      ├── Virtual Machine
      ├── Storage Account
      ├── Key Vault
      └── Network Security Group
```

### Simple way to remember

```text
Resource Group = Logical Container for Azure Resources
```

---

## Why Do We Use Resource Groups?

Resource Groups help organize resources based on:

* Application
* Environment
* Project
* Team
* Lifecycle

Example:

```text
rg-payment-dev
rg-payment-uat
rg-payment-prod
```

Each Resource Group can contain resources belonging to that environment.

---

## Resource Group Structure

```text
Azure Subscription
       │
       ├── rg-app-dev
       │      ├── VM
       │      ├── VNet
       │      └── Storage
       │
       ├── rg-app-uat
       │      ├── AKS
       │      ├── ACR
       │      └── Key Vault
       │
       └── rg-app-prod
              ├── AKS
              ├── SQL
              └── Application Gateway
```

---

# Azure CLI Hands-on

## 1. List Resource Groups

```powershell
az group list -o table
```

This shows the Resource Groups available in the active subscription.

---

## 2. Show a Specific Resource Group

```powershell
az group show --name "<resource-group>"
```

Example:

```powershell
az group show --name "rg-payment-prod"
```

---

## 3. Show Resource Group Location

```powershell
az group show \
  --name "<resource-group>" \
  --query location \
  -o tsv
```

In PowerShell, you can also use:

```powershell
az group show --name "<resource-group>" --query location -o tsv
```

---

## 4. List Resources Inside a Resource Group

```powershell
az resource list --resource-group "<resource-group>" -o table
```

Example:

```powershell
az resource list --resource-group "rg-payment-prod" -o table
```

This is useful when you want to understand what resources belong to a particular application or environment.

---

# Real DevOps Scenario

Suppose I join a project and I'm given:

```text
rg-payment-prod
```

Before making any change, I check the resources inside it:

```powershell
az resource list --resource-group "rg-payment-prod" -o table
```

I may find:

```text
AKS
ACR
VNet
Key Vault
Load Balancer
Storage
```

Then I understand how the application infrastructure is organized before touching anything.

---

# Resource Group and Resource Lifecycle

Resources inside a Resource Group can be managed together.

For example:

```text
rg-payment-dev
       │
       ├── VM
       ├── Storage
       └── VNet
```

If the entire development environment is no longer required, the Resource Group can be removed as part of an intentional cleanup process.

This can affect the resources contained in that Resource Group, so it should never be done without confirming the scope and dependencies.

---

# Resource Group vs Subscription

These are different levels.

```text
Tenant
   ↓
Subscription
   ↓
Resource Group
   ↓
Resources
```

### Subscription

Used for:

* Resource management
* Billing
* Governance
* Access boundaries

### Resource Group

Used for:

* Logical organization
* Resource grouping
* Lifecycle management
* Access management at resource-group scope

---

# Interview Question

### What is a Resource Group?

### Simple Interview Answer

> A Resource Group is a logical container for related Azure resources. I normally organize resources based on application and environment, such as `rg-payment-prod`, so that access, governance and lifecycle management become easier.

---

# Interview Scenario

### Question

**You are given an existing production Resource Group. What will you check first?**

### Answer

> First I list the resources inside the Resource Group and understand what services are deployed. Then I check their dependencies, access, tags and existing Terraform configuration before making any changes.

---

## Key Takeaway

```text
Subscription
      ↓
Resource Group
      ↓
Related Azure Resources
```

**Remember:**

> **Resource Group = Logical container for related Azure resources**
