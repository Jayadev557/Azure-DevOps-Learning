# Real Azure Project Structure

## 1. Why We Need a Project Structure

In a real production environment, we don't create Azure resources randomly.

We first define:

* Tenant
* Subscriptions
* Resource Groups
* Regions
* Availability Zones
* Resources
* Tags
* Networking
* Access
* Monitoring

A simple Azure structure looks like:

```text
Microsoft Entra Tenant
        |
        +-----------------------------+
        |                             |
   Dev Subscription             Production Subscription
        |                             |
   Resource Groups               Resource Groups
        |                             |
   Azure Resources               Azure Resources
```

---

# 2. Example Production Application

Suppose our company has a real-time payment application.

The application contains:

```text
Frontend
Backend APIs
Database
Storage
Container Platform
Secrets
Monitoring
```

A possible Azure architecture:

```text
                    Azure Tenant
                         |
                  Production Subscription
                         |
                payment-prod-rg
                         |
       +-----------------+------------------+
       |                 |                  |
      AKS              Azure SQL         Storage
       |                                    |
    Backend APIs                         Files/Data
       |
    Frontend
       |
    App Gateway
       |
      Users
```

---

# 3. Tenant Level

The Microsoft Entra tenant provides the identity boundary.

```text
Microsoft Entra Tenant
        |
        +-- Users
        +-- Groups
        +-- Applications
        +-- Service Principals
        +-- Managed Identities
```

Authentication and authorization are handled through Microsoft Entra ID.

For example:

```text
DevOps Engineer
      |
      ↓
Microsoft Entra ID
      |
      ↓
Azure RBAC
      |
      ↓
Azure Resource
```

---

# 4. Subscription Level

A subscription provides an important boundary for:

* Resources
* Billing
* Access control
* Quotas
* Governance

A company may separate environments using subscriptions:

```text
Azure Tenant
     |
     +-- Dev Subscription
     |
     +-- UAT Subscription
     |
     +-- Production Subscription
```

Another organization may use fewer subscriptions and separate environments using Resource Groups.

The exact structure depends on organizational requirements.

---

# 5. Resource Group Structure

Inside the Production subscription, resources can be organized using Resource Groups.

Example:

```text
Production Subscription
        |
        +-- payment-prod-network-rg
        |
        +-- payment-prod-app-rg
        |
        +-- payment-prod-data-rg
        |
        +-- payment-prod-monitoring-rg
```

The exact Resource Group strategy depends on how the organization manages lifecycle, ownership and access.

---

# 6. Application Resource Group

Example:

```text
payment-prod-app-rg
        |
        +-- AKS
        +-- ACR
        +-- Application Gateway
        +-- Managed Identity
```

Application resources are grouped logically.

---

# 7. Data Resource Group

Example:

```text
payment-prod-data-rg
        |
        +-- Azure SQL
        +-- Storage Account
        +-- Key Vault
```

This can make ownership and access management easier.

---

# 8. Network Resource Group

Example:

```text
payment-prod-network-rg
        |
        +-- VNet
        +-- Subnets
        +-- NSGs
        +-- Route Tables
        +-- Private Endpoints
        +-- Load Balancer
```

The actual design depends on the organization's architecture.

---

# 9. Region Selection

Suppose the application serves customers in a particular geography.

We select a suitable Azure region based on requirements such as:

```text
Region Selection
       |
       +-- Customer latency
       +-- Data residency
       +-- Compliance
       +-- Service availability
       +-- Availability Zones
       +-- Cost
       +-- Disaster Recovery
```

Example:

```text
Production
     |
     +-- Primary Region
     |
     +-- Secondary Region
```

The secondary region can be used as part of a disaster recovery strategy.

---

# 10. Availability Zones

If the selected region supports Availability Zones for the required services, production workloads can use multiple zones.

Example:

```text
Primary Region
|
+-- Zone 1
|    |
|    +-- Application instances
|
+-- Zone 2
|    |
|    +-- Application instances
|
+-- Zone 3
     |
     +-- Application instances
```

The goal is to reduce the impact of a failure affecting one physical zone.

Not every Azure service or SKU supports Availability Zones in every region.

---

# 11. Network Structure

A typical application may have:

```text
VNet
 |
 +-- Application Subnet
 |
 +-- AKS Subnet
 |
 +-- Database Subnet
 |
 +-- Private Endpoint Subnet
```

Example traffic flow:

```text
Internet
   |
   ↓
Application Gateway / WAF
   |
   ↓
AKS / Application
   |
   ↓
Private Network
   |
   ↓
Azure SQL
```

NSGs, route tables, firewalls and private endpoints can be used according to the security architecture.

---

# 12. Resource Structure

A production application may contain:

```text
Resource Group
      |
      +-- VNet
      +-- Subnets
      +-- NSGs
      +-- AKS
      +-- ACR
      +-- Application Gateway
      +-- Key Vault
      +-- Azure SQL
      +-- Storage Account
      +-- Log Analytics
      +-- Application Insights
```

Each resource has a specific responsibility.

---

# 13. Tags

We should apply a consistent tagging strategy.

Example:

```text
Environment = Production
Application = Payment
Owner       = Platform-Team
CostCenter  = CC1001
Project     = Payment-Platform
```

Tags help with:

* Cost analysis
* Ownership
* Resource identification
* Operations
* Reporting
* Governance

---

# 14. Resource Providers

Azure resources are managed through Resource Providers.

For example:

```text
Microsoft.Compute
    ↓
Virtual Machines

Microsoft.Network
    ↓
VNet / NSG / Public IP

Microsoft.Storage
    ↓
Storage Account

Microsoft.ContainerService
    ↓
AKS

Microsoft.KeyVault
    ↓
Key Vault

Microsoft.Sql
    ↓
Azure SQL
```

The provider must be available/registered in the subscription before the corresponding resource types can be created.

---

# 15. Resource ID

Every Azure resource has a unique Resource ID.

Example structure:

```text
/subscriptions/<subscription-id>
/resourceGroups/<resource-group>
/providers/Microsoft.Storage
/storageAccounts/<storage-account>
```

For a VM:

```text
/subscriptions/<subscription-id>
/resourceGroups/<resource-group>
/providers/Microsoft.Compute
/virtualMachines/<vm-name>
```

Resource IDs are commonly used for:

* RBAC
* Monitoring
* Automation
* Resource references
* Terraform
* Azure CLI

---

# 16. How Everything Connects

Now combine everything:

```text
Microsoft Entra Tenant
        |
        ↓
Subscription
        |
        ↓
Resource Groups
        |
        +-----------------------------+
        |                             |
   Network RG                     App RG
        |                             |
        +-- VNet                      +-- AKS
        +-- Subnets                   +-- ACR
        +-- NSGs                      +-- App Gateway
        +-- Private Endpoints         +-- Managed Identity
        |
        +-----------------------------+
                      |
                   Data RG
                      |
                 +----+----+
                 |         |
              Azure SQL  Storage
```

Across the architecture:

```text
Region
  ↓
Availability Zones
  ↓
Resources
  ↓
Tags
  ↓
Monitoring
```

---

# 17. Where Azure CLI Fits

As a DevOps engineer, I can inspect the environment using Azure CLI.

Check subscription:

```bash
az account show -o table
```

List Resource Groups:

```bash
az group list -o table
```

List resources:

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

Check a resource:

```bash
az resource show \
  --ids <resource-id> \
  -o json
```

Check Activity Logs:

```bash
az monitor activity-log list \
  --resource-group <resource-group> \
  --max-events 20 \
  -o table
```

---

# 18. Where Terraform Fits

In a real DevOps environment, we don't normally create the entire infrastructure manually.

Terraform can define the infrastructure as code.

Example:

```text
Git Repository
      |
      ↓
Terraform Code
      |
      ↓
Terraform Plan
      |
      ↓
Approval
      |
      ↓
Terraform Apply
      |
      ↓
Azure Resources
```

Example Terraform structure:

```text
terraform-project/
|
+-- modules/
|   +-- network/
|   +-- aks/
|   +-- storage/
|   +-- database/
|
+-- environments/
    +-- dev/
    +-- uat/
    +-- prod/
```

This makes infrastructure repeatable and easier to manage.

---

# 19. Real Production Deployment Flow

A typical high-level flow can look like:

```text
Developer
    |
    ↓
Git Repository
    |
    ↓
CI/CD Pipeline
    |
    ↓
Terraform
    |
    ↓
Azure Infrastructure
    |
    +-- Network
    +-- AKS
    +-- ACR
    +-- Database
    +-- Key Vault
    |
    ↓
Application Deployment
    |
    ↓
Monitoring
    |
    ↓
Alerts / Incident Response
```

---

# 20. Production Troubleshooting Example

Suppose users report:

> "The payment application is not responding."

As a DevOps engineer, I don't immediately restart everything.

I follow the architecture.

```text
1. Check subscription
        ↓
2. Check Resource Group
        ↓
3. Check application resources
        ↓
4. Check Application Gateway / Load Balancer
        ↓
5. Check NSG / networking
        ↓
6. Check AKS
        ↓
7. Check pods
        ↓
8. Check database connectivity
        ↓
9. Check Key Vault / identity
        ↓
10. Check monitoring and logs
```

For AKS:

```bash
az aks show \
  --resource-group <resource-group> \
  --name <aks-name> \
  -o table
```

Then:

```bash
kubectl get nodes
kubectl get pods
kubectl get svc
```

This is how the Azure hierarchy becomes useful during real troubleshooting.

---

# 21. Interview Scenario

**Question: How would you explain the Azure structure of your project?**

> "At the top we have the Microsoft Entra tenant, and under it we manage Azure subscriptions. We organize resources into Resource Groups based on application, network or data requirements. The resources are deployed in suitable Azure regions and, where supported, across Availability Zones for resilience. We use tags for ownership and cost tracking, and Terraform is used to provision the infrastructure consistently."

---

# 22. Interview Scenario: Why Resource Groups?

**Question: Why don't you put everything into one Resource Group?**

> "It depends on the project, but separating resources can help with lifecycle management, ownership and access control. For example, network resources may have a different lifecycle from application resources. So we can organize them into separate Resource Groups when that makes operational sense."

---

# 23. Interview Scenario: How Do You Troubleshoot Azure?

**Answer:**

> "I first confirm the subscription and Resource Group, then check the affected resource and its provisioning state. After that I check networking, identity, dependencies and activity logs. For application platforms like AKS, I continue with Kubernetes-level checks such as nodes, pods, services and application logs."

---

# 24. Final Mental Model

Remember this flow:

```text
TENANT
  ↓
SUBSCRIPTION
  ↓
RESOURCE GROUP
  ↓
RESOURCES
  ↓
NETWORK / COMPUTE / DATA / SECURITY
  ↓
MONITORING
```

And remember the location dimensions:

```text
REGION
  ↓
AVAILABILITY ZONES
  ↓
RESOURCES
```

And management:

```text
PORTAL → Visual
CLI    → Commands / Automation
TERRAFORM → Infrastructure as Code
```

And identification:

```text
RESOURCE
   |
   +-- Resource ID
   +-- Tags
```

---

# 25. Key Takeaways

```text
Tenant        → Identity boundary
Subscription  → Resource / billing boundary
RG            → Logical resource container
Region        → Geographic Azure location
AZ            → Failure-isolated location within a region
Resource      → Actual Azure service
Provider      → Service namespace
Resource ID   → Unique resource identifier
Tags          → Resource metadata
CLI           → Management / troubleshooting
Terraform     → Infrastructure as Code
```

The main goal is to understand **how these pieces connect in a real Azure environment**, not just memorize individual definitions.
