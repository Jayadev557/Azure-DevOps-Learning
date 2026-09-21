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

# Resource Group Limitations

Resource Groups have some important limitations and design considerations.

## 1. A Resource Belongs to One Resource Group

An Azure resource can belong to **only one Resource Group at a time**.

Example:

```text
rg-payment-prod
      │
      └── payment-vm
```

The same VM cannot simultaneously belong to another Resource Group.

A resource can sometimes be moved to another Resource Group if the resource type and circumstances support the move.

---

## 2. Resource Groups Are Subscription-Specific

A Resource Group belongs to a specific Azure Subscription.

```text
Subscription A
      │
      └── rg-payment-prod
```

You cannot have one Resource Group simultaneously belong to two subscriptions.

Resources can sometimes be moved between subscriptions when the resource type and required conditions support the move.

---

## 3. Resource Group Has a Location

A Resource Group has a location, sometimes called its **region**.

However, the Resource Group's location does **not automatically determine the location of every resource inside it**.

Example:

```text
Resource Group
Location → East US

Resources
   ├── VM → East US
   ├── Storage → West Europe
   └── Another resource → Central India
```

The actual deployment locations depend on the individual resource and service capabilities.

---

## 4. Resource Group Deletion Can Affect Resources

Deleting a Resource Group can delete the resources contained within it.

Example:

```text
rg-payment-dev
      │
      ├── VM
      ├── Storage
      └── VNet

Delete Resource Group
      ↓
Resources can also be deleted
```

Therefore, Resource Group deletion should be treated as a significant operation.

---

## 5. Not Every Azure Resource Can Be Moved

Some Azure resources support moving between Resource Groups or subscriptions, while others have restrictions.

Before moving a resource, check whether:

* The resource type supports movement.
* Dependent resources also support the move.
* The target subscription supports the resource.
* There are service-specific restrictions.

In production, resource movement should be planned carefully.

---

## 6. Resource Groups Are Not Strictly Deployment Environments

A Resource Group is a logical management boundary.

It does not automatically mean:

```text
rg-dev     = Development
rg-prod    = Production
```

Organizations often use this naming pattern, but the actual environment design is a choice made by the architecture and governance model.

---

## 7. Resources Can Have Dependencies Across Resource Groups

Related resources do not always have to be in the same Resource Group.

Example:

```text
rg-application
      │
      └── Application

rg-network
      │
      └── VNet

rg-monitoring
      │
      └── Log Analytics
```

The resources can still communicate or depend on each other.

Therefore, don't assume:

> "If resources are in different Resource Groups, they cannot be connected."

---

# Real DevOps Scenario

Suppose I join a project and I'm given:

```text
rg-payment-prod
```

Before making any change, I check:

```powershell
az resource list --resource-group "rg-payment-prod" -o table
```

Then I identify:

```text
AKS
ACR
VNet
Key Vault
Storage
```

I also check whether some of these resources have dependencies on resources in other Resource Groups.

Only after understanding the architecture do I plan the change.

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
* Access management at Resource Group scope

---

# Interview Question

### What is a Resource Group?

### Simple Interview Answer

> A Resource Group is a logical container for related Azure resources. I normally organize resources based on application and environment, such as `rg-payment-prod`, so that access, governance and lifecycle management become easier.

---

# Interview Question

### What are the limitations of a Resource Group?

### Simple Interview Answer

> A resource can belong to only one Resource Group at a time, and a Resource Group belongs to one subscription. Resource Groups have their own location, but resources inside them can use different supported regions. Also, deleting a Resource Group can affect the resources inside it, and not every resource supports moving between Resource Groups or subscriptions.

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

### Remember

> **Resource Group = Logical container for related Azure resources**

### Important Limitations

```text
One resource
      ↓
One Resource Group at a time

One Resource Group
      ↓
One Subscription

Resource Group Location
      ↓
Does not force all resources to the same region

Delete Resource Group
      ↓
Can affect resources inside it

Resource Move
      ↓
Depends on service-specific support
```
