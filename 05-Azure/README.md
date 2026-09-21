# Day 1 — Azure Fundamentals

## Objective

Understand the basic Azure structure and learn how to inspect an existing Azure environment using Azure CLI.

> **Security Note:**
> Real tenant IDs, subscription IDs, email addresses, resource IDs, credentials, tokens and other sensitive information are intentionally excluded from this documentation.

---

## Azure Core Hierarchy

```text
Microsoft Entra Tenant
        ↓
   Subscription
        ↓
  Resource Group
        ↓
    Resources
```

### Easy Memory Trick

* **Tenant** → Identity boundary
* **Subscription** → Resource + billing boundary
* **Resource Group** → Logical resource container
* **Region** → Geographic Azure location
* **Availability Zone** → Separate infrastructure location within a supported region
* **Resource** → Actual Azure service
* **Resource Provider** → Namespace that manages a service
* **Resource ID** → Unique identifier of a resource
* **Tags** → Resource metadata

---

# Topics Covered

| Topic                     | Type              | Status    |
| ------------------------- | ----------------- | --------- |
| Azure Tenant              | Read + Understand | Completed |
| Azure Subscription        | Read + Hands-on   | Completed |
| Resource Group            | Read + Hands-on   | Completed |
| Azure Region              | Read + Hands-on   | Completed |
| Availability Zones        | Read              | Completed |
| Azure Resources           | Read + Hands-on   | Completed |
| Resource Providers        | Read              | Completed |
| Azure Resource ID         | Read              | Completed |
| Azure Tags                | Read + Inspection | Completed |
| Portal / CLI / PowerShell | Read + Hands-on   | Completed |
| Azure CLI Basics          | Hands-on          | Completed |
| Real Project Structure    | Read + Design     | Completed |
| Scenario Questions        | Practice          | Completed |
| Mini Lab                  | Hands-on          | Completed |

---

# Practical Azure CLI Commands

## 1. Check Current Azure Context

```powershell
az account show
```

### Purpose

Used to verify the currently active Azure subscription and login context.

> Do not commit the raw output of this command because it can contain account-specific information.

---

## 2. List Resource Groups

```powershell
az group list -o table
```

### Purpose

Used to identify existing Resource Groups and their locations.

---

## 3. List Azure Resources

```powershell
az resource list -o table
```

### Purpose

Used to perform a basic inventory of resources deployed in the current subscription.

---

## 4. Inspect Resource Type, Resource Group and Location

```powershell
az resource list --query "[].{Name:name,Type:type,RG:resourceGroup,Location:location}" -o table
```

This helps identify:

```text
Resource Name
      ↓
Resource Type
      ↓
Resource Group
      ↓
Azure Region
```

---

## 5. Inspect Resource Tags

```powershell
az resource list --query "[].{Name:name,ResourceGroup:resourceGroup,Tags:tags}" -o table
```

Example tag structure:

```text
Environment = Dev
Application = MyApplication
Owner       = DevOps
CostCenter  = CC1001
```

---

# Real Azure Environment Assessment

When joining an existing Azure project, I would follow this basic flow:

```text
Check Subscription
        ↓
Check Resource Groups
        ↓
Inventory Resources
        ↓
Identify Azure Services
        ↓
Check Regions
        ↓
Check Tags / Ownership
        ↓
Understand Dependencies
        ↓
Check RBAC / Access
        ↓
Review Terraform / IaC
        ↓
Review CI/CD
        ↓
Review Monitoring
        ↓
Make Changes
```

The important point is:

> **Understand the existing environment before making changes.**

---

# Example Azure Architecture

A typical application environment may contain:

```text
                    Azure Subscription
                           │
                           ▼
                    Resource Group
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
       AKS                ACR              Storage
        │
        ├── VNet
        ├── NSG
        ├── Load Balancer
        ├── Managed Identity
        └── VM Scale Set
```

Other application components may include:

```text
Azure SQL
Azure Key Vault
Application Gateway
Azure Monitor
Log Analytics
```

---

# DevOps Safety Mindset

Before changing an existing Azure environment:

* Check the active subscription.
* Identify the Resource Group.
* Understand the resource and its purpose.
* Check dependencies.
* Check ownership.
* Check RBAC/access.
* Review Terraform or other IaC.
* Review CI/CD pipelines.
* Check monitoring and alerts.
* Understand rollback/recovery options.
* Never delete an unfamiliar resource without verification.

---

# Security Rules for GitHub

Never commit the following to a public repository:

```text
Tenant ID
Subscription ID
Client ID
Client Secret
Passwords
Access Tokens
Private Keys
Connection Strings
Personal Email Addresses
Production Credentials
Sensitive Resource IDs
Internal Network Details
```

Use placeholders instead:

```text
<tenant-id>
<subscription-id>
<resource-group>
<resource-name>
<storage-account>
<client-id>
```

---

# Day 1 Interview Scenario

### Question

**You joined an existing Azure project. How would you understand the environment before making changes?**

### Answer

> First I check the active subscription and resource groups. Then I inventory the resources and identify services like AKS, ACR, VNet, Storage, SQL and VMs. After that I check dependencies, access, tags and existing Terraform or deployment configuration before making any changes.

---

# Day 1 Completion

Day 1 covered:

* Azure Tenant
* Azure Subscription
* Resource Groups
* Azure Regions
* Availability Zones
* Azure Resources
* Resource Providers
* Resource IDs
* Azure Tags
* Azure CLI
* Portal / CLI / PowerShell
* Azure environment assessment
* Scenario-based interview questions
* Security practices


**Day 2 — Azure Networking & Compute Fundamentals**
