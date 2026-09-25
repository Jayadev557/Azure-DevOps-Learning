# Azure Day-1 Scenario-Based Questions

## 1. Scenario: Wrong Azure Subscription

### Question

You run an Azure CLI command, but you don't see the resources you expected. What would you check first?

### Answer

> "First, I would check the active Azure subscription using `az account show`. If I have access to multiple subscriptions, I would verify that I'm working in the correct subscription and use `az account set` if required. Then I would check the Resource Group and resources."

### Commands

```bash
az account show -o table
az account list -o table
az account set --subscription <subscription-id>
```

### Flow

```text
CLI
 ↓
Check Subscription
 ↓
Check Resource Group
 ↓
Check Resources
```

---

# 2. Scenario: Resource Not Found

### Question

A developer says a VM exists, but you cannot find it in the Azure Portal or CLI. What would you check?

### Answer

> "I would first verify the subscription and Resource Group. Then I would list the resources using Azure CLI and check whether the resource name or resource type is correct. I would also verify that I'm looking in the correct region and environment."

### Commands

```bash
az account show -o table
```

```bash
az group list -o table
```

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

---

# 3. Scenario: Deployment Failed

### Question

A resource deployment failed. How would you troubleshoot it?

### Answer

> "First, I would check the resource provisioning state and the deployment or activity logs. Then I would identify the actual error, such as permission, quota, networking or provider registration. After fixing the root cause, I would retry the deployment."

### Commands

```bash
az resource show \
  --ids <resource-id> \
  --query "{Name:name,State:provisioningState}" \
  -o table
```

```bash
az monitor activity-log list \
  --resource-group <resource-group> \
  --status Failed \
  -o table
```

### Flow

```text
Deployment Failed
       ↓
Check Error
       ↓
Identify Root Cause
       ↓
Fix
       ↓
Retry
```

---

# 4. Scenario: Resource Provider Not Registered

### Question

Terraform or Azure CLI says the resource provider is not registered. What would you do?

### Answer

> "I would identify which resource provider is required and check its registration state. If it is not registered, I would register it at the subscription level and then retry the deployment."

### Commands

```bash
az provider show \
  --namespace <provider-namespace> \
  --query registrationState \
  -o tsv
```

Register:

```bash
az provider register \
  --namespace <provider-namespace>
```

Check again:

```bash
az provider show \
  --namespace <provider-namespace> \
  --query registrationState \
  -o tsv
```

---

# 5. Scenario: Application Is Not Reachable

### Question

A production application is deployed, but users cannot access it. What would you check?

### Answer

> "I would troubleshoot from the entry point toward the application. I would check the Application Gateway or Load Balancer, public or private connectivity, NSG rules, backend health and finally the application itself. If it's AKS, I would also check nodes, pods and services."

### Flow

```text
User
 ↓
DNS
 ↓
Application Gateway / Load Balancer
 ↓
NSG / Network
 ↓
AKS / VM / App Service
 ↓
Application
```

For AKS:

```bash
kubectl get nodes
kubectl get pods
kubectl get svc
```

---

# 6. Scenario: Resource Group Is in One Region but Resource Is in Another

### Question

Can a Resource Group and its resources be in different Azure regions?

### Answer

> "Yes. A Resource Group has its own location, but it does not force every resource inside it to be deployed in the same region. Resource placement depends on the individual resource and service requirements."

Example:

```text
Resource Group
Location = <region-a>

Resources:
    VM        → <region-a>
    Storage   → <region-b>
```

The exact design should follow the organization's architecture and service requirements.

---

# 7. Scenario: Region Selection

### Question

How would you choose an Azure region for a production application?

### Answer

> "I would consider customer latency, data residency and compliance requirements, service and SKU availability, Availability Zone support, cost and disaster recovery requirements. I would select the region based on the application's actual requirements."

### Memory Trick

```text
Region Selection
      ↓
Latency
Compliance
Service Availability
AZ Support
Cost
DR
```

---

# 8. Scenario: Availability Zone Failure

### Question

What happens if one Availability Zone has an infrastructure failure?

### Answer

> "If the application is designed across multiple Availability Zones and the required services support zone redundancy, workloads can continue running in the other zones. The exact behavior depends on the service architecture and how the application was deployed."

Example:

```text
Region
 |
 +-- Zone 1 → Application
 |
 +-- Zone 2 → Application
 |
 +-- Zone 3 → Application
```

---

# 9. Scenario: Tags Missing

### Question

Your organization wants every production resource to have Environment, Owner and CostCenter tags. Some resources don't have them. What would you do?

### Answer

> "I would first define the required tagging standard. Then I would use Azure Policy to enforce or modify the required tags according to the organization's governance design. Existing resources can also be remediated where appropriate."

Example:

```text
Environment = Production
Owner       = Platform-Team
CostCenter  = CC1001
```

---

# 10. Scenario: Cannot Identify Resource Owner

### Question

You find an old Azure resource but nobody knows which team owns it. How can tags help?

### Answer

> "If the organization has a standard tagging strategy, I can check tags such as Owner, Application and CostCenter. This can help identify the responsible team and business application. If the tags are missing, I would follow the organization's ownership and governance process before making changes."

---

# 11. Scenario: Need to Quickly Check 100 Resources

### Question

Would you open every resource in the Portal?

### Answer

> "For a large number of resources, I would use Azure CLI because it is faster and can be scripted. I can use `az resource list` with `--query` to extract only the information I need."

Example:

```bash
az resource list \
  --query "[].{Name:name,Type:type,Location:location}" \
  -o table
```

---

# 12. Scenario: Portal Works but CLI Doesn't

### Question

The resource is visible in Portal, but your Azure CLI command doesn't find it. What would you check?

### Answer

> "I would first check whether CLI is logged into the correct tenant and subscription. Then I would verify the Resource Group, resource name and resource type. I would also check whether my account has sufficient permissions."

### Flow

```text
Portal Resource
      ↓
CLI Login
      ↓
Tenant
      ↓
Subscription
      ↓
Resource Group
      ↓
RBAC Permission
      ↓
Resource
```

---

# 13. Scenario: Dev and Production Resources

### Question

How would you separate Dev and Production resources?

### Answer

> "The exact design depends on the organization's governance model. We can use separate subscriptions, Resource Groups or a combination of both. For production, I would also apply appropriate access control, policies, tagging and monitoring."

Example:

```text
Tenant
 |
 +-- Dev Subscription
 |      |
 |      +-- Dev RGs
 |
 +-- Production Subscription
        |
        +-- Production RGs
```

---

# 14. Scenario: Resource Has Failed Provisioning State

### Question

You see a resource with a failed provisioning state. What would you do?

### Answer

> "I would not immediately delete or recreate it. First I would inspect the resource details and Activity Logs to identify the failure. Then I would check dependencies such as networking, permissions, quota or provider registration and fix the actual cause."

### Commands

```bash
az resource show \
  --ids <resource-id> \
  -o json
```

```bash
az monitor activity-log list \
  --resource-group <resource-group> \
  --status Failed \
  -o table
```

---

# 15. Scenario: Access to Azure Resource

### Question

A developer needs access to one Azure resource. Would you give them Owner access to the entire subscription?

### Answer

> "I would follow least-privilege access. Instead of giving subscription-level Owner access, I would assign the required Azure RBAC role at the narrowest appropriate scope, such as the resource or Resource Group."

Example:

```text
Subscription
   |
   +-- Resource Group
          |
          +-- Resource
                ↑
             RBAC Role
```

---

# 16. Scenario: Resource ID Needed

### Question

Where would you use an Azure Resource ID?

### Answer

> "Resource IDs are commonly used when referencing Azure resources in RBAC, monitoring, automation and Infrastructure as Code. For example, Terraform or Azure Monitor configurations may need the exact Resource ID of a resource."

Get it using:

```bash
az resource show \
  --ids <resource-id> \
  --query id \
  -o tsv
```

---

# 17. Scenario: Need to Troubleshoot Without Portal

### Question

The Azure Portal is temporarily unavailable to you. Can you still troubleshoot?

### Answer

> "Yes. I can use Azure CLI or PowerShell to inspect resources, provisioning states, activity logs and networking. For application platforms such as AKS, I can also use Kubernetes commands like `kubectl`."

Example:

```text
Azure CLI
   ↓
Azure Resources
   ↓
Activity Logs
   ↓
Networking
   ↓
AKS
   ↓
kubectl
```

---

# 18. Scenario: Production Resource Creation

### Question

Would you manually create production infrastructure from the Portal?

### Answer

> "For repeatable production infrastructure, I would normally use Infrastructure as Code such as Terraform. The Portal can still be useful for investigation, verification and certain operational tasks. This gives us consistency and makes infrastructure changes reviewable."

---

# 19. Scenario: Complete Azure Troubleshooting Flow

### Question

A production application is down. Explain your troubleshooting approach.

### Answer

> "First, I confirm the correct subscription and Resource Group. Then I check the affected Azure resources and their provisioning state. I move through networking, identity, dependencies and application health, and I check Activity Logs and monitoring data for errors. If the application is running on AKS, I continue with Kubernetes-level checks like nodes, pods, services and logs."

### Practical Flow

```text
User reports issue
       ↓
Check Subscription
       ↓
Check Resource Group
       ↓
Check Resource State
       ↓
Check Activity Logs
       ↓
Check DNS / Network
       ↓
Check NSG / Firewall
       ↓
Check Load Balancer / App Gateway
       ↓
Check Compute / AKS
       ↓
Check Application
       ↓
Check Logs / Monitoring
```

---

# 20. Quick Interview Revision

### Tenant

> "Tenant is the identity boundary where users, groups, applications and identities are managed."

### Subscription

> "Subscription is an important resource, billing, access and quota boundary."

### Resource Group

> "Resource Group is a logical container used to organize and manage related Azure resources."

### Region

> "Region is the geographic Azure location where resources are deployed."

### Availability Zone

> "An Availability Zone is a separate failure-isolated location within a supported Azure region."

### Resource

> "A resource is an actual Azure service instance such as a VM, AKS cluster, Storage Account or SQL database."

### Resource Provider

> "A Resource Provider is the Azure service namespace that manages a particular resource type."

### Resource ID

> "Resource ID is the unique identifier of an Azure resource."

### Tags

> "Tags are key-value metadata used for ownership, environment, cost tracking and governance."

### Azure CLI

> "Azure CLI lets us manage, inspect and automate Azure resources from the command line."

---

# 21. Day-1 Mental Model

```text
                 TENANT
                    |
              SUBSCRIPTION
                    |
             RESOURCE GROUP
                    |
       +------------+------------+
       |            |            |
    NETWORK       COMPUTE       DATA
       |            |            |
      VNet         AKS          SQL
      NSG          VM           Storage
      LB           ACR          Key Vault
       |
       +-------- REGION / AZ --------+
                    |
                  TAGS
                    |
              MONITORING
```

Management:

```text
Portal
  ↓
Visual Operations

CLI / PowerShell
  ↓
Commands + Automation

Terraform
  ↓
Infrastructure as Code
```

---

# 22. Final Interview Scenario

### Question

"Imagine you join a project and are given an Azure production environment. What would you understand first?"

### Answer

> "I would first understand the tenant and subscription structure, then the Resource Groups and region strategy. After that I would map the major resources such as networking, compute, databases, storage, identity and monitoring. Finally, I would understand the tagging, RBAC and Infrastructure-as-Code approach so I know how the environment is managed end to end."

---

# 23. Key Takeaways

```text
Don't memorize Azure services individually.

Understand the relationship:

Tenant
  ↓
Subscription
  ↓
Resource Group
  ↓
Resources
  ↓
Network / Compute / Data / Security
  ↓
Monitoring
```

For troubleshooting:

```text
Scope
 ↓
Resource
 ↓
Dependencies
 ↓
Logs
 ↓
Root Cause
 ↓
Fix
```

For production:

```text
Design
 ↓
Terraform
 ↓
CI/CD
 ↓
Azure
 ↓
Monitoring
 ↓
Operations
```
