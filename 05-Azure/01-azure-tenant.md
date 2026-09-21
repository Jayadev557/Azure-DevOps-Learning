# Azure Tenant

## What is an Azure Tenant?

An Azure Tenant is the **identity boundary** for an organization in Microsoft Entra ID.

It contains:

* Users
* Groups
* Applications
* Service Principals
* Managed identities
* Authentication and identity-related configuration

### Simple way to remember

```text
Tenant = Identity Boundary
```

---

## Real Project Example

Imagine a company has an Azure environment for multiple applications.

The organization has one Microsoft Entra Tenant.

Inside that tenant, users and applications are managed:

```text
Microsoft Entra Tenant
        │
        ├── Users
        ├── Groups
        ├── Applications
        ├── Service Principals
        └── Managed Identities
```

The tenant provides the identity foundation used to authenticate users and applications.

---

## Tenant vs Subscription

These two are commonly confused.

```text
Tenant
  ↓
Identity / Authentication
  ↓
Users, Groups, Applications

Subscription
  ↓
Azure Resources / Billing
  ↓
VM, AKS, Storage, VNet, SQL
```

### Easy Memory Trick

> **Tenant = Who you are**

> **Subscription = Where you deploy Azure resources**

---

## Azure CLI

To see the current Azure account context:

```powershell
az account show
```

Important fields include:

```text
tenantId
tenantDisplayName
tenantDefaultDomain
```

For learning documentation, the actual account-specific values are not required.

---

## Tenant in Authentication Flow

A simplified authentication flow looks like:

```text
User / Application
        ↓
Microsoft Entra ID
        ↓
Authentication
        ↓
Access Token
        ↓
Azure Resource
```

For example, when a user logs into Azure CLI:

```text
az login
   ↓
Microsoft Entra ID
   ↓
User Authentication
   ↓
Azure CLI Session
   ↓
Azure Subscription
```

---

## Real DevOps Scenario

Suppose I join an existing Azure project and `az login` succeeds, but I cannot see the expected subscription.

I would first check the current Azure context:

```powershell
az account show
```

Then I can list the subscriptions available to my account:

```powershell
az account list -o table
```

This helps verify whether I am connected to the expected tenant and subscription.

---

## Interview Question

### What is an Azure Tenant?

### Simple Interview Answer

> An Azure Tenant is mainly the identity boundary of an organization. It is associated with Microsoft Entra ID and contains users, groups, applications and identities. Azure subscriptions are associated with the tenant and use it for authentication and identity management.

---

## Interview Scenario

### Question

**You logged into Azure but cannot find the expected subscription. What would you check?**

### Answer

> First I check the current account and tenant context using `az account show`. Then I list the available subscriptions using `az account list`. This helps me verify whether I'm connected to the correct tenant and whether my account has access to the expected subscription.

---

## Key Takeaway

```text
Tenant
  ↓
Identity Boundary
  ↓
Microsoft Entra ID
  ↓
Users / Groups / Applications / Identities
```

**Remember:**

> **Tenant = Identity boundary**
