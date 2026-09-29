# Azure NAT Gateway

## 1. What is Azure NAT Gateway?

Azure NAT Gateway provides **outbound internet connectivity** for resources inside an Azure Virtual Network.

NAT stands for:

```text
Network Address Translation
```

The important point is:

> NAT Gateway allows private resources to initiate outbound connections to the internet without giving those resources their own public IP addresses.

Example:

```text
Private VM
10.0.2.10
    |
    v
NAT Gateway
Public IP
    |
    v
Internet
```

The VM remains private.

---

# 2. Real-World Example

Suppose we have an application VM:

```text
VM
Private IP: 10.0.2.10
Public IP: None
```

The application needs to download:

```text
apt packages
Docker images
OS updates
External API data
```

We don't want to assign a public IP directly to the VM.

So we use:

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

The VM can initiate outbound internet connections while remaining without a public IP.

---

# 3. Why Do We Need NAT Gateway?

Imagine we have 50 backend VMs:

```text
Backend Subnet
|
+-- VM1  Private IP
+-- VM2  Private IP
+-- VM3  Private IP
+-- VM4  Private IP
+-- ...
+-- VM50 Private IP
```

We don't want to assign 50 public IP addresses.

Instead:

```text
                    NAT Gateway
                        |
                    Public IP
                        |
        +---------------+---------------+
        |               |               |
       VM1             VM2             VM3
        |               |               |
        +---------------+---------------+
                        |
                     Internet
```

All these private resources can use the NAT Gateway for outbound internet access.

---

# 4. NAT Gateway Is Mainly for Outbound Traffic

This is one of the most important concepts.

NAT Gateway is designed for:

```text
Private Resource
      |
      v
Internet
```

It is **not a general inbound load balancer**.

For example:

```text
VM
 |
 | HTTPS request
 v
Internet
```

This is outbound traffic.

NAT Gateway handles this scenario.

---

# 5. Outbound vs Inbound

### Outbound

```text
VM
 |
 | Request
 v
Internet
```

Example:

```text
VM → Docker Hub
VM → Microsoft endpoint
VM → External API
VM → Linux package repository
```

NAT Gateway is useful here.

### Inbound

```text
Internet
 |
 | Request
 v
VM
```

NAT Gateway is not used as the service for exposing the VM to inbound internet traffic.

For inbound application traffic, you would typically use services such as:

```text
Application Gateway
Load Balancer
Azure Front Door
```

depending on the architecture.

---

# 6. NAT Gateway Architecture

Basic architecture:

```text
                         Internet
                            ^
                            |
                            |
                       Public IP
                            |
                            v
                     NAT Gateway
                            |
                            |
                    Backend Subnet
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
            VM1            VM2            VM3
         Private IP      Private IP      Private IP
```

The NAT Gateway is associated with the **subnet**.

You don't normally attach NAT Gateway directly to an individual VM.

---

# 7. NAT Gateway Works at Subnet Level

This is a key interview point.

Example:

```text
VNet
 |
 +-- Frontend Subnet
 |
 +-- Backend Subnet
       |
       +-- NAT Gateway
       |
       +-- VM1
       +-- VM2
       +-- VM3
```

Resources in the associated subnet can use the NAT Gateway for outbound connectivity.

This makes subnet-level architecture very useful.

---

# 8. Public IP Used by NAT Gateway

NAT Gateway requires a public IP or public IP prefix for outbound connectivity.

Example:

```text
Private VM
10.0.2.10
     |
     v
NAT Gateway
     |
     v
Public IP
20.x.x.x
     |
     v
Internet
```

The external service sees the NAT Gateway's public IP as the source address for the outbound connection.

---

# 9. Why Static Outbound IP Is Important

Suppose your company uses an external payment API.

The external provider says:

```text
Only allow requests from:
20.x.x.x
```

If your application has unpredictable outbound IP addresses, this becomes difficult.

With NAT Gateway:

```text
Application
    |
    v
NAT Gateway
    |
    v
Static Public IP
    |
    v
Payment API
```

The external team can allowlist the NAT Gateway public IP.

This is a very common production use case.

---

# 10. Example — Backend VM Without Public IP

Suppose:

```text
Backend VM
Private IP = 10.0.2.10
Public IP = None
```

The VM executes:

```bash
apt update
```

Traffic flows:

```text
VM
10.0.2.10
   |
   v
NAT Gateway
   |
   | Source translated
   v
Public IP
20.x.x.x
   |
   v
Internet
```

The external package repository sees the request coming from the NAT Gateway public IP.

---

# 11. NAT Translation

Suppose the VM sends:

```text
Source:
10.0.2.10

Destination:
8.8.8.8
```

NAT Gateway translates the outbound source address to its public IP.

Conceptually:

```text
Before NAT:

10.0.2.10
     |
     v
8.8.8.8
```

After NAT:

```text
20.x.x.x
     |
     v
8.8.8.8
```

The private VM IP is not directly exposed to the internet.

---

# 12. NAT Gateway and SNAT

SNAT means:

```text
Source Network Address Translation
```

NAT Gateway performs source address translation for outbound flows.

Example:

```text
Private Source
10.0.2.10:50000

        |
        v

NAT Gateway

        |
        v

Public Source
20.x.x.x:<translated-port>

        |
        v

Internet
```

This allows many private resources to share the outbound public IP/prefix.

---

# 13. NAT Gateway and Multiple VMs

Suppose:

```text
Backend Subnet
|
+-- VM1
+-- VM2
+-- VM3
+-- VM4
+-- VM5
```

NAT Gateway:

```text
               NAT Gateway
                    |
               Public IP
                    |
       +------------+------------+
       |            |            |
      VM1          VM2          VM3
       |            |            |
       +------------+------------+
```

All these VMs can use the NAT Gateway for outbound connectivity.

---

# 14. NAT Gateway Does Not Give Private IPs a Public IP

This is an important distinction.

Suppose:

```text
VM1
Private IP = 10.0.1.10
Public IP = None
```

After NAT Gateway is attached:

```text
VM1
Private IP = 10.0.1.10
Public IP = None
```

The VM still has no public IP.

The NAT Gateway provides the outbound public source.

---

# 15. NAT Gateway vs Public IP on VM

### Direct Public IP

```text
VM
 |
 +-- Private IP
 |
 +-- Public IP
```

The VM itself has internet-facing addressing.

### NAT Gateway

```text
VM
 |
 +-- Private IP
 |
 +-- No Public IP
       |
       v
   NAT Gateway
       |
       v
   Public IP
```

For private backend workloads, NAT Gateway is generally a cleaner outbound architecture.

---

# 16. NAT Gateway vs Load Balancer

This is a common interview question.

| Feature                | NAT Gateway           | Load Balancer                                                |
| ---------------------- | --------------------- | ------------------------------------------------------------ |
| Main purpose           | Outbound connectivity | Traffic distribution                                         |
| Primary direction      | Outbound              | Inbound and/or outbound features depending on configuration  |
| Layer                  | Network-level NAT     | L4                                                           |
| Backend load balancing | No                    | Yes                                                          |
| Static outbound IP     | Yes                   | Can provide outbound connectivity depending on configuration |
| Health probes          | No                    | Yes                                                          |
| Path routing           | No                    | No                                                           |

Memory:

```text
NAT Gateway
→ Outbound

Load Balancer
→ Load distribution
```

---

# 17. NAT Gateway vs Azure Firewall

Both can be involved in outbound traffic architecture, but their purposes are different.

### NAT Gateway

Main focus:

```text
Outbound connectivity
+
SNAT
+
Scalable outbound connections
+
Predictable public IP
```

### Azure Firewall

Main focus:

```text
Network security
+
Traffic filtering
+
Centralized inspection
+
Application/network rules
```

Example architecture:

```text
Private Subnet
      |
      v
Azure Firewall
      |
      v
NAT / Internet path
      |
      v
Internet
```

The exact design depends on the security and routing requirements.

---

# 18. NAT Gateway vs Service Endpoint

These solve completely different problems.

### NAT Gateway

Provides:

```text
Private Resource
      |
      v
Internet
```

### Service Endpoint

Provides private subnet access to supported Azure services through Azure's backbone.

Example:

```text
VM
 |
 v
Subnet
 |
 v
Service Endpoint
 |
 v
Azure Storage
```

Memory:

```text
NAT Gateway
→ Internet outbound

Service Endpoint
→ Supported Azure service access
```

---

# 19. NAT Gateway vs Private Endpoint

Again, different purposes.

### NAT Gateway

```text
Private VM
   |
   v
Internet
```

### Private Endpoint

```text
VM
 |
 v
Private IP
 |
 v
Azure PaaS Service
```

Example:

```text
VM
 |
 v
Private Endpoint
 |
 v
Azure Storage
```

Memory:

```text
NAT Gateway
→ Outbound Internet

Private Endpoint
→ Private access to Azure service
```

---

# 20. NAT Gateway vs Azure Bastion

### NAT Gateway

Used for:

```text
Outbound internet
```

### Azure Bastion

Used for:

```text
Secure administrative access
to VMs
```

Example:

```text
Admin
 |
 v
Azure Bastion
 |
 v
Private VM
```

While:

```text
Private VM
 |
 v
NAT Gateway
 |
 v
Internet
```

These services solve different problems.

---

# 21. NAT Gateway and AKS

NAT Gateway is also important in AKS architectures.

Suppose AKS nodes are private:

```text
AKS
 |
 +-- Node 1
 +-- Node 2
 +-- Node 3
```

Applications may need outbound access for:

```text
Container image pulls
External APIs
Package downloads
Azure services
Monitoring endpoints
```

Architecture:

```text
                Internet
                    ^
                    |
              NAT Gateway
                    |
              AKS Subnet
                    |
        +-----------+-----------+
        |           |           |
      Node1       Node2       Node3
```

This gives predictable outbound connectivity.

---

# 22. NAT Gateway in a Production Architecture

A common architecture:

```text
                         Internet
                            ^
                            |
                     NAT Gateway
                            |
                     Backend Subnet
                    /       |       \
                   /        |        \
                 VM1       VM2       VM3
                   |
                   |
              Private IP only
```

Frontend:

```text
Internet
   |
   v
Application Gateway + WAF
   |
   v
Private Backend
```

Backend outbound:

```text
Private Backend
   |
   v
NAT Gateway
   |
   v
Internet
```

This creates a useful separation:

```text
Inbound
Internet
   ↓
Application Gateway
   ↓
Private Backend

Outbound
Private Backend
   ↓
NAT Gateway
   ↓
Internet
```

---

# 23. NAT Gateway and NSG

NAT Gateway does not replace an NSG.

Example:

```text
Backend Subnet
      |
      +---- NSG
      |
      +---- NAT Gateway
      |
      +---- Private VMs
```

Think:

```text
NSG
→ Allow/Deny network traffic

NAT Gateway
→ Provide outbound NAT
```

Both can be used together.

---

# 24. NAT Gateway and Route Table

NAT Gateway is also different from a route table.

Route table:

```text
Where should traffic go?
```

NAT Gateway:

```text
How should outbound traffic get a public source address?
```

Example:

```text
VM
 |
 v
Routing decision
 |
 v
NAT Gateway
 |
 v
Internet
```

Do not confuse:

```text
UDR = Routing
NAT = Address Translation
```

---

# 25. Create Public IP for NAT Gateway

Example:

```bash
az network public-ip create \
  --resource-group <resource-group> \
  --name <nat-public-ip> \
  --sku Standard \
  --allocation-method Static
```

Check it:

```bash
az network public-ip show \
  --resource-group <resource-group> \
  --name <nat-public-ip> \
  --query ipAddress \
  -o tsv
```

---

# 26. Create NAT Gateway

Example:

```bash
az network nat gateway create \
  --resource-group <resource-group> \
  --name <nat-gateway-name> \
  --public-ip-addresses <nat-public-ip> \
  --idle-timeout 10
```

The idle timeout controls how long an idle flow can remain established before the NAT mapping is removed.

---

# 27. Associate NAT Gateway with a Subnet

This is the most important configuration step.

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --nat-gateway <nat-gateway-name>
```

Architecture becomes:

```text
VNet
 |
 +-- Backend Subnet
       |
       +-- NAT Gateway
       |
       +-- VM1
       +-- VM2
       +-- VM3
```

---

# 28. Verify NAT Gateway

List NAT Gateways:

```bash
az network nat gateway list -o table
```

Show NAT Gateway:

```bash
az network nat gateway show \
  --resource-group <resource-group> \
  --name <nat-gateway-name>
```

Check subnet configuration:

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name>
```

Look for the NAT Gateway association.

---

# 29. Test Outbound Connectivity

From a Linux VM without a public IP:

```bash
curl https://ifconfig.me
```

or:

```bash
curl https://api.ipify.org
```

The returned public IP should correspond to the outbound public IP configured for the NAT Gateway, subject to the actual network path and configuration.

Example:

```text
20.x.x.x
```

You can compare it with:

```bash
az network public-ip show \
  --resource-group <resource-group> \
  --name <nat-public-ip> \
  --query ipAddress \
  -o tsv
```

---

# 30. Hands-On Lab

Use an existing private subnet and VM where possible.

Do not create additional resources just for practice if you already have a suitable environment.

### Step 1 — Find existing VNets

```bash
az network vnet list -o table
```

### Step 2 — Find subnets

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

Choose a suitable private/backend subnet.

### Step 3 — Create a Standard static public IP

```bash
az network public-ip create \
  --resource-group <resource-group> \
  --name <nat-public-ip> \
  --sku Standard \
  --allocation-method Static
```

### Step 4 — Create NAT Gateway

```bash
az network nat gateway create \
  --resource-group <resource-group> \
  --name <nat-gateway-name> \
  --public-ip-addresses <nat-public-ip> \
  --idle-timeout 10
```

### Step 5 — Associate it with the subnet

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --nat-gateway <nat-gateway-name>
```

### Step 6 — Test from a private VM

```bash
curl https://ifconfig.me
```

Expected:

```text
NAT Gateway public IP
```

### Step 7 — Verify from Azure CLI

```bash
az network nat gateway show \
  --resource-group <resource-group> \
  --name <nat-gateway-name>
```

---

# 31. Troubleshooting NAT Gateway

Suppose:

```text
Private VM
    |
    v
NAT Gateway
    |
    v
Internet
```

But the VM cannot access the internet.

Troubleshoot in this order.

### Step 1 — Check NAT Gateway association

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name>
```

Verify the subnet has the NAT Gateway association.

---

### Step 2 — Check Public IP

```bash
az network public-ip show \
  --resource-group <resource-group> \
  --name <nat-public-ip>
```

Verify the public IP exists and is associated with the NAT Gateway.

---

### Step 3 — Check VM private IP

```bash
az vm list-ip-addresses \
  --resource-group <resource-group> \
  --name <vm-name> \
  -o table
```

Confirm the VM is using the expected subnet.

---

### Step 4 — Check NSG

Verify outbound traffic is not being blocked by network security rules.

Remember:

```text
NAT Gateway ≠ NSG
```

NAT provides translation.

NSG controls allowed/denied traffic.

---

### Step 5 — Check Route Table

If a custom route sends internet traffic somewhere else, investigate the routing path.

Example:

```text
Private VM
   |
   v
UDR
   |
   v
Firewall/NVA
```

The actual outbound path may therefore differ from a simple NAT-only architecture.

---

### Step 6 — Test DNS

First test connectivity by IP if appropriate, then test DNS.

Example:

```bash
nslookup google.com
```

or:

```bash
dig google.com
```

If DNS fails, the problem may not be NAT.

---

### Step 7 — Test HTTPS

```bash
curl -I https://example.com
```

If DNS works but HTTPS fails, investigate:

```text
NSG
Routing
Firewall
NAT configuration
Destination availability
```

---

# 32. Important Production Scenario

### Problem

A backend VM has:

```text
Private IP
No Public IP
```

The application needs to call a third-party API.

The third party requires:

```text
Source IP allowlisting
```

### Solution

Use:

```text
Backend VM
     |
     v
NAT Gateway
     |
     v
Static Public IP
     |
     v
Third-Party API
```

Then provide the NAT Gateway public IP to the third party for allowlisting.

This gives you predictable outbound identity without exposing the backend VM directly.

---

# 33. Scenario — VM Has No Public IP

### Interviewer:

Your VM does not have a public IP. How can it access the internet?

### Answer:

> I can associate a NAT Gateway with the VM's subnet. The VM keeps its private IP, while the NAT Gateway provides outbound source NAT using its public IP. This allows the VM to access the internet without directly exposing it with a public IP.

---

# 34. Scenario — Static Outbound IP

### Interviewer:

A third-party API only allows requests from a fixed IP. How would you design it?

### Answer:

> I would place the application resources in a subnet associated with a NAT Gateway and assign a static public IP to the NAT Gateway. The third party can then allowlist that public IP while the application resources remain private.

---

# 35. Scenario — Multiple VMs

### Interviewer:

You have 20 private VMs and all of them need outbound internet access. Would you assign 20 public IPs?

### Answer:

> No. I would normally associate a NAT Gateway with the subnet containing those VMs. The VMs can remain private and use the NAT Gateway for outbound connectivity, which also gives us predictable outbound public IPs.

---

# 36. Scenario — Backend Security

### Interviewer:

Why don't you simply give a public IP to the backend VM?

### Answer:

> If the backend doesn't need direct inbound internet access, I keep it private and use services such as Application Gateway for inbound application traffic and NAT Gateway for outbound internet access. This reduces direct exposure of the backend VM.

---

# 37. Scenario — NAT Gateway vs Load Balancer

### Interviewer:

Why would you use NAT Gateway instead of Load Balancer?

### Answer:

> NAT Gateway is primarily used for outbound internet connectivity and source NAT. Load Balancer is used to distribute traffic across backend instances. They solve different networking problems.

---

# 38. Scenario — NAT Gateway vs Azure Firewall

### Interviewer:

Why would you use Azure Firewall if NAT Gateway already provides outbound access?

### Answer:

> NAT Gateway provides outbound NAT and scalable outbound connectivity. Azure Firewall provides centralized traffic inspection and network security controls. If the requirement is security inspection and filtering, Firewall may be required in addition to NAT.

---

# 39. Scenario — NAT Gateway vs Private Endpoint

### Interviewer:

You need a private VM to access Azure Storage without using a public endpoint. Would you use NAT Gateway?

### Answer:

> Not if the requirement is private access to Azure Storage. I would use a Private Endpoint so the Storage service is reachable through a private IP in the VNet. NAT Gateway is mainly for outbound internet connectivity.

---

# 40. Scenario — NAT Gateway vs Service Endpoint

### Interviewer:

You want a subnet to access Azure Storage using Azure's backbone. What would you consider?

### Answer:

> I would consider a Service Endpoint if the requirement fits the supported service and network design. A Service Endpoint provides subnet-based access to supported Azure services, while NAT Gateway is mainly for outbound internet connectivity.

---

# 41. Senior-Level Scenario

### Interviewer:

Your backend VM has no public IP. It can access some Azure services but cannot access an external SaaS API. What would you investigate?

### Answer:

> First I would identify whether the destination is an Azure service or an external internet endpoint. For an external endpoint, I would check the subnet's NAT Gateway association, public IP, NSG, route table, DNS, and any Azure Firewall or NVA in the path. I would also test the connection from the VM to separate DNS and HTTPS endpoints.

---

# 42. Production Architecture Example

Consider an application with:

```text
Frontend
API
Database
```

Architecture:

```text
                         Internet
                            |
                            v
                Application Gateway + WAF
                            |
                            v
                       Web Subnet
                            |
                            v
                       API Subnet
                            |
                            +----------------+
                            |                |
                            v                v
                       API VM1            API VM2
                            |
                            v
                       Database
```

Outbound API traffic:

```text
API VM1
   |
API VM2
   |
   +----------+
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

The backend servers therefore have:

```text
Private IP
No direct Public IP
Controlled outbound path
```

---

# 43. NAT Gateway in Hub-Spoke Architecture

Example:

```text
                    Hub VNet
                       |
                Azure Firewall
                       |
        +--------------+--------------+
        |                             |
        v                             v
   Spoke VNet 1                  Spoke VNet 2
        |                             |
 Backend Subnet                 Backend Subnet
        |                             |
        v                             v
 NAT Gateway                    NAT Gateway
        |                             |
        v                             v
     Internet                     Internet
```

The exact architecture depends on whether outbound traffic is intended to go directly through NAT Gateway or through a centralized firewall/NVA.

When a centralized security appliance is used, routing must be designed accordingly.

---

# 44. NAT Gateway and Terraform

In a real DevOps environment, I would normally create NAT Gateway using Terraform.

Conceptually:

```text
Terraform
   |
   +-- Public IP
   |
   +-- NAT Gateway
   |
   +-- Subnet association
```

The important relationship is:

```text
Public IP
    ↓
NAT Gateway
    ↓
Subnet
    ↓
Private Resources
```

This makes the outbound networking configuration repeatable across environments such as:

```text
DEV
UAT
PROD
```

---

# 45. Common Mistakes

### Mistake 1

Thinking NAT Gateway gives the VM a public IP.

Incorrect:

```text
VM → Public IP
```

Correct:

```text
VM
 |
 v
NAT Gateway
 |
 v
Public IP
```

---

### Mistake 2

Thinking NAT Gateway handles inbound application traffic.

NAT Gateway is primarily for outbound connectivity.

---

### Mistake 3

Thinking NAT Gateway replaces NSG.

It doesn't.

```text
NSG → Traffic filtering
NAT → Source NAT / outbound connectivity
```

---

### Mistake 4

Thinking NAT Gateway replaces Application Gateway.

It doesn't.

```text
Application Gateway → Inbound web traffic / L7 routing

NAT Gateway → Outbound internet connectivity
```

---

### Mistake 5

Thinking Private Endpoint and NAT Gateway are the same.

They are different:

```text
Private Endpoint
→ Private access to Azure PaaS

NAT Gateway
→ Outbound internet
```

---

# 46. Quick Comparison

| Azure Service       | Main Purpose                              |
| ------------------- | ----------------------------------------- |
| NAT Gateway         | Outbound internet + SNAT                  |
| Load Balancer       | L4 traffic distribution                   |
| Application Gateway | L7 HTTP/HTTPS routing                     |
| WAF                 | Web request protection                    |
| Azure Firewall      | Centralized network security              |
| Private Endpoint    | Private access to Azure PaaS              |
| Service Endpoint    | Subnet access to supported Azure services |
| Bastion             | Private VM administration                 |

---

# 47. Interview Answer — What is NAT Gateway?

> Azure NAT Gateway provides outbound internet connectivity for private resources in a subnet. It performs source NAT using its public IP, so VMs can access external endpoints without having individual public IPs. I commonly use it for private backend workloads and predictable outbound IP requirements.

---

# 48. Interview Answer — How Does NAT Gateway Work?

> I associate the NAT Gateway with a subnet and attach a public IP or public IP prefix to it. Resources in that subnet initiate outbound connections through the NAT Gateway, which translates their private source address to the NAT public IP before sending traffic to the internet.

---

# 49. Interview Answer — Why Use NAT Gateway?

> I use NAT Gateway when private resources need outbound internet access without exposing them with individual public IPs. Another important use case is providing a predictable static outbound IP that external services can allowlist.

---

# 50. Final Mental Model

Remember this:

```text
                 INTERNET
                     ^
                     |
                Public IP
                     |
                     |
               NAT GATEWAY
                     |
                     |
              PRIVATE SUBNET
                     |
          +----------+----------+
          |          |          |
         VM1        VM2        VM3
       Private    Private    Private
```

### Traffic direction

```text
VM
 ↓
NAT Gateway
 ↓
Public IP
 ↓
Internet
```

### Three things to remember

```text
NAT Gateway
    ↓
Subnet-level
    ↓
Outbound connectivity
```

### One-line interview memory

> **NAT Gateway = private resources → outbound internet through a predictable public IP.**
