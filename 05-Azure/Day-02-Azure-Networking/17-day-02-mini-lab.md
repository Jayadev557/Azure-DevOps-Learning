# Azure Networking — Day-02 Mini Lab

## 1. Lab Objective

In this lab, we will connect the major Azure networking concepts learned during Day-02.

We will practice:

```text
VNet
Subnet
NSG
Private IP
Public IP
NIC
Route Table
NAT Gateway
Private Endpoint
Private DNS
```

The goal is not to create a huge production environment.

The goal is to understand:

```text
How traffic enters
How traffic moves
How traffic is filtered
How private connectivity works
How outbound traffic works
How DNS resolves names
```

---

# 2. Target Architecture

We will build a simplified architecture:

```text
                         INTERNET
                            |
                            |
                      Public IP
                            |
                            v
                         VM / NIC
                            |
                       Private IP
                            |
                     +------+------+
                     |             |
                    NSG          Route
                     |             |
                     +------+------+
                            |
                            v
                          VNet
                            |
              +-------------+-------------+
              |                           |
              v                           v
         App Subnet              Private Endpoint Subnet
              |                           |
              v                           v
        Private VM                 Private Endpoint
                                          |
                                          v
                                     Azure Storage
                                          |
                                          v
                                     Private DNS
```

For outbound connectivity:

```text
Private VM
    |
    v
NAT Gateway
    |
    v
Public IP
    |
    v
Internet
```

---

# 3. Before Starting

Check your Azure login:

```bash
az account show -o table
```

Verify the subscription:

```bash
az account show \
  --query "{Subscription:id,Name:name}" \
  -o table
```

List resource groups:

```bash
az group list -o table
```

Use an existing resource group if you already have one.

Otherwise create a lab resource group:

```bash
az group create \
  --name <resource-group> \
  --location <location>
```

---

# 4. Create VNet

Create:

```text
VNet:
10.20.0.0/16
```

Command:

```bash
az network vnet create \
  --resource-group <resource-group> \
  --name <vnet-name> \
  --address-prefix 10.20.0.0/16
```

Verify:

```bash
az network vnet show \
  --resource-group <resource-group> \
  --name <vnet-name> \
  -o table
```

Expected architecture:

```text
VNet
10.20.0.0/16
```

---

# 5. Create Application Subnet

Create:

```text
App Subnet
10.20.1.0/24
```

Command:

```bash
az network vnet subnet create \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <app-subnet-name> \
  --address-prefixes 10.20.1.0/24
```

Verify:

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

---

# 6. Create Private Endpoint Subnet

Create a separate subnet:

```text
Private Endpoint Subnet
10.20.2.0/24
```

Command:

```bash
az network vnet subnet create \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <private-endpoint-subnet-name> \
  --address-prefixes 10.20.2.0/24
```

Verify:

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

Architecture:

```text
VNet
10.20.0.0/16
   |
   +-- App Subnet
   |     10.20.1.0/24
   |
   +-- Private Endpoint Subnet
         10.20.2.0/24
```

---

# 7. Create NSG

Create an NSG:

```bash
az network nsg create \
  --resource-group <resource-group> \
  --name <nsg-name>
```

Verify:

```bash
az network nsg list \
  --resource-group <resource-group> \
  -o table
```

---

# 8. Create HTTPS Inbound Rule

Create an example HTTPS rule:

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name Allow-HTTPS \
  --priority 100 \
  --source-address-prefixes Internet \
  --destination-port-ranges 443 \
  --protocol Tcp \
  --access Allow \
  --direction Inbound
```

List rules:

```bash
az network nsg rule list \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  -o table
```

---

# 9. Associate NSG with Application Subnet

Associate:

```text
NSG
  ↓
App Subnet
```

Command:

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <app-subnet-name> \
  --network-security-group <nsg-name>
```

Verify:

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <app-subnet-name> \
  --query networkSecurityGroup.id
```

---

# 10. Create a Route Table

Create:

```bash
az network route-table create \
  --resource-group <resource-group> \
  --name <route-table-name>
```

Verify:

```bash
az network route-table show \
  --resource-group <resource-group> \
  --name <route-table-name> \
  -o table
```

---

# 11. Understand the Route Table

For this lab, don't create a custom default route just for the sake of creating one.

First understand:

```text
VM
 |
 v
Subnet
 |
 v
Route Table
 |
 v
Azure routing
```

List routes:

```bash
az network route-table route list \
  --resource-group <resource-group> \
  --route-table-name <route-table-name> \
  -o table
```

A production UDR should be created only when you have a real routing requirement.

---

# 12. Associate Route Table with Subnet

Associate it with the application subnet:

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <app-subnet-name> \
  --route-table <route-table-name>
```

Verify:

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <app-subnet-name> \
  --query routeTable.id
```

Now:

```text
App Subnet
   |
   +-- NSG
   |
   +-- Route Table
```

---

# 13. Create Public IP for Lab VM

If you already have a suitable VM/public IP, you can skip this step.

Otherwise:

```bash
az network public-ip create \
  --resource-group <resource-group> \
  --name <public-ip-name> \
  --sku Standard \
  --allocation-method Static
```

Verify:

```bash
az network public-ip show \
  --resource-group <resource-group> \
  --name <public-ip-name> \
  --query "{IP:ipAddress,SKU:sku.name}" \
  -o table
```

---

# 14. Create NIC

Create a NIC in the application subnet:

```bash
az network nic create \
  --resource-group <resource-group> \
  --name <nic-name> \
  --vnet-name <vnet-name> \
  --subnet <app-subnet-name> \
  --public-ip-address <public-ip-name>
```

Verify:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  -o table
```

---

# 15. Understand the NIC

The relationship is:

```text
VM
 |
 v
NIC
 |
 +-- Private IP
 |
 +-- Public IP association
 |
 +-- Subnet
 |
 +-- NSG
```

Check the NIC IP configuration:

```bash
az network nic ip-config list \
  --resource-group <resource-group> \
  --nic-name <nic-name> \
  -o table
```

---

# 16. Verify Private IP

Run:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query "ipConfigurations[].privateIpAddress"
```

You should see an IP from:

```text
10.20.1.0/24
```

For example:

```text
10.20.1.x
```

---

# 17. Create NAT Gateway

If you already have a NAT Gateway in your lab, you can reuse it.

Create a Standard static public IP:

```bash
az network public-ip create \
  --resource-group <resource-group> \
  --name <nat-public-ip-name> \
  --sku Standard \
  --allocation-method Static
```

Create NAT Gateway:

```bash
az network nat gateway create \
  --resource-group <resource-group> \
  --name <nat-gateway-name> \
  --public-ip-addresses <nat-public-ip-name> \
  --idle-timeout 10
```

Verify:

```bash
az network nat gateway show \
  --resource-group <resource-group> \
  --name <nat-gateway-name> \
  -o table
```

---

# 18. Associate NAT Gateway with App Subnet

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <app-subnet-name> \
  --nat-gateway <nat-gateway-name>
```

Verify:

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <app-subnet-name> \
  --query natGateway.id
```

Architecture:

```text
Private VM
    |
    v
App Subnet
    |
    v
NAT Gateway
    |
    v
Static Public IP
    |
    v
Internet
```

---

# 19. Important Lab Observation

If the VM already has a public IP, that is not a good demonstration of a private backend.

For the NAT Gateway concept, the important production pattern is:

```text
Private VM
   |
   v
NAT Gateway
   |
   v
Internet
```

not:

```text
VM Public IP
   |
   v
Internet
```

The purpose of NAT Gateway is to provide controlled outbound connectivity for private resources.

---

# 20. Test DNS from the VM

If you have a Linux VM in the application subnet:

```bash
nslookup google.com
```

Then:

```bash
dig google.com
```

Then:

```bash
curl -I https://example.com
```

This verifies:

```text
DNS
+
Outbound connectivity
```

---

# 21. Check Outbound Public IP

From the Linux VM:

```bash
curl https://api.ipify.org
```

or:

```bash
curl https://ifconfig.me
```

If NAT Gateway is being used as the outbound path, the observed public IP should correspond to the NAT public IP.

This is a very useful production troubleshooting test.

---

# 22. Create Storage Account for Private Endpoint Test

If you already have a Storage Account, reuse it.

Otherwise:

```bash
az storage account create \
  --resource-group <resource-group> \
  --name <storage-account> \
  --location <location> \
  --sku Standard_LRS
```

Verify:

```bash
az storage account show \
  --resource-group <resource-group> \
  --name <storage-account> \
  -o table
```

---

# 23. Get Storage Account Resource ID

Run:

```bash
az storage account show \
  --resource-group <resource-group> \
  --name <storage-account> \
  --query id \
  -o tsv
```

Save the output mentally as:

```text
Storage Account Resource ID
```

You will use it for the Private Endpoint.

---

# 24. Create Private DNS Zone for Blob Storage

Create:

```bash
az network private-dns zone create \
  --resource-group <resource-group> \
  --name privatelink.blob.core.windows.net
```

Verify:

```bash
az network private-dns zone list \
  --resource-group <resource-group> \
  -o table
```

---

# 25. Link Private DNS Zone to VNet

Create a VNet link:

```bash
az network private-dns link vnet create \
  --resource-group <resource-group> \
  --zone-name privatelink.blob.core.windows.net \
  --name <private-dns-link-name> \
  --virtual-network <vnet-resource-id> \
  --registration-enabled false
```

Verify:

```bash
az network private-dns link vnet list \
  --resource-group <resource-group> \
  --zone-name privatelink.blob.core.windows.net \
  -o table
```

Architecture:

```text
Private DNS Zone
       |
       v
VNet Link
       |
       v
Application VNet
```

---

# 26. Create Private Endpoint

Create the Private Endpoint for Blob:

```bash
az network private-endpoint create \
  --resource-group <resource-group> \
  --name <private-endpoint-name> \
  --vnet-name <vnet-name> \
  --subnet <private-endpoint-subnet-name> \
  --private-connection-resource-id <storage-account-resource-id> \
  --group-id blob \
  --connection-name <private-connection-name>
```

Verify:

```bash
az network private-endpoint show \
  --resource-group <resource-group> \
  --name <private-endpoint-name> \
  -o table
```

---

# 27. Create Private DNS Zone Group

Associate the Private Endpoint with the Private DNS zone:

```bash
az network private-endpoint dns-zone-group create \
  --resource-group <resource-group> \
  --endpoint-name <private-endpoint-name> \
  --name <dns-zone-group-name> \
  --private-dns-zone privatelink.blob.core.windows.net \
  --zone-name blob
```

Verify:

```bash
az network private-endpoint dns-zone-group list \
  --resource-group <resource-group> \
  --endpoint-name <private-endpoint-name> \
  -o table
```

---

# 28. Check Private Endpoint IP

Run:

```bash
az network private-endpoint show \
  --resource-group <resource-group> \
  --name <private-endpoint-name> \
  --query "customDnsConfigs"
```

You can also inspect the Private Endpoint NIC:

```bash
az network private-endpoint show \
  --resource-group <resource-group> \
  --name <private-endpoint-name> \
  --query "networkInterfaces"
```

The Private Endpoint should have a private IP from the Private Endpoint subnet.

---

# 29. Test Private DNS Resolution

From a VM that uses the VNet DNS path:

```bash
nslookup <storage-account>.blob.core.windows.net
```

The expected design is:

```text
Storage hostname
       |
       v
Private DNS
       |
       v
Private IP
       |
       v
Private Endpoint
       |
       v
Storage
```

The exact DNS response may show aliases/CNAME chains before the private IP.

---

# 30. Test with Dig

Run:

```bash
dig <storage-account>.blob.core.windows.net
```

Look for the final resolved address.

The important thing is:

```text
Does the hostname resolve through the intended private DNS path?
```

---

# 31. Check Private DNS Records

List records:

```bash
az network private-dns record-set list \
  --resource-group <resource-group> \
  --zone-name privatelink.blob.core.windows.net \
  -o table
```

You should see the relevant record created through the Private Endpoint DNS integration.

---

# 32. Final Lab Architecture

After completing the lab, your logical architecture should look like:

```text
                         INTERNET
                            |
                            v
                       Public DNS
                            |
                            v
                         Public IP
                            |
                            v
                         NIC
                            |
                      Private IP
                            |
                            v
                    Application Subnet
                            |
              +-------------+-------------+
              |                           |
             NSG                     Route Table
              |                           |
              +-------------+-------------+
                            |
                            v
                           VNet
                            |
                +-----------+-----------+
                |                       |
                v                       v
           NAT Gateway          Private Endpoint
                |                       |
                v                       v
            Public IP              Private DNS
                |                       |
                v                       v
             Internet               Azure Storage
```

---

# 33. Verify All Components

### VNet

```bash
az network vnet list -o table
```

### Subnets

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

### NSG

```bash
az network nsg list -o table
```

### Route Table

```bash
az network route-table list -o table
```

### NAT Gateway

```bash
az network nat gateway list -o table
```

### Private Endpoint

```bash
az network private-endpoint list -o table
```

### Private DNS

```bash
az network private-dns zone list -o table
```

### Public IPs

```bash
az network public-ip list -o table
```

---

# 34. Troubleshooting Exercise 1 — DNS

### Problem

Your VM cannot resolve the Storage hostname.

Check:

```bash
nslookup <storage-account>.blob.core.windows.net
```

Then verify:

```text
Private DNS Zone
        |
        v
VNet Link
        |
        v
DNS Configuration
        |
        v
Private Endpoint DNS Zone Group
```

---

# 35. Troubleshooting Exercise 2 — No Internet

### Problem

Private VM cannot access:

```text
https://example.com
```

Check:

```text
DNS
 ↓
Route
 ↓
NSG
 ↓
NAT Gateway
 ↓
Public IP
 ↓
Internet
```

Useful commands:

```bash
nslookup example.com
```

```bash
curl -I https://example.com
```

```bash
curl https://api.ipify.org
```

---

# 36. Troubleshooting Exercise 3 — Private Endpoint

### Problem

Private Endpoint exists, but application resolves Storage publicly.

Check:

```text
1. Private DNS zone
2. VNet link
3. DNS zone group
4. VNet DNS configuration
5. nslookup/dig result
```

Expected:

```text
Storage hostname
       |
       v
Private IP
       |
       v
Private Endpoint
```

---

# 37. Troubleshooting Exercise 4 — NSG

### Problem

Application cannot connect to a backend port.

Check:

```text
Source
Destination
Port
Protocol
Priority
Direction
```

Then verify whether another NSG exists at:

```text
Subnet
NIC
```

---

# 38. Troubleshooting Exercise 5 — Routing

### Problem

Traffic is reaching the wrong network path.

Check:

```text
System Routes
Custom Routes
UDRs
VNet Peering
Virtual Appliance
Firewall
```

Remember:

```text
NSG = Allow / Deny

Route = Traffic Path
```

---

# 39. Interview Question — Explain This Lab

### Question

Can you explain a networking lab you worked on in Azure?

### Answer

> I created a VNet with separate application and private endpoint subnets, associated an NSG and route table with the application subnet, and configured NAT Gateway for outbound connectivity. I also configured a Storage Private Endpoint with Private DNS and verified that the Storage hostname resolved through the private endpoint path.

---

# 40. Interview Question — How Did You Test NAT Gateway?

### Answer

> I placed the workload in a subnet associated with the NAT Gateway and tested outbound connectivity from the VM. I used `curl` against an external IP-check service to verify that the outbound source IP matched the NAT Gateway public IP.

---

# 41. Interview Question — How Did You Validate Private Endpoint?

### Answer

> I checked that the Private Endpoint had a private IP and that the Private DNS zone was linked to the VNet. Then I used `nslookup` or `dig` for the Azure service hostname and verified that the resolution followed the private endpoint path.

---

# 42. Interview Question — What Did You Learn from the Lab?

### Answer

> The main thing I learned was how Azure networking components work together instead of independently. DNS resolves the destination, routing determines the path, NSGs control traffic, NAT handles outbound connectivity, and Private Endpoint with Private DNS provides private access to Azure services.

---

# 43. Production Mapping

What you practiced in this lab maps directly to production:

```text
Lab Component
       |
       v
Production Usage
```

```text
VNet
 ↓
Application network

Subnet
 ↓
Workload segmentation

NSG
 ↓
Network security

Route Table
 ↓
Traffic routing

NAT Gateway
 ↓
Controlled outbound access

Private Endpoint
 ↓
Private PaaS connectivity

Private DNS
 ↓
Private service name resolution
```

---

# 44. Final Day-02 Architecture

Remember this:

```text
                              INTERNET
                                  |
                                  v
                             PUBLIC DNS
                                  |
                                  v
                         APPLICATION GATEWAY
                               + WAF
                                  |
                                  v
                            PRIVATE APP
                                  |
                    +-------------+-------------+
                    |                           |
                    v                           v
              NAT GATEWAY                PRIVATE ENDPOINT
                    |                           |
                    v                           v
                INTERNET                  PRIVATE DNS
                                                |
                                                v
                                           AZURE PaaS
```

Enterprise version:

```text
                         HUB VNET
                            |
          +-----------------+-----------------+
          |                 |                 |
       Firewall            DNS          VPN / ExpressRoute
          |
     +----+----+
     |         |
     v         v
   PROD       DEV
  SPOKE      SPOKE
```

---

# 45. Day-02 Final Mental Model

Don't memorize every command.

Remember the traffic journey:

```text
NAME
 |
 v
DNS
 |
 v
IP
 |
 v
ROUTE
 |
 v
NSG / FIREWALL
 |
 v
PORT
 |
 v
APPLICATION
```

For private Azure services:

```text
SERVICE HOSTNAME
       |
       v
PRIVATE DNS
       |
       v
PRIVATE IP
       |
       v
PRIVATE ENDPOINT
       |
       v
AZURE SERVICE
```

For outbound internet:

```text
PRIVATE RESOURCE
       |
       v
NAT GATEWAY
       |
       v
PUBLIC IP
       |
       v
INTERNET
```

### Final interview line

> **I troubleshoot Azure networking layer by layer — DNS, IP, routing, NSG/firewall, port and application. For private Azure services, I additionally verify Private Endpoint and Private DNS, and for private outbound traffic I verify the NAT Gateway path.**
