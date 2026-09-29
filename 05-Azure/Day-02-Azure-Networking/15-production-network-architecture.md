# Azure Production Network Architecture

## 1. Why Production Network Architecture Matters

In a real project, we don't create Azure networking components independently.

We connect them together to build a secure and scalable application platform.

A typical production architecture may contain:

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
Application Subnet
   |
   v
Database / Private Services
```

Around this architecture we may also use:

```text
VNet
Subnets
NSG
ASG
Route Tables
Private Endpoint
Private DNS
NAT Gateway
Load Balancer
VNet Peering
Azure Firewall
```

---

# 2. Production Network Mental Model

Think about the network in this order:

```text
VNet
  |
  +-- Subnet
        |
        +-- NIC
              |
              +-- NSG
              |
              +-- Private IP
```

Then traffic control:

```text
Traffic
   |
   +-- DNS → Where is the destination?
   |
   +-- Route → Where should traffic go?
   |
   +-- NSG → Is traffic allowed?
   |
   +-- Firewall → Should traffic pass security inspection?
```

For internet-facing applications:

```text
Internet
   |
   v
Application Gateway
   |
   v
Backend
```

For private Azure services:

```text
Application
   |
   v
Private DNS
   |
   v
Private Endpoint
   |
   v
Azure PaaS
```

---

# 3. Example Production Application

Suppose we have an e-commerce application.

It contains:

```text
Frontend
Backend API
Database
Storage
Key Vault
```

Users access:

```text
https://shop.example.com
```

A simple production design:

```text
                     INTERNET
                         |
                         v
                Public DNS
                         |
                         v
              Application Gateway
                    + WAF
                         |
                         v
                  Web Subnet
                         |
                         v
                  Application Tier
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Azure SQL               Azure Storage
             |                       |
             |                 Private Endpoint
             |                       |
             +-----------+-----------+
                         |
                    Private DNS
```

---

# 4. VNet Design

First create a VNet.

Example:

```text
Production VNet
10.10.0.0/16
```

This provides the overall private address space.

Inside it we create multiple subnets.

```text
Production VNet
10.10.0.0/16
       |
       +-- App Gateway Subnet
       |
       +-- Web Subnet
       |
       +-- App Subnet
       |
       +-- Private Endpoint Subnet
       |
       +-- Management Subnet
```

The actual CIDR ranges depend on the organization's IP planning.

---

# 5. Example Subnet Design

Example:

```text
VNet: 10.10.0.0/16

Application Gateway:
10.10.1.0/24

Web:
10.10.2.0/24

Application:
10.10.3.0/24

Private Endpoint:
10.10.4.0/24

Management:
10.10.5.0/24
```

The important concept is separation of workloads.

---

# 6. Why Separate Subnets?

Suppose everything is placed in one subnet:

```text
VNet
 |
 +-- App Gateway
 +-- Web
 +-- Backend
 +-- Private Endpoint
 +-- Management
```

It becomes harder to apply:

```text
Security
Routing
Access control
Troubleshooting
```

Instead:

```text
VNet
 |
 +-- Web subnet
 |
 +-- App subnet
 |
 +-- Private endpoint subnet
 |
 +-- Management subnet
```

provides better network organization.

---

# 7. NSG Design

NSGs control network traffic.

Example:

```text
Internet
   |
   v
Application Gateway
   |
   v
Web Tier
   |
   v
Application Tier
   |
   v
Database
```

We don't want:

```text
Internet → Database
```

Instead:

```text
Internet
   |
   X
Database
```

Only required application flows should be allowed.

---

# 8. Example NSG Rules

### Application Gateway

Allow:

```text
Internet
   |
   +-- TCP 443
```

### Web Tier

Allow traffic only from the Application Gateway.

```text
App Gateway
    |
    | Required application port
    v
Web Tier
```

### Application Tier

Allow traffic from the Web Tier.

```text
Web Tier
   |
   | Required API port
   v
Application Tier
```

### Database

Allow traffic only from the Application Tier.

```text
Application Tier
      |
      | Database port
      v
Database
```

---

# 9. Security Flow

The target design is:

```text
Internet
   |
   | HTTPS 443
   v
Application Gateway
   |
   | Application traffic
   v
Web Tier
   |
   | API traffic
   v
Application Tier
   |
   | Database traffic
   v
Database
```

Not:

```text
Internet
   |
   +------> Web
   |
   +------> App
   |
   +------> Database
```

This is the basic principle of network segmentation.

---

# 10. ASG in Production

Application Security Groups help organize NICs by application role.

Example:

```text
Web-ASG
   |
   +-- Web VM1
   +-- Web VM2

App-ASG
   |
   +-- API VM1
   +-- API VM2

DB-ASG
   |
   +-- DB VM
```

Then NSG rules can use these groups.

Example:

```text
Source: Web-ASG
Destination: App-ASG
Port: 8080
Action: Allow
```

And:

```text
Source: App-ASG
Destination: DB-ASG
Port: <database-port>
Action: Allow
```

Memory:

```text
ASG = Grouping
NSG = Filtering
```

---

# 11. Route Table / UDR

NSG decides whether traffic is allowed.

Route tables decide where traffic goes.

Example:

```text
Application Subnet
        |
        v
Route Table
        |
        v
Azure Firewall
        |
        v
Internet
```

A UDR can send traffic through a virtual appliance or firewall when required by the architecture.

---

# 12. Hub-Spoke Architecture

Large organizations commonly use a hub-and-spoke network design.

```text
                       HUB VNET
                          |
             +------------+------------+
             |            |            |
             v            v            v
         Firewall       DNS         VPN/ER
             |
     +-------+-------+
     |               |
     v               v
 Spoke VNet 1     Spoke VNet 2
     |               |
     v               v
Production        Development
```

The Hub contains shared network services.

Spokes contain workloads.

---

# 13. Hub VNet

A Hub VNet may contain:

```text
Azure Firewall
VPN Gateway
ExpressRoute Gateway
DNS infrastructure
Network monitoring
Shared services
```

Example:

```text
Hub VNet
   |
   +-- Azure Firewall
   |
   +-- VPN Gateway
   |
   +-- DNS
   |
   +-- Management
```

---

# 14. Spoke VNet

A Spoke VNet contains application workloads.

Example:

```text
Production Spoke
      |
      +-- Web
      +-- API
      +-- Data
```

Another:

```text
Development Spoke
      |
      +-- Web
      +-- API
      +-- Data
```

---

# 15. VNet Peering

The Hub and Spoke VNets can communicate using VNet peering.

```text
                 HUB
                  |
              Peering
                  |
          +-------+-------+
          |               |
          v               v
       PROD            DEV
```

Remember:

> VNet peering provides private connectivity between VNets, but security rules and routing still need to be designed correctly.

---

# 16. Application Gateway + WAF

For a web application:

```text
Internet
   |
   v
Application Gateway
   |
   +-- WAF
   |
   +-- TLS termination
   |
   +-- Routing
   |
   v
Backend
```

Application Gateway can route based on:

```text
Host
Path
```

Example:

```text
example.com/
        |
        v
Frontend

example.com/api/*
        |
        v
Backend API
```

---

# 17. Load Balancer

If the backend consists of multiple VMs:

```text
Application Gateway
        |
        v
Internal Load Balancer
        |
        +---- App VM1
        |
        +---- App VM2
        |
        +---- App VM3
```

The Load Balancer distributes Layer 4 traffic between healthy backend instances.

---

# 18. NAT Gateway

Private application VMs may need outbound internet access.

Instead of assigning public IPs to every VM:

```text
Private VM
10.10.3.10
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

This keeps the VM private while providing predictable outbound connectivity.

---

# 19. Why NAT Gateway Is Useful

Example:

Your application calls a third-party payment API.

The vendor asks:

```text
"Provide your public IP for allowlisting."
```

Instead of giving every VM a public IP, use:

```text
Application VM
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

Now the vendor can allowlist the NAT public IP.

---

# 20. Private Endpoint

For Azure PaaS services, private connectivity can be provided using Private Endpoint.

Example:

```text
Application
    |
    v
Private Endpoint
    |
    v
Azure Key Vault
```

Other common services:

```text
Storage
Key Vault
Azure SQL
ACR
Cosmos DB
```

---

# 21. Private DNS

Private Endpoint needs correct DNS resolution.

Example:

```text
Application
     |
     v
storage.blob.core.windows.net
     |
     v
Private DNS
     |
     v
10.10.4.10
     |
     v
Private Endpoint
     |
     v
Storage
```

Without correct DNS, an application may resolve the service hostname to the wrong endpoint.

---

# 22. Complete Production Architecture

Now combine everything.

```text
                              INTERNET
                                  |
                                  v
                             Public DNS
                                  |
                                  v
                       Application Gateway
                              + WAF
                                  |
                         +--------+--------+
                         |                 |
                         v                 v
                    Web Tier          Web Tier
                         |                 |
                         +--------+--------+
                                  |
                                  v
                         Internal LB
                                  |
                         +--------+--------+
                         |                 |
                         v                 v
                      API VM1           API VM2
                         |                 |
                         +--------+--------+
                                  |
                         Private Connectivity
                                  |
              +-------------------+-------------------+
              |                                       |
              v                                       v
        Azure SQL                              Private Endpoint
                                                    |
                                                    v
                                               Private DNS
                                                    |
                                                    v
                                               Key Vault /
                                               Storage / ACR
```

---

# 23. Add Hub-Spoke

For a larger enterprise:

```text
                              HUB VNET
                                  |
             +--------------------+--------------------+
             |                    |                    |
             v                    v                    v
        Azure Firewall          DNS              VPN/ExpressRoute
             |
             |
      +------+------+
      |             |
      v             v
 PROD SPOKE      DEV SPOKE
      |
      v
Application Architecture
```

---

# 24. Complete Traffic Flow

Suppose a user opens:

```text
https://shop.example.com
```

### Step 1 — DNS

```text
shop.example.com
        |
        v
Public DNS
        |
        v
Application Gateway Public IP
```

### Step 2 — Application Gateway

```text
Application Gateway
        |
        +-- WAF
        +-- TLS
        +-- Routing
```

### Step 3 — Backend

```text
Application Gateway
        |
        v
Web/Application Tier
```

### Step 4 — Database

```text
Application Tier
        |
        v
Private Database Connectivity
        |
        v
Azure SQL
```

### Step 5 — Storage

```text
Application
      |
      v
Private DNS
      |
      v
Private Endpoint
      |
      v
Azure Storage
```

---

# 25. Inbound vs Outbound Traffic

This distinction is important.

### Inbound

```text
Internet
   |
   v
Application Gateway
   |
   v
Application
```

### Outbound

```text
Application
   |
   v
NAT Gateway
   |
   v
Internet
```

Remember:

```text
Application Gateway → Incoming web traffic

NAT Gateway → Outgoing internet traffic
```

---

# 26. Production Network Security Layers

A production network can have multiple security layers.

```text
Internet
   |
   v
WAF
   |
   v
Application Gateway
   |
   v
NSG
   |
   v
Application
   |
   v
NSG
   |
   v
Private Database
```

For larger environments:

```text
Internet
   |
   v
WAF
   |
   v
Application Gateway
   |
   v
Azure Firewall / Network Controls
   |
   v
Application
```

The exact placement depends on the organization's architecture and traffic requirements.

---

# 27. Public vs Private Resources

A common production principle is:

```text
Public:
Application Gateway frontend
```

while:

```text
Private:
Backend VMs
Database
Key Vault
Storage
ACR
```

Example:

```text
                 PUBLIC
                   |
                   v
          Application Gateway
                   |
              PRIVATE
                   |
          +--------+--------+
          |        |        |
          v        v        v
        API      SQL     Storage
```

This reduces unnecessary public exposure.

---

# 28. Production IP Design

Before creating subnets, plan IP ranges.

Example:

```text
10.10.0.0/16
```

Then:

```text
10.10.1.0/24 → App Gateway
10.10.2.0/24 → Web
10.10.3.0/24 → App
10.10.4.0/24 → Private Endpoint
10.10.5.0/24 → Management
```

Leave sufficient address space for future scaling.

Avoid overlapping address ranges between VNets that need to communicate.

---

# 29. Network Troubleshooting Flow

Suppose the application cannot reach the database.

Don't immediately change everything.

Use this flow:

```text
Application
    |
    v
DNS Resolution
    |
    v
Destination IP
    |
    v
Route
    |
    v
NSG
    |
    v
Firewall
    |
    v
Port
    |
    v
Database
```

Check one layer at a time.

---

# 30. Example Troubleshooting

### Problem

API VM cannot reach Azure SQL.

First check DNS:

```bash
nslookup <sql-hostname>
```

Then test connectivity:

```bash
nc -vz <sql-hostname> <port>
```

or:

```bash
curl -v <endpoint>
```

Then check:

```text
Private Endpoint
Private DNS
NSG
Route
Firewall
SQL networking configuration
```

---

# 31. Problem: Backend Returns 502

Suppose:

```text
Client
  |
  v
Application Gateway
  |
  X
502 Bad Gateway
```

Check:

```text
1. Backend health
2. Backend port
3. Health probe
4. HTTP settings
5. NSG
6. Route
7. Application process
```

Example:

```text
Application Gateway
       |
       v
Health Probe
       |
       X
Backend not responding
       |
       v
502
```

---

# 32. Problem: Private Endpoint Not Working

Check:

```text
Private Endpoint
      |
      +-- NIC
      |
      +-- Private IP
      |
      +-- DNS Zone
      |
      +-- VNet Link
```

Then:

```bash
nslookup <service-hostname>
```

If it resolves to the wrong IP, investigate DNS first.

---

# 33. Problem: VM Has No Internet Access

Check:

```text
VM
 |
 v
Subnet
 |
 +-- Route Table
 |
 +-- NSG
 |
 +-- NAT Gateway
 |
 +-- DNS
```

For a private VM, verify that the subnet has the intended outbound path, such as NAT Gateway.

---

# 34. Problem: VM Can Reach Internet but Vendor Rejects Request

Suppose:

```text
VM
 |
 v
Third-party API
 |
 X
Access denied
```

The vendor may be allowlisting source IPs.

Check the outbound IP.

If NAT Gateway is used:

```text
VM
 |
 v
NAT Gateway
 |
 v
Static Public IP
 |
 v
Vendor API
```

Provide the NAT public IP for allowlisting.

---

# 35. Production Architecture Checklist

Before considering a network design ready, verify:

```text
[ ] VNet address space planned
[ ] Subnets separated by workload
[ ] No overlapping CIDRs
[ ] NSGs designed
[ ] ASGs used where useful
[ ] Routes/UDRs planned
[ ] Public exposure minimized
[ ] Application Gateway/WAF configured where required
[ ] Load Balancer configured where required
[ ] NAT Gateway configured for predictable outbound traffic
[ ] Private Endpoints configured for required PaaS services
[ ] Private DNS configured
[ ] VNet links configured
[ ] Hub-spoke connectivity planned where required
[ ] DNS resolution tested
[ ] Network connectivity tested
```

---

# 36. Azure CLI — Useful Verification Commands

### List VNets

```bash
az network vnet list -o table
```

### List subnets

```bash
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

### List NSGs

```bash
az network nsg list -o table
```

### List route tables

```bash
az network route-table list -o table
```

### List peerings

```bash
az network vnet peering list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  -o table
```

### List Private Endpoints

```bash
az network private-endpoint list -o table
```

### List NAT Gateways

```bash
az network nat gateway list -o table
```

### List Application Gateways

```bash
az network application-gateway list -o table
```

### List Load Balancers

```bash
az network lb list -o table
```

---

# 37. Production Scenario — Design an Azure Network

### Interviewer:

You need to deploy a three-tier application in Azure. How would you design the network?

### Answer:

> I would create a VNet with separate subnets for the application tiers and management components. I would expose only the required frontend through Application Gateway with WAF, keep backend and data services private, use NSGs and route controls for segmentation, and use Private Endpoints with Private DNS for Azure PaaS services.

---

# 38. Production Scenario — Secure Backend

### Interviewer:

How would you prevent direct internet access to backend VMs?

### Answer:

> I would keep the backend VMs private without public IPs and allow inbound traffic only from the required frontend or application tier using NSGs. For outbound internet access, I can use NAT Gateway instead of assigning public IPs to the backend VMs.

---

# 39. Production Scenario — Private Azure Services

### Interviewer:

How would your application access Key Vault and Storage securely?

### Answer:

> I would use Private Endpoints for the required Azure services and configure the corresponding Private DNS zones and VNet links. The application would continue using the normal Azure service hostname, which resolves to the private endpoint IP.

---

# 40. Production Scenario — Hub-Spoke

### Interviewer:

Why would you use Hub-Spoke architecture?

### Answer:

> I would use Hub-Spoke when multiple environments or workloads need shared network services. The Hub can provide services such as Azure Firewall, VPN or ExpressRoute connectivity, and centralized DNS, while production and development workloads remain isolated in separate Spoke VNets.

---

# 41. Production Scenario — Outbound Connectivity

### Interviewer:

Your private application servers need to call an external API, but the vendor only allows specific source IPs. What would you do?

### Answer:

> I would keep the application servers private and associate their subnet with a NAT Gateway using a static public IP. The vendor can then allowlist that NAT public IP while the application servers remain without public IPs.

---

# 42. Production Scenario — Network Troubleshooting

### Interviewer:

An application cannot connect to another service. How do you troubleshoot?

### Answer:

> I first verify DNS resolution and confirm the destination IP. Then I check the route, NSG, firewall rules, required port, and the application or service itself. For Private Endpoints, I also verify the Private DNS zone and VNet link.

---

# 43. Interview Answer — Explain Your Production Network

### Answer:

> In my production design, I use a VNet with separate subnets for different application tiers. Internet traffic enters through Application Gateway with WAF, while backend workloads remain private and are controlled using NSGs and routing. For Azure PaaS services, I use Private Endpoints with Private DNS, and NAT Gateway provides controlled outbound connectivity where required.

---

# 44. Interview Answer — Complete Traffic Flow

### Answer:

> A user request first resolves through DNS to the Application Gateway endpoint. Application Gateway handles HTTPS, WAF and routing, then sends traffic to the private backend tier. The backend accesses databases and Azure services through private connectivity, while Private DNS resolves Private Endpoint hostnames to private IPs.

---

# 45. Day-2 Network Mental Model

Remember this architecture:

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
                        WEB SUBNET
                             |
                             v
                    APPLICATION SUBNET
                             |
                +------------+------------+
                |                         |
                v                         v
          PRIVATE DATABASE        PRIVATE ENDPOINT
                                        |
                                        v
                                  PRIVATE DNS
                                        |
                                        v
                                  Azure PaaS
```

For outbound traffic:

```text
Private Application
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

For enterprise networking:

```text
                       HUB VNET
                          |
             +------------+------------+
             |            |            |
          Firewall       DNS      VPN/ExpressRoute
             |
       +-----+-----+
       |           |
       v           v
   PROD SPOKE   DEV SPOKE
```

---

# 46. Final Memory Trick

Remember the responsibility of each component:

```text
VNet
↓
Network boundary

Subnet
↓
Network segmentation

NSG
↓
Allow / Deny traffic

ASG
↓
Group application NICs

Route Table
↓
Choose traffic path

VNet Peering
↓
Connect VNets privately

Application Gateway
↓
Layer 7 web traffic

Load Balancer
↓
Layer 4 traffic

NAT Gateway
↓
Predictable outbound internet

Private Endpoint
↓
Private access to Azure PaaS

Private DNS
↓
Private hostname resolution

Azure Firewall
↓
Centralized network security and traffic inspection
```

### One-line production mental model

> **DNS finds the destination, routing chooses the path, NSG controls access, security services inspect traffic, and private endpoints keep Azure PaaS connectivity private.**
