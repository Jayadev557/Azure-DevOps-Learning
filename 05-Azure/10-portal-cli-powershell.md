# Azure Portal vs CLI vs PowerShell

## 1. Azure Management Options

Azure resources can be managed in different ways:

```text
Azure Portal → GUI
Azure CLI    → Commands
PowerShell   → Cmdlets
Terraform    → Infrastructure as Code
```

The important thing for a DevOps engineer is knowing **when to use each one**.

---

# 2. Azure Portal

Azure Portal is a web-based graphical interface.

We can use it to:

* Create Azure resources
* Check resource configuration
* Troubleshoot issues
* View monitoring and logs
* Check networking
* Manage access and permissions

Example:

```text
Azure Portal
     ↓
Resource Group
     ↓
AKS
     ↓
Networking
     ↓
Monitoring
     ↓
Logs
```

Portal is especially useful when we need to visually inspect a resource.

---

# 3. Azure CLI

Azure CLI is a command-line tool used to manage Azure resources.

Example:

```bash
az group list -o table
```

Check the current subscription:

```bash
az account show -o table
```

Get a Resource Group:

```bash
az group show --name <resource-group> -o json
```

List resources:

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

### Why DevOps Engineers Use CLI

Suppose we have 50 resources.

Using Portal:

```text
Open Portal
   ↓
Search resources
   ↓
Open each resource
   ↓
Check configuration
```

Using CLI:

```bash
az resource list --resource-group <resource-group> -o table
```

One command can quickly give us the resource list.

---

# 4. Azure PowerShell

Azure PowerShell provides PowerShell cmdlets for managing Azure.

Login:

```powershell
Connect-AzAccount
```

List subscriptions:

```powershell
Get-AzSubscription
```

List Resource Groups:

```powershell
Get-AzResourceGroup
```

Get a specific Resource Group:

```powershell
Get-AzResourceGroup -Name <resource-group>
```

List resources:

```powershell
Get-AzResource
```

PowerShell is especially useful when an organization already uses PowerShell heavily for administration and automation.

---

# 5. Portal vs CLI vs PowerShell

| Tool         | Typical Usage                                  |
| ------------ | ---------------------------------------------- |
| Azure Portal | Visual management and troubleshooting          |
| Azure CLI    | Commands, scripting and automation             |
| PowerShell   | PowerShell-based administration and automation |
| Terraform    | Repeatable Infrastructure as Code              |

Example:

```text
Need to check configuration?
        ↓
      Portal

Need a quick command?
        ↓
     Azure CLI

Need PowerShell automation?
        ↓
     PowerShell

Need repeatable infrastructure?
        ↓
      Terraform
```

---

# 6. Hands-On Lab

## Step 1: Login with Azure CLI

```bash
az login
```

Check the logged-in account:

```bash
az account show -o table
```

---

## Step 2: List Resource Groups

```bash
az group list -o table
```

Observe:

```text
Name
Location
ProvisioningState
```

---

## Step 3: Inspect a Resource Group

```bash
az group show \
  --name <resource-group> \
  -o json
```

Use a filtered query:

```bash
az group show \
  --name <resource-group> \
  --query "{Name:name,Location:location,State:properties.provisioningState}" \
  -o table
```

Expected type of output:

```text
Name              Location    State
----------------  ----------  ----------
<resource-group>  <location>  Succeeded
```

---

# 7. Common Azure CLI Troubleshooting Commands

These are useful commands to know during real production troubleshooting.

## 7.1 Check Current Login and Subscription

```bash
az account show -o table
```

Useful when commands are running against the wrong subscription.

List all subscriptions:

```bash
az account list -o table
```

Change subscription:

```bash
az account set --subscription <subscription-id>
```

---

## 7.2 Check Resource Group

```bash
az group show \
  --name <resource-group> \
  -o table
```

List all Resource Groups:

```bash
az group list -o table
```

---

## 7.3 Check Resources

List all resources in a Resource Group:

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

Find a specific resource:

```bash
az resource show \
  --resource-group <resource-group> \
  --name <resource-name> \
  --resource-type <resource-type> \
  -o json
```

Get only the resource ID:

```bash
az resource show \
  --resource-group <resource-group> \
  --name <resource-name> \
  --resource-type <resource-type> \
  --query id \
  -o tsv
```

---

## 7.4 Check Resource Provisioning State

```bash
az resource show \
  --ids <resource-id> \
  --query "{Name:name,State:provisioningState}" \
  -o table
```

This helps identify whether a resource is:

```text
Succeeded
Creating
Updating
Failed
Deleting
```

---

## 7.5 Check Activity Logs

If a deployment or resource operation failed, check Azure Activity Log:

```bash
az monitor activity-log list \
  --resource-group <resource-group> \
  --max-events 20 \
  -o table
```

This is useful for finding events such as:

```text
Resource creation failure
Deployment failure
Permission failure
Resource update
Resource deletion
```

---

## 7.6 Check Failed Operations

For a specific Resource Group:

```bash
az monitor activity-log list \
  --resource-group <resource-group> \
  --status Failed \
  -o table
```

This is useful when someone says:

> "The deployment failed. Find out what happened."

---

## 7.7 Check Network Resources

List network resources:

```bash
az network resource list \
  --resource-group <resource-group> \
  -o table
```

List VNets:

```bash
az network vnet list \
  --resource-group <resource-group> \
  -o table
```

List NSGs:

```bash
az network nsg list \
  --resource-group <resource-group> \
  -o table
```

List Public IPs:

```bash
az network public-ip list \
  --resource-group <resource-group> \
  -o table
```

List Load Balancers:

```bash
az network lb list \
  --resource-group <resource-group> \
  -o table
```

---

## 7.8 Check VM Status

List VMs:

```bash
az vm list \
  --resource-group <resource-group> \
  -o table
```

Check power state:

```bash
az vm get-instance-view \
  --resource-group <resource-group> \
  --name <vm-name> \
  --query "instanceView.statuses[].displayStatus" \
  -o tsv
```

Example output:

```text
VM running
```

---

## 7.9 Check NSG Rules

```bash
az network nsg rule list \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  -o table
```

Useful when troubleshooting:

```text
Application not reachable
        ↓
NSG
        ↓
Inbound rule
        ↓
Port 80 / 443 / application port
```

---

## 7.10 Check AKS

List AKS clusters:

```bash
az aks list \
  --resource-group <resource-group> \
  -o table
```

Get AKS details:

```bash
az aks show \
  --resource-group <resource-group> \
  --name <aks-name> \
  -o json
```

Check Kubernetes version:

```bash
az aks show \
  --resource-group <resource-group> \
  --name <aks-name> \
  --query kubernetesVersion \
  -o tsv
```

Get AKS credentials:

```bash
az aks get-credentials \
  --resource-group <resource-group> \
  --name <aks-name>
```

Then:

```bash
kubectl get nodes
```

---

# 8. Practical Troubleshooting Flow

Suppose the application is not reachable.

I would troubleshoot from Azure infrastructure toward the application:

```text
1. Check subscription
        ↓
az account show

2. Check Resource Group
        ↓
az group show

3. Check resource state
        ↓
az resource list

4. Check networking
        ↓
VNet / NSG / Public IP / Load Balancer

5. Check Activity Logs
        ↓
az monitor activity-log list

6. If AKS
        ↓
az aks show
        ↓
kubectl get nodes
        ↓
kubectl get pods
        ↓
kubectl get svc
```

This gives us a practical troubleshooting sequence instead of randomly checking resources.

---

# 9. Useful Output Formats

Azure CLI supports different output formats.

Table:

```bash
az resource list -o table
```

JSON:

```bash
az resource list -o json
```

TSV:

```bash
az resource list -o tsv
```

Use `table` when you want human-readable output.

Use `json` when you need detailed structured information.

Use `tsv` when you want clean output for scripts.

---

# 10. Azure CLI Query

The `--query` option is very useful during troubleshooting.

Example:

```bash
az resource list \
  --resource-group <resource-group> \
  --query "[].{Name:name,Type:type,Location:location}" \
  -o table
```

Instead of getting a large JSON response, we can extract only the information we need.

---

# 11. Same Task Using PowerShell

Login:

```powershell
Connect-AzAccount
```

List Resource Groups:

```powershell
Get-AzResourceGroup
```

Get one Resource Group:

```powershell
Get-AzResourceGroup -Name <resource-group>
```

List resources:

```powershell
Get-AzResource
```

---

# 12. Practical Production Example

Suppose a production application is not reachable.

As a DevOps engineer, I may first use Portal to visually inspect:

```text
Application
    ↓
Load Balancer / App Gateway
    ↓
Networking
    ↓
NSG
    ↓
Backend
    ↓
Monitoring
```

Then I can use CLI to quickly verify resource state:

```bash
az resource list \
  --resource-group <resource-group> \
  -o table
```

If the same checks are required repeatedly, I can automate them using Azure CLI or PowerShell.

---

# 13. Real DevOps Approach

In a real project, we don't normally depend on only one method.

```text
Portal
  → Troubleshooting / Visual inspection

CLI
  → Quick checks / Scripts / Automation

PowerShell
  → PowerShell-based administration

Terraform
  → Infrastructure provisioning
```

The tool depends on the requirement.

---

# 14. Interview Answer

**Question: Do you prefer Azure Portal or CLI?**

> "I use both depending on the requirement. I use Portal mainly for visual troubleshooting and checking configurations, while I use Azure CLI for quick checks and automation. For repetitive administration, I can use CLI or PowerShell scripts. For infrastructure provisioning, I prefer Terraform because it gives us repeatable Infrastructure as Code."

---

# 15. Key Takeaways

```text
Portal      → Visual management
Azure CLI   → Commands + troubleshooting + automation
PowerShell  → Cmdlets + automation
Terraform   → Infrastructure as Code
```

The important point is **knowing when to use each tool**, not just knowing the commands.
