# Azure Resource Providers

## What is an Azure Resource Provider?

An Azure Resource Provider (RP) is the **Azure service namespace that manages a particular type of Azure resource**.

For example:

```text
Microsoft.Compute
Microsoft.Network
Microsoft.Storage
Microsoft.ContainerService
Microsoft.KeyVault
Microsoft.Sql
```

### Easy Memory

> **Resource Provider = Azure service namespace responsible for managing a resource**

---

# Real Examples

| Azure Resource  | Resource Provider            |
| --------------- | ---------------------------- |
| Virtual Machine | `Microsoft.Compute`          |
| Virtual Network | `Microsoft.Network`          |
| Storage Account | `Microsoft.Storage`          |
| AKS             | `Microsoft.ContainerService` |
| Key Vault       | `Microsoft.KeyVault`         |
| Azure SQL       | `Microsoft.Sql`              |

Example:

```text
Virtual Machine
      ↓
Microsoft.Compute
      ↓
Azure manages the VM resource
```

---

# Resource Provider vs Resource

These are different things.

### Resource Provider

Defines/manages a category of Azure resources.

```text
Microsoft.Compute
```

### Resource

The actual instance created under that provider.

```text
Virtual Machine: vm-payment-prod
```

So:

```text
Microsoft.Compute
       ↓
Virtual Machine
       ↓
vm-payment-prod
```

### Easy Memory

> **Provider = Who manages it**
> **Resource = What you created**

---

# Resource Provider Namespace

A Resource ID contains the provider namespace.

Example:

```text
/subscriptions/<subscription-id>
/resourceGroups/<resource-group>
/providers/Microsoft.Compute
/virtualMachines/<vm-name>
```

Here:

```text
Microsoft.Compute
       ↑
Resource Provider
```

Another example:

```text
/subscriptions/<subscription-id>
/resourceGroups/<resource-group>
/providers/Microsoft.Storage
/storageAccounts/<storage-account>
```

Here:

```text
Microsoft.Storage
       ↑
Resource Provider
```

---

# Check Resource Providers

Azure CLI can show Resource Providers available in the subscription.

```bash
az provider list -o table
```

To check a specific provider:

```bash
az provider show \
  --namespace Microsoft.Compute \
  -o table
```

You can also check its registration state:

```bash
az provider show \
  --namespace Microsoft.Compute \
  --query registrationState \
  -o tsv
```

Possible output:

```text
Registered
```

---

# Hands-On Lab — Register a Resource Provider

## Objective

In this lab, we will:

```text
1. Check Resource Provider status
2. Register the provider
3. Verify registration
```

We will use:

```text
Microsoft.Compute
```

No resource needs to be created for this lab.

---

## Step 1 — Check Current Subscription

First verify which Azure subscription is active.

```bash
az account show -o table
```

Confirm that you are working with the intended subscription.

---

## Step 2 — Check Provider Status

Run:

```bash
az provider show \
  --namespace Microsoft.Compute \
  --query registrationState \
  -o tsv
```

Possible output:

```text
Registered
```

or:

```text
NotRegistered
```

If it already shows:

```text
Registered
```

you can still continue the lab by understanding the registration and verification commands.

---

## Step 3 — Register the Provider

Run:

```bash
az provider register \
  --namespace Microsoft.Compute
```

Expected response will contain information about the provider registration request.

---

## Step 4 — Verify Registration

Run:

```bash
az provider show \
  --namespace Microsoft.Compute \
  --query registrationState \
  -o tsv
```

Expected final state:

```text
Registered
```

If the result initially shows another state, wait for registration to complete and check again.

---

## Step 5 — Check Provider Details

Run:

```bash
az provider show \
  --namespace Microsoft.Compute \
  --query "{Namespace:namespace,State:registrationState}" \
  -o table
```

Example output:

```text
Namespace           State
------------------  ----------
Microsoft.Compute   Registered
```

---

# Hands-On Understanding

The flow we just performed is:

```text
Azure Subscription
       ↓
Check Resource Provider
       ↓
Microsoft.Compute
       ↓
Check Registration State
       ↓
Register if required
       ↓
Verify
       ↓
Registered
```

---

# Optional — Check Another Provider

You can repeat the same exercise with:

```text
Microsoft.Network
```

Check:

```bash
az provider show \
  --namespace Microsoft.Network \
  --query registrationState \
  -o tsv
```

Register if required:

```bash
az provider register \
  --namespace Microsoft.Network
```

Verify:

```bash
az provider show \
  --namespace Microsoft.Network \
  --query registrationState \
  -o tsv
```

---

# Important Learning Point

Provider registration does **not** create an Azure resource.

For example:

```text
Register Microsoft.Compute
        ↓
Does NOT create a VM
```

It only enables the subscription to use resources provided through that Resource Provider, subject to permissions and service availability.

---

# Real Production Scenario

Suppose Terraform is creating a VM and deployment fails with:

```text
Resource provider Microsoft.Compute is not registered.
```

I would first verify the provider:

```bash
az provider show \
  --namespace Microsoft.Compute \
  --query registrationState \
  -o tsv
```

If it is not registered and I have the required permission:

```bash
az provider register \
  --namespace Microsoft.Compute
```

Then I verify the state:

```bash
az provider show \
  --namespace Microsoft.Compute \
  --query registrationState \
  -o tsv
```

Once it becomes:

```text
Registered
```

I can retry the deployment.

---

# Resource Provider and Terraform

Example:

```hcl
resource "azurerm_linux_virtual_machine" "vm" {
  name                = "<vm-name>"
  resource_group_name = "<resource-group>"
  location            = "<location>"
}
```

The interaction is:

```text
Terraform
   ↓
AzureRM Provider
   ↓
Azure API
   ↓
Microsoft.Compute
   ↓
Virtual Machine
```

For a VNet:

```text
Terraform
   ↓
AzureRM Provider
   ↓
Azure API
   ↓
Microsoft.Network
   ↓
Virtual Network
```

---

# Important Interview Point

Resource Provider registration is generally a **subscription-level requirement**.

Example:

```text
Subscription
│
├── Microsoft.Compute → Registered
├── Microsoft.Network → Registered
├── Microsoft.Storage → Registered
└── Microsoft.ContainerService → Registered
```

Once a required provider is registered, resources from that provider can be created subject to the appropriate permissions, service availability and configuration.

---

# Interview Scenario

### Question:

**Terraform is failing while creating an Azure resource. You receive an error saying the Resource Provider is not registered. What will you do?**

### Answer:

> First I will identify which Resource Provider is required and check its registration state using Azure CLI. If it is not registered and I have the required permission, I will register the provider and wait until the state becomes Registered. Then I will retry the Terraform deployment.

---

# Another Interview Question

### Question:

**What is the difference between a Resource Provider and a Resource?**

### Answer:

> A Resource Provider is the Azure service namespace that manages a type of resource. The resource is the actual instance we create. For example, `Microsoft.Compute` manages Virtual Machines, while `vm-payment-prod` would be the actual VM resource.

---

# Key Takeaways

```text
Resource Provider
        ↓
Azure service namespace
        ↓
Manages a resource type

Microsoft.Compute
        ↓
Virtual Machine

Microsoft.Network
        ↓
Virtual Network

Microsoft.Storage
        ↓
Storage Account

Microsoft.ContainerService
        ↓
AKS
```

### Memory Trick

> **Provider = Who manages it**
> **Resource = What you created**
