# Azure Subscription

## What is an Azure Subscription?

An Azure Subscription is a **management, billing and resource boundary** used to deploy and manage Azure resources.

Azure resources such as VMs, AKS, Storage Accounts, VNets and Azure SQL are created inside a subscription.

### Simple way to remember

```text
Subscription = Resources + Billing + Governance Boundary
```

---

## Azure Structure

```text
Microsoft Entra Tenant
        ↓
   Subscription
        ↓
  Resource Group
        ↓
    Resources
```

A tenant can have multiple subscriptions.

Example:

```text
Microsoft Entra Tenant
        │
        ├── Dev Subscription
        │
        ├── UAT Subscription
        │
        └── Production Subscription
```

The exact subscription structure depends on the organization's architecture and governance model.

---

## What Can a Subscription Contain?

A subscription can contain multiple Resource Groups:

```text
Subscription
     │
     ├── rg-application-dev
     │      ├── VM
     │      ├── Storage
     │      └── Key Vault
     │
     ├── rg-application-uat
     │      ├── AKS
     │      ├── ACR
     │      └── VNet
     │
     └── rg-application-prod
            ├── AKS
            ├── SQL
            └── Application Gateway
```

---

## Subscription and Billing

Azure usage is tracked at the subscription level.

For example:

```text
Subscription
     │
     ├── Compute usage
     ├── Storage usage
     ├── Networking usage
     └── Database usage
              ↓
        Azure Cost Management
```

This allows organizations to monitor and manage Azure spending.

---

## Subscription and Access Control

Azure RBAC can be assigned at the subscription scope.

Example:

```text
Subscription
     │
     ├── Contributor
     ├── Reader
     └── Owner
```

Permissions assigned at the subscription level can affect resources within that scope, depending on the role and inheritance.

---

# Azure CLI Hands-on

## 1. Show Current Subscription

```powershell
az account show -o table
```

This helps verify which subscription is currently active.

---

## 2. List Available Subscriptions

```powershell
az account list -o table
```

Example output:

```text
Name                 CloudName    SubscriptionId      State
-------------------  -----------  ------------------  --------
Dev-Subscription     AzureCloud   <subscription-id>   Enabled
Production-Sub       AzureCloud   <subscription-id>   Enabled
```

---

## 3. Change the Active Subscription

If you have access to multiple subscriptions:

```powershell
az account set --subscription "<subscription-id>"
```

You can also use the subscription name:

```powershell
az account set --subscription "<subscription-name>"
```

Then verify:

```powershell
az account show -o table
```

---

# Real DevOps Scenario

Suppose I have access to multiple Azure subscriptions:

```text
Dev
UAT
Production
```

Before running Terraform or deployment commands, I verify the active subscription:

```powershell
az account show -o table
```

If the wrong subscription is selected, I switch it:

```powershell
az account set --subscription "<subscription-id>"
```

Then I verify it again before making any infrastructure changes.

This avoids accidentally deploying resources into the wrong subscription.

---

# Interview Question

### What is an Azure Subscription?

### Simple Interview Answer

> An Azure Subscription is a boundary where Azure resources are deployed and managed. It is also used for billing, governance and access control. For example, an organization can have separate subscriptions for development, UAT and production.

---

# Interview Scenario

### Question

**You have access to Dev and Production subscriptions. Before running Terraform, what will you check?**

### Answer

> First I check the active Azure subscription using `az account show`. If it is not the expected subscription, I switch it using `az account set`. Then I verify the subscription again before running Terraform or making any infrastructure changes.

---

## Key Takeaway

```text
Tenant
   ↓
Identity Boundary

Subscription
   ↓
Resource + Billing + Governance Boundary

Resource Group
   ↓
Logical Container

Resources
   ↓
Actual Azure Services
```

**Remember:**

> **Subscription = Where Azure resources are managed and billed**
