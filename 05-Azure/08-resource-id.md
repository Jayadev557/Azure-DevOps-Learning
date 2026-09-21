# Azure Resource ID

## What is an Azure Resource ID?

An Azure Resource ID is a **unique identifier used to identify an Azure resource**.

Every Azure resource has its own Resource ID.

### Easy Memory

> **Resource ID = Unique address of an Azure resource**

---

# Resource ID Structure

A typical Azure Resource ID looks like:

```text
/subscriptions/<subscription-id>
/resourceGroups/<resource-group>
/providers/<resource-provider>
/<resource-type>/<resource-name>
```

Example:

```text
/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/virtualMachines/<vm-name>
```

Breakdown:

```text
/subscriptions/<subscription-id>
        ↓
Which subscription?

/resourceGroups/<resource-group>
        ↓
Which Resource Group?

/providers/Microsoft.Compute
        ↓
Which Resource Provider?

/virtualMachines/<vm-name>
        ↓
Which resource?
```

---

# Another Example — Storage Account

```text
/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

Here:

```text
Microsoft.Storage
        ↓
Resource Provider

storageAccounts
        ↓
Resource Type

<storage-account>
        ↓
Resource Name
```

---

# Get Resource ID Using Azure CLI

You can get the Resource ID of a resource using:

```bash
az resource show \
  --resource-group <resource-group> \
  --name <resource-name> \
  --resource-type <resource-type> \
  --query id \
  -o tsv
```

Example:

```bash
az resource show \
  --resource-group <resource-group> \
  --name <vm-name> \
  --resource-type Microsoft.Compute/virtualMachines \
  --query id \
  -o tsv
```

Output:

```text
/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/virtualMachines/<vm-name>
```

---

# Get Resource IDs of All Resources in a Resource Group

Run:

```bash
az resource list \
  --resource-group <resource-group> \
  --query "[].{Name:name,Type:type,ID:id}" \
  -o table
```

This is useful when you want to understand an existing environment.

Example output:

```text
Name              Type                              ID
----------------  --------------------------------  -------------------------------
<vm-name>         Microsoft.Compute/virtualMachines /subscriptions/<subscription-id>/...
<vnet-name>       Microsoft.Network/virtualNetworks /subscriptions/<subscription-id>/...
<storage-account> Microsoft.Storage/storageAccounts /subscriptions/<subscription-id>/...
```

---

# Resource ID in Terraform

Resource IDs are commonly used when one Azure resource needs to reference another.

Example:

```hcl
resource "azurerm_network_interface" "nic" {
  name                = "<nic-name>"
  resource_group_name = "<resource-group>"
  location            = "<location>"

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "Dynamic"
  }
}
```

Here:

```text
azurerm_subnet.subnet.id
            ↓
Subnet Resource ID
            ↓
NIC uses that ID
```

The dependency looks like:

```text
VNet
 ↓
Subnet
 ↓
Subnet Resource ID
 ↓
NIC
```

---

# Real Production Scenario

Suppose an application team tells me:

> "The VM needs access to a specific subnet."

Instead of manually guessing the subnet name, I can work with the subnet's Resource ID.

```text
Application
     ↓
NIC
     ↓
Subnet Resource ID
     ↓
Azure VNet/Subnet
```

In Terraform, this is useful because resources can reference each other directly.

---

# Resource ID and RBAC

Azure RBAC can be assigned at different scopes.

For example:

```text
Management Group
      ↓
Subscription
      ↓
Resource Group
      ↓
Resource
```

A resource-level permission can target a specific resource using its Resource ID.

Conceptually:

```text
User / Service Principal / Managed Identity
              ↓
          RBAC Role
              ↓
        Resource Scope
              ↓
        Resource ID
```

Example:

```text
Storage Account
       ↓
Resource ID
       ↓
Storage Blob Data Reader
       ↓
Managed Identity
```

---

# Resource ID and Azure Monitor

Resource IDs are also useful for identifying resources in monitoring and operational workflows.

For example:

```text
Alert
  ↓
Resource ID
  ↓
AKS / VM / Storage / Database
```

When investigating an alert, the Resource ID helps identify exactly which Azure resource generated or is associated with the alert.

---

# Hands-On Lab

## Objective

In this lab, we will:

```text
1. List resources
2. Find a resource ID
3. Understand its components
```

### Step 1 — List Resources

```bash
az resource list -o table
```

Identify any resource you are allowed to inspect.

---

### Step 2 — Get Its Resource ID

```bash
az resource show \
  --resource-group <resource-group> \
  --name <resource-name> \
  --resource-type <resource-type> \
  --query id \
  -o tsv
```

---

### Step 3 — Read the Resource ID

Take the returned ID and identify:

```text
Subscription
Resource Group
Resource Provider
Resource Type
Resource Name
```

For example:

```text
/subscriptions/<subscription-id>
/resourceGroups/<resource-group>
/providers/Microsoft.Storage
/storageAccounts/<storage-account>
```

---

# Interview Scenario

### Question:

**What is an Azure Resource ID and where have you used it?**

### Answer:

> Resource ID is the unique identifier of an Azure resource. I commonly use it in Terraform when one resource needs to reference another resource, and also in Azure RBAC, monitoring and troubleshooting to identify the exact resource and scope.

---

# Another Interview Question

### Question:

**What is the difference between Resource Name and Resource ID?**

### Answer:

> The Resource Name identifies the resource within its context, while the Resource ID is the complete unique path of that resource in Azure. For example, the VM name can be `<vm-name>`, while the Resource ID also contains the subscription, resource group, provider and resource type.

---

# Key Takeaways

```text
Resource Name
      ↓
<vm-name>

Resource ID
      ↓
/subscriptions/<subscription-id>
/resourceGroups/<resource-group>
/providers/Microsoft.Compute
/virtualMachines/<vm-name>
```

### Memory Trick

> **Name = Short identity**
> **Resource ID = Complete Azure address**
