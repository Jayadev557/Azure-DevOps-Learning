# Azure VNet

## 1. What is an Azure VNet?

Azure VNet (Virtual Network) is our private network inside Azure.

When we deploy applications in Azure, we normally place the resources inside a VNet so that we can control:

* IP addressing
* Network segmentation
* Traffic flow
* Security
* Communication between resources

### Simple Example

Suppose we are deploying an application:

```text
                    Azure VNet
                        |
          +-------------+-------------+
          |                           |
          v                           v
   Frontend Subnet              Backend Subnet
          |                           |
          v                           v
     Web Servers                 API Servers
```

The VNet is the overall network, while the subnets divide that network into smaller sections.

---

# 2. Easy Memory Trick

Remember:

```text
VNet    → Overall private network
Subnet  → Smaller network inside VNet
NSG     → Controls allowed/blocked traffic
```

Think of a VNet like a **company campus**.

```text
Company Campus = VNet

Building 1 = Frontend Subnet
Building 2 = Backend Subnet
Building 3 = Database Subnet
```

---

# 3. VNet Address Space

When creating a VNet, we define an IP address range using CIDR.

Example:

```text
10.0.0.0/16
```

This becomes the address space available for the VNet.

We can divide it into smaller subnet ranges.

Example:

```text
VNet
10.0.0.0/16
│
├── Frontend Subnet
│   10.0.1.0/24
│
├── Backend Subnet
│   10.0.2.0/24
│
└── Database Subnet
    10.0.3.0/24
```

The subnet ranges must be within the VNet address space and should not overlap.

---

# 4. Why Do We Need Subnets?

We normally don't put everything into one large subnet.

Instead, we separate workloads based on their role.

Example:

```text
VNet: 10.0.0.0/16

Frontend
10.0.1.0/24
    |
    +-- Web servers

Backend
10.0.2.0/24
    |
    +-- API servers

Database
10.0.3.0/24
    |
    +-- Database resources
```

This helps with:

* Security
* Traffic control
* Network organization
* Routing
* Isolation
* Production management

---

# 5. VNet and Subnet Relationship

The relationship is:

```text
VNet
 |
 +-- Subnet 1
 |
 +-- Subnet 2
 |
 +-- Subnet 3
 |
 +-- Subnet 4
```

A subnet cannot exist independently from a VNet.

The subnet belongs to a specific VNet.

---

# 6. Can Resources in Different VNets Communicate?

By default, separate VNets are isolated from each other.

Example:

```text
VNet-A                  VNet-B

10.0.0.0/16             10.1.0.0/16
    |                       |
    v                       v
   VM-A                    VM-B

       X  No direct communication
```

If communication is required, we can use technologies such as:

* VNet Peering
* VPN Gateway
* ExpressRoute

For Azure-to-Azure VNet communication, VNet Peering is a common option.

---

# 7. VNet Communication Between Subnets

Resources in different subnets of the same VNet can communicate through Azure networking, subject to applicable network security and routing rules.

Example:

```text
                 VNet
                  |
        +---------+---------+
        |                   |
        v                   v
 Frontend Subnet       Backend Subnet
        |                   |
      VM-01               VM-02
        |                   |
        +------ Traffic ---+
```

If VM-01 cannot reach VM-02, we would check things such as:

1. NSG rules
2. Route table
3. IP configuration
4. Application/service listening port
5. Host firewall
6. Network connectivity

---

# 8. VNet Does Not Automatically Mean Internet Access

A common mistake is thinking:

> "My VM is inside a VNet, so it should have internet access."

Not necessarily.

For example:

```text
VM
 |
 | Private IP
 v
Subnet
 |
 v
VNet
```

The VM may not have direct inbound internet connectivity.

For outbound connectivity, the design may use services such as:

* NAT Gateway
* Load Balancer outbound rules
* Other Azure networking mechanisms

For inbound connectivity, depending on the design, we may use:

* Public IP
* Load Balancer
* Application Gateway
* Front Door

---

# 9. Production VNet Example

Suppose we have an e-commerce application.

We can design:

```text
                 Internet
                    |
                    v
            Application Gateway
                  + WAF
                    |
                    v
              Frontend Subnet
                    |
                    v
               Backend Subnet
                    |
                    v
             Private Endpoint
                    |
                    v
             Azure SQL / Storage
```

Example VNet:

```text
VNet
10.10.0.0/16

├── AppGatewaySubnet
│   10.10.1.0/24
│
├── FrontendSubnet
│   10.10.2.0/24
│
├── BackendSubnet
│   10.10.3.0/24
│
└── PrivateEndpointSubnet
    10.10.4.0/24
```

The exact subnet design depends on the application's architecture and Azure service requirements.

---

# 10. Creating a VNet Using Azure CLI

First create a Resource Group:

```bash
az group create \
  --name <resource-group> \
  --location <location>
```

Create a VNet:

```bash
az network vnet create \
  --resource-group <resource-group> \
  --name <vnet-name> \
  --address-prefix 10.10.0.0/16
```

Check the VNet:

```bash
az network vnet show \
  --resource-group <resource-group> \
  --name <vnet-name> \
  -o table
```

---

# 11. Create a Subnet Inside the VNet

```bash
az network vnet subnet create \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --address-prefixes 10.10.1.0/24
```

List subnets:

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

---

# 12. Check VNet Address Space

```bash
az network vnet show \
  --resource-group <resource-group> \
  --name <vnet-name> \
  --query addressSpace.addressPrefixes \
  -o table
```

Example output:

```text
Result
------------
10.10.0.0/16
```

---

# 13. Check Subnet Address Space

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query addressPrefix \
  -o tsv
```

Example:

```text
10.10.1.0/24
```

---

# 14. VNet Resource ID

Every VNet has a unique Azure Resource ID.

Get it using:

```bash
az network vnet show \
  --resource-group <resource-group> \
  --name <vnet-name> \
  --query id \
  -o tsv
```

The structure looks like:

```text
/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Network/virtualNetworks/<vnet-name>
```

---

# 15. Production Troubleshooting Scenario

### Problem

A backend VM cannot communicate with another VM inside the same VNet.

### My troubleshooting flow

First I verify that both VMs are actually connected to the expected VNet and subnets.

```bash
az vm list \
  --resource-group <resource-group> \
  -d \
  -o table
```

Then I check their NIC and private IP configuration.

```bash
az vm nic list \
  --resource-group <resource-group> \
  --vm-name <vm-name> \
  -o table
```

Then I check:

```text
1. NSG rules
2. Route table
3. Private IP addresses
4. Application port
5. Host firewall
6. Network connectivity
```

The important point is:

> Being in the same VNet does not guarantee that the application port is reachable.

Security rules, routes, and the application itself can still block the traffic.

---

# 16. Common VNet Mistakes

### Mistake 1 — Overlapping address spaces

Example:

```text
VNet-A
10.0.0.0/16

VNet-B
10.0.0.0/16
```

This can create problems when connecting the VNets.

---

### Mistake 2 — Poor subnet planning

Using the entire VNet for one subnet can make future segmentation difficult.

---

### Mistake 3 — Wrong subnet range

Example:

```text
VNet:
10.10.0.0/16

Subnet:
10.20.1.0/24
```

The subnet range is outside the VNet address space.

---

### Mistake 4 — Assuming VNet provides security by itself

A VNet provides network isolation, but traffic control normally requires additional controls such as NSGs, route controls, firewalls, and private connectivity patterns depending on the architecture.

---

# 17. VNet vs Subnet — Interview Answer

### Interviewer:

**What is the difference between VNet and Subnet?**

### Answer:

> "VNet is the overall private network in Azure, and subnet is a smaller network segment inside that VNet. For example, I can create one VNet and separate frontend, backend, and database workloads into different subnets. This helps me manage traffic, security, and network architecture."

---

# 18. Scenario-Based Interview Answer

### Interviewer:

**Two VMs are in the same VNet but cannot communicate. What will you check?**

### Answer:

> "First I will verify their private IPs, VNet and subnet configuration. Then I will check NSG rules, route tables, and whether the required application port is listening. I will also check the VM's OS firewall and application health."

---

# 19. Real DevOps Mental Model

When I see this:

```text
VNet
 |
 +-- Frontend Subnet
 |
 +-- Backend Subnet
 |
 +-- Database Subnet
```

I should immediately think:

```text
VNet
 ↓
Subnet
 ↓
NIC / Private IP
 ↓
NSG
 ↓
Route
 ↓
Application
```

And when troubleshooting:

```text
Source
  ↓
IP
  ↓
Subnet
  ↓
NSG
  ↓
Route
  ↓
Destination
  ↓
Application Port
```

---

# Key Takeaways

```text
VNet       → Overall Azure private network
CIDR       → Defines the IP address range
Subnet     → Smaller network inside VNet
Private IP → Internal communication
Public IP  → Internet-facing connectivity
NSG        → Controls network traffic
UDR        → Controls custom routing
Peering    → Connects VNets
```

### One-line memory trick

> **VNet is the network, Subnet divides it, NSG controls it, and routing decides where traffic goes.**
