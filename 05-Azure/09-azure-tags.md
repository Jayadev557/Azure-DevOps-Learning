# Azure Tags

## 1. What are Azure Tags?

Azure Tags are **key-value pairs** attached to Azure resources.

They help us identify and organize resources based on business or operational information.

Example:

```text
Environment = Dev
Application = Payment
Owner       = Platform-Team
CostCenter  = CC1001
```

### Easy Memory Trick

```text
Tag = Label for an Azure resource
```

---

# 2. Why Do We Use Tags?

In a real environment, we may have hundreds or thousands of Azure resources.

Tags help us answer questions like:

* Which environment does this resource belong to?
* Which application uses it?
* Who owns it?
* Which team should be charged?
* Which resources are production?
* Which resources can be included in automation?

Example:

```text
Resource: <resource-name>

Environment = Production
Application = Recharge-Portal
Owner       = DevOps-Team
CostCenter  = CC1001
```

Now the resource is easier to identify and manage.

---

# 3. Common Production Tags

A typical organization may use tags like:

```text
Environment = Dev
Application = Payment
Owner       = Platform-Team
CostCenter  = CC1001
Project     = Recharge-Portal
Department  = Engineering
Criticality = High
```

The exact tagging standard depends on the organization.

---

# 4. Tags and Cost Management

Suppose a company has:

```text
100 VMs
20 Storage Accounts
5 AKS Clusters
10 SQL Databases
```

The finance team wants to understand which application is consuming Azure resources.

If resources have:

```text
Application = Payment
CostCenter  = CC1001
Environment = Production
```

the organization can use those tags for cost analysis and reporting.

---

# 5. Tags for Operations

Tags are also useful during production support.

For example:

```text
Environment = Production
Owner       = Payments-Team
Application = Recharge-Portal
```

If an incident happens, the operations team can quickly identify:

```text
Which application?
Which environment?
Which team owns it?
```

---

# 6. Check Tags Using Azure CLI

Get the tags of a resource:

```bash
az resource show \
  --ids <resource-id> \
  --query tags \
  -o json
```

Example output:

```json
{
  "Environment": "Dev",
  "Application": "Azure-Learning",
  "Owner": "Platform-Team"
}
```

---

# 7. Hands-On Lab

For this lab, use a test resource group that you manage.

## Step 1: Create a Test Resource Group

```bash
az group create \
  --name <resource-group> \
  --location <location>
```

Example structure:

```text
<resource-group>
        |
        +-- Location: <location>
```

---

## Step 2: Add Tags During Creation

```bash
az group create \
  --name <resource-group> \
  --location <location> \
  --tags Environment=Dev Application=Azure-Learning Owner=Platform-Team
```

---

## Step 3: Verify the Tags

```bash
az group show \
  --name <resource-group> \
  --query tags \
  -o json
```

Expected:

```json
{
  "Environment": "Dev",
  "Application": "Azure-Learning",
  "Owner": "Platform-Team"
}
```

---

# 8. Add Another Tag

Add a Cost Center tag:

```bash
az group update \
  --name <resource-group> \
  --set tags.CostCenter=CC1001
```

Verify:

```bash
az group show \
  --name <resource-group> \
  --query tags \
  -o json
```

Expected:

```json
{
  "Environment": "Dev",
  "Application": "Azure-Learning",
  "Owner": "Platform-Team",
  "CostCenter": "CC1001"
}
```

---

# 9. Remove a Tag

Remove the Owner tag:

```bash
az group update \
  --name <resource-group> \
  --remove tags.Owner
```

Verify:

```bash
az group show \
  --name <resource-group> \
  --query tags \
  -o json
```

---

# 10. Resource-Level Tags

Tags can also be applied directly to resources.

First get the resource ID:

```bash
az resource show \
  --ids <resource-id> \
  --query id \
  -o tsv
```

Add a tag:

```bash
az resource update \
  --ids <resource-id> \
  --set tags.Environment=Dev
```

Add another:

```bash
az resource update \
  --ids <resource-id> \
  --set tags.Owner=Platform-Team
```

Verify:

```bash
az resource show \
  --ids <resource-id> \
  --query tags \
  -o json
```

---

# 11. Resource Group Tags vs Resource Tags

A Resource Group can have tags:

```text
Resource Group
    |
    +-- Tags
```

Individual resources can also have their own tags:

```text
Resource Group
    |
    +-- VM
    |    +-- Tags
    |
    +-- Storage
    |    +-- Tags
    |
    +-- AKS
         +-- Tags
```

Tags on a Resource Group are **not automatically inherited by existing resources**.

Organizations can use **Azure Policy** to enforce, add, or modify tags according to their governance requirements.

---

# 12. Real Production Scenario

Suppose the company has hundreds of Azure resources.

The finance team reports:

> "We don't know which application is consuming the Azure budget."

During investigation, we find that many resources don't have consistent ownership or cost-center information.

We introduce a tagging standard:

```text
Environment
Application
Owner
CostCenter
```

Example:

```text
Environment = Production
Application = Payment
Owner       = Payments-Team
CostCenter  = CC1001
```

Now resources can be grouped and analyzed based on these business attributes.

---

# 13. Interview Scenario

**Question: How have you used Azure Tags in a real project?**

> "We use tags to identify resources by environment, application, owner and cost center. For example, a production resource can have Environment=Prod and Application=Payment. This helps operations identify ownership and helps the organization with cost analysis and governance. We can also use Azure Policy to enforce the required tagging standard."

---

# 14. Important Interview Point

If the interviewer asks:

**"Are tags used only for cost management?"**

Answer:

> "No. Cost management is one use case, but tags are also useful for ownership, environment identification, operations, reporting and automation. In production, we normally define a standard tagging strategy and enforce it using governance policies."

---

# 15. Key Takeaways

```text
Tag
 ↓
Key + Value
 ↓
Identification
 ↓
Cost Management
 ↓
Ownership
 ↓
Operations
 ↓
Governance
```

### Remember:

```text
Tags = Metadata about resources
```
