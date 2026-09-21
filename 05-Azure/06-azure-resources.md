# Azure Resources

## What is an Azure Resource?

An Azure Resource is an **individual service or component that you create and manage in Azure**.

Examples:

```text
Virtual Machine
AKS Cluster
Storage Account
Azure SQL Database
Virtual Network
Key Vault
Container Registry
Public IP
Load Balancer
```

### Easy Memory

> **Resource = Actual Azure service/component that you deploy and manage**

---

# Resource Group and Resources

Resources are organized inside Resource Groups.

Example:

```text
Subscription
│
└── Resource Group
    │
    ├── Virtual Network
    ├── AKS
    ├── Azure Container Registry
    ├── Key Vault
    ├── Storage Account
    └── Log Analytics Workspace
```

A Resource Group helps us manage related resources together.

---

# Resource Naming

In a real project, resources should follow a consistent naming convention.

Example:

```text
rg-payment-prod
aks-payment-prod
acrpaymentprod
kv-payment-prod
stpaymentprod
```

A common pattern is:

```text
<resource-type>-<application>-<environment>
```

Example:

```text
aks-payment-prod
```

Where:

```text
aks       → Resource type
payment   → Application
prod      → Environment
```

The exact naming convention depends on the organization's standards.

---

# Resource Lifecycle

A resource normally goes through a lifecycle like:

```text
Plan
  ↓
Create
  ↓
Configure
  ↓
Use
  ↓
Monitor
  ↓
Update / Scale
  ↓
Delete
```

For example, for an Azure VM:

```text
Create VM
   ↓
Configure Networking
   ↓
Install Application
   ↓
Monitor
   ↓
Scale / Patch
   ↓
Decommission
```

---

# Creating a Resource Using Azure CLI

Example: Create a Resource Group.

```bash
az group create \
  --name <resource-group> \
  --location <location>
```

Example: Create a Storage Account.

```bash
az storage account create \
  --name <storage-account> \
  --resource-group <resource-group> \
  --location <location> \
  --sku Standard_LRS
```

The actual resource name and supported configuration depend on Azure naming rules and service requirements.

---

# List Resources

To see resources in a Resource Group:

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

To list all resources available to the current subscription:

```bash
az resource list -o table
```

This is useful when troubleshooting an existing environment.

---

# Inspect a Specific Resource

Example:

```bash
az resource show \
  --resource-group <resource-group> \
  --name <resource-name> \
  --resource-type <resource-type>
```

For example, when troubleshooting a production environment, I may inspect a resource to understand its configuration, location, resource ID and other properties.

---

# Resource Dependencies

Resources often depend on other resources.

Example:

```text
AKS
 │
 ├── VNet
 │    └── Subnet
 │
 ├── ACR
 │
 ├── Key Vault
 │
 └── Log Analytics
```

Another example:

```text
VM
 │
 ├── NIC
 ├── VNet/Subnet
 ├── NSG
 ├── Public IP
 └── Managed Disk
```

Understanding these dependencies is important before modifying or deleting resources.

---

# Real Production Scenario

Suppose I join an existing Azure project and receive an issue:

> "The production application is not reachable."

I would not immediately restart or delete anything.

First I would identify the resources involved:

```text
Application
    ↓
Load Balancer / Application Gateway
    ↓
Backend
    ↓
VM / AKS
    ↓
Network
    ↓
Database / Storage
```

Then I would inspect the relevant resources and their configuration.

For example:

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

Then I can identify resources such as:

```text
AKS
VNet
Subnet
Load Balancer
Key Vault
Storage
Database
```

This gives me an initial view of the environment before troubleshooting.

---

# Azure Resource ID

Every Azure resource has a unique Resource ID.

Typical structure:

```text
/subscriptions/<subscription-id>
/resourceGroups/<resource-group>
/providers/<resource-provider>
/<resource-type>/<resource-name>
```

Example:

```text
/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>
```

The Resource ID is useful when working with:

* Azure CLI
* Terraform
* ARM/Bicep
* Azure Monitor
* RBAC
* Automation scripts

---

# Resource Tags

Tags are metadata attached to Azure resources.

Example:

```text
Environment = Production
Application = Payment
Owner       = Platform-Team
CostCenter  = Engineering
```

Tags help with:

* Resource identification
* Cost tracking
* Ownership
* Environment classification
* Governance
* Automation

Example:

```bash
az resource tag \
  --resource-group <resource-group> \
  --name <resource-name> \
  --resource-type <resource-type> \
  --tags Environment=Production Application=Payment
```

---

# Portal vs CLI vs Terraform

The same Azure resource can be managed using different methods.

```text
Azure Portal
     │
     ├── Manual management
     │
Azure CLI / PowerShell
     │
     ├── Scripts / automation
     │
Terraform
     │
     └── Infrastructure as Code
```

### Real Project Approach

For production infrastructure, I prefer Infrastructure as Code such as Terraform where applicable because infrastructure changes can be reviewed, version-controlled and reproduced.

CLI is very useful for:

* Troubleshooting
* Quick inspection
* Operational tasks
* Automation scripts

Portal is useful for:

* Visual inspection
* Initial exploration
* Checking configuration
* Troubleshooting

---

# Interview Answer

> In Azure, a resource is an individual service or component such as a VM, AKS cluster, VNet, Storage Account or Key Vault. In a project, I first understand how resources are grouped and how they depend on each other. For troubleshooting, I usually inventory the resources, check their configuration and dependencies, and then identify the failing component before making changes.

---

# Key Takeaways

```text
Resource
   ↓
Actual Azure service/component

Examples:
VM
AKS
VNet
Storage
SQL
Key Vault
ACR

Resource Group
   ↓
Logical container for resources

Resource ID
   ↓
Unique identifier

Tags
   ↓
Metadata for management and governance

CLI
   ↓
Inspection + automation

Terraform
   ↓
Infrastructure as Code
```
