# Azure Subnet

## 1. What is a Subnet?

A subnet is a smaller network segment created inside an Azure VNet.

Instead of putting every workload into one network segment, we divide the VNet into multiple subnets based on application roles.

Example:

```text
VNet: 10.10.0.0/16
        |
        +-------------------+
        |                   |
        v                   v
 Frontend Subnet       Backend Subnet
 10.10.1.0/24          10.10.2.0/24
        |                   |
     Web App              API
```

### Easy memory

> **VNet = overall network, Subnet = smaller network inside the VNet.**

---

# 2. Why Do We Need Subnets?

In a production environment, we usually separate different workloads.

For example:

```text
                    VNet
                     |
       +-------------+-------------+
       |             |             |
       v             v             v
   Frontend       Backend       Database
    Subnet         Subnet         Subnet
       |             |             |
     Web VM        API VM       Database
```

This helps with:

* Network segmentation
* Security
* Traffic control
* Routing
* Application organization
* Service-specific requirements
* Troubleshooting

---

# 3. Example VNet and Subnet Design

Suppose our VNet is:

```text
10.10.0.0/16
```

We can divide it into:

```text
VNet: 10.10.0.0/16

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

The exact design depends on the application and Azure services being deployed.

---

# 4. Understanding CIDR

Subnet ranges are normally written using CIDR notation.

Example:

```text
10.10.2.0/24
```

Here:

```text
10.10.2.0  → Network address
/24        → Prefix length
```

A `/24` IPv4 subnet contains 256 addresses in total.

However, Azure reserves five IP addresses in each subnet.

Therefore:

```text
256 total
- 5 Azure-reserved
-----------------
251 usable
```

For most DevOps interviews, remember:

> **Azure reserves five IP addresses from every subnet.**

---

# 5. CIDR Size Examples

Common examples:

```text
/16 → 65,536 total addresses
/20 → 4,096 total addresses
/24 → 256 total addresses
/26 → 64 total addresses
/27 → 32 total addresses
/28 → 16 total addresses
```

For example:

```text
10.10.1.0/24
```

provides:

```text
10.10.1.0
through
10.10.1.255
```

Azure reserves five addresses from that range.

---

# 6. Subnet Must Belong to the VNet Address Space

Suppose the VNet is:

```text
10.10.0.0/16
```

This subnet is valid:

```text
10.10.5.0/24
```

because it falls within the VNet range.

This subnet is invalid:

```text
10.20.5.0/24
```

because it is outside the VNet address space.

---

# 7. Subnets Cannot Overlap

Example:

```text
VNet
10.10.0.0/16

Subnet-A
10.10.1.0/24

Subnet-B
10.10.1.0/24
```

This is an overlapping design and cannot be used as two separate subnets.

A better design is:

```text
Subnet-A
10.10.1.0/24

Subnet-B
10.10.2.0/24
```

---

# 8. Creating a Subnet Using Azure CLI

Assuming the VNet already exists:

```bash
az network vnet subnet create \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --address-prefixes 10.10.2.0/24
```

Check the subnet:

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  -o table
```

List all subnets:

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

---

# 9. Check Subnet Address Prefix

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query addressPrefix \
  -o tsv
```

Example output:

```text
10.10.2.0/24
```

---

# 10. Subnet and NSG

A very important production concept is that an NSG can be associated with a subnet.

Example:

```text
                 Backend Subnet
                       |
                       v
                      NSG
                       |
              +--------+--------+
              |                 |
              v                 v
             VM1               VM2
```

The NSG controls network traffic according to its rules.

Example rule:

```text
Source:      Frontend subnet
Destination: Backend subnet
Port:        8080
Protocol:    TCP
Action:      Allow
```

This can allow frontend servers to communicate with backend servers on the required port.

---

# 11. NSG at Subnet vs NIC Level

An NSG can be associated with:

* Subnet
* Network Interface

Conceptually:

```text
Subnet NSG
    |
    +---- VM1
    |
    +---- VM2
    |
    +---- VM3
```

or:

```text
VM
 |
NIC
 |
NSG
```

In production, subnet-level NSGs are commonly useful for applying a common security boundary to workloads in that subnet.

---

# 12. Service-Specific Subnets

Some Azure services require or recommend dedicated subnets.

For example:

```text
VNet
 |
 +-- Application Gateway Subnet
 |
 +-- Azure Bastion Subnet
 |
 +-- AKS Node Subnet
 |
 +-- Application Subnet
 |
 +-- Private Endpoint Subnet
```

The exact subnet requirements depend on the Azure service and architecture.

For example, Azure Bastion requires a dedicated subnet named:

```text
AzureBastionSubnet
```

When working with service-specific subnets, always check the service requirements before assigning unrelated resources or network security configurations.

---

# 13. Subnet Delegation

Some Azure services can use subnet delegation.

Delegation tells Azure that a particular subnet is intended for a specific service.

Conceptually:

```text
Subnet
   |
   +---- Delegated to specific Azure service
```

This is useful when a service needs control over certain network configuration within the subnet.

Do not assume every subnet needs delegation. It depends on the Azure service being deployed.

---

# 14. Production Example

Suppose we are deploying a three-tier application.

```text
                    Internet
                       |
                       v
                Application Gateway
                       |
                       v
               Frontend Subnet
                10.10.1.0/24
                       |
                       v
                Backend Subnet
                10.10.2.0/24
                       |
                       v
                Database Subnet
                10.10.3.0/24
```

Security can be designed like:

```text
Internet
   |
   X
Database Subnet

Frontend
   |
   | Allowed application traffic
   v
Backend
   |
   | Allowed database traffic
   v
Database
```

The objective is not simply to create subnets.

The objective is to create **logical network boundaries** and then control traffic between those boundaries.

---

# 15. Real DevOps Scenario

### Requirement

The application has:

* Web tier
* API tier
* Database tier

The team asks:

> "Can we put everything into one subnet?"

Technically, a simple environment may work that way, but for a production architecture we generally want segmentation.

I would design:

```text
VNet
 |
 +-- Web Subnet
 |
 +-- API Subnet
 |
 +-- Data Subnet
```

Then I would control traffic using:

```text
NSG
+
Route tables
+
Application Gateway / Load Balancer
+
Private connectivity
```

This gives us better control and makes troubleshooting easier.

---

# 16. Troubleshooting Scenario

### Problem

A VM was created successfully but cannot communicate with another VM.

Both VMs appear to be inside the same VNet.

I would check:

```text
1. Are both VMs in the expected VNet?
2. Are their subnets correct?
3. Are the subnet ranges valid?
4. Is an NSG associated with the subnet?
5. Is the NSG allowing the required port?
6. Is a route table associated?
7. Is the application listening on the expected port?
8. Is the OS firewall blocking traffic?
```

Useful command:

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  -o json
```

---

# 17. Production Design Example

Consider an Azure application:

```text
                         Internet
                            |
                            v
                  Application Gateway
                            |
                            v
                 +-------------------+
                 | Frontend Subnet   |
                 | 10.10.1.0/24      |
                 +-------------------+
                            |
                            v
                 +-------------------+
                 | Backend Subnet    |
                 | 10.10.2.0/24      |
                 +-------------------+
                            |
                            v
                 +-------------------+
                 | Private Endpoint  |
                 | 10.10.4.0/24      |
                 +-------------------+
                            |
                            v
                     Azure Storage
```

The important thing is that every subnet has a purpose.

---

# 18. Subnet vs VNet

| VNet                          | Subnet                          |
| ----------------------------- | ------------------------------- |
| Overall Azure private network | Smaller network segment         |
| Contains address space        | Uses part of VNet address space |
| Contains multiple subnets     | Belongs to one VNet             |
| Provides network boundary     | Provides network segmentation   |
| Example: `10.10.0.0/16`       | Example: `10.10.1.0/24`         |

### Easy interview line

> "A VNet is the overall private network, while subnets divide that network into logical segments for different application workloads."

---

# 19. Interview Scenario

### Interviewer:

**Why do you create different subnets for frontend and backend?**

### Answer:

> "I use separate subnets to create network segmentation. For example, frontend servers can be placed in one subnet and backend servers in another, and I can control communication between them using NSGs and routing. This gives better security and makes the production network easier to manage."

---

# 20. Interview Scenario

### Interviewer:

**Can two VMs in different subnets communicate?**

### Answer:

> "Yes. If they are in the same VNet, Azure provides connectivity between the subnets, but the actual traffic still depends on routing and security controls such as NSGs. So if communication fails, I would check the subnet configuration, NSG, route table, and application port."

---

# 21. Interview Scenario

### Interviewer:

**How do you plan subnet CIDR ranges?**

### Answer:

> "I first estimate the workload and future growth, then divide the VNet address space into non-overlapping subnet ranges. I keep separate ranges for different workloads such as application, backend, and private endpoints, and I leave enough address space for future scaling."

---

# 22. Hands-On Lab

Use the VNet created in the previous topic.

### Step 1 - List existing VNets

```bash
az network vnet list -o table
```

### Step 2 - List subnets

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

### Step 3 - Create a backend subnet

```bash
az network vnet subnet create \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name backend-subnet \
  --address-prefixes 10.10.2.0/24
```

### Step 4 - Create another subnet

```bash
az network vnet subnet create \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name frontend-subnet \
  --address-prefixes 10.10.3.0/24
```

### Step 5 - Verify

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

You should see something similar to:

```text
Name              AddressPrefix
----------------  -------------
backend-subnet    10.10.2.0/24
frontend-subnet   10.10.3.0/24
```

---

# 23. Final Mental Model

Remember the flow:

```text
VNet
 |
 +-------------------+
 |                   |
 v                   v
Frontend           Backend
Subnet             Subnet
 |                   |
 v                   v
Web                 API
 |
 +------ Traffic --->+
```

And security:

```text
Subnet
   |
   v
  NSG
   |
   v
Traffic allowed / denied
```

### One-line memory trick

> **VNet gives the network, Subnet divides it, and NSG controls the traffic entering or leaving that segment.**
