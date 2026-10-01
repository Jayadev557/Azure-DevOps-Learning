# Azure Networking — Scenario-Based Interview Questions

## 1. VNet vs Subnet

### Question

What is the difference between a VNet and a subnet?

### Answer

> A VNet is the overall private network in Azure, while a subnet is a smaller network segment inside that VNet. I use different subnets to separate workloads like Application Gateway, web, application, and private endpoints.

### Example

```text
VNet: 10.10.0.0/16
   |
   +-- Web Subnet
   +-- App Subnet
   +-- Private Endpoint Subnet
```

---

# 2. Application Cannot Reach Another VM

### Question

A VM in the application subnet cannot connect to another VM. How will you troubleshoot?

### Answer

> I first check whether both VMs can resolve each other's hostname or whether I know the destination private IP. Then I check the route, NSG rules, firewall rules, and destination port. Finally, I verify that the application is actually listening on that port.

### Troubleshooting flow

```text
VM
 |
 v
DNS / Destination IP
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
Application
```

---

# 3. VM Has No Internet Access

### Question

A private VM cannot access the internet. What will you check?

### Answer

> I check DNS resolution first, then the subnet route and NSG. If the VM is private, I also verify that the intended outbound path, such as NAT Gateway, is configured correctly.

### Architecture

```text
Private VM
    |
    v
Subnet
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

# 4. Why Not Give Every VM a Public IP?

### Question

Why don't you assign public IPs to all backend VMs?

### Answer

> Backend VMs normally don't need to be directly reachable from the internet. I keep them private and expose only the required frontend through Application Gateway or another controlled entry point. For outbound internet access, I can use NAT Gateway.

---

# 5. NSG vs Route Table

### Question

What is the difference between NSG and Route Table?

### Answer

> NSG controls whether network traffic is allowed or denied. A route table determines where the traffic should go next. So I think of NSG as traffic filtering and routing as traffic direction.

### Memory

```text
NSG
↓
Allow / Deny

Route Table
↓
Where should traffic go?
```

---

# 6. NSG vs Azure Firewall

### Question

When would you use Azure Firewall instead of only NSGs?

### Answer

> NSGs provide subnet or NIC-level traffic filtering, while Azure Firewall is designed for centralized network security and traffic inspection. In a hub-spoke architecture, I can use Azure Firewall as a central security control for traffic between networks or toward external destinations.

---

# 7. NSG Rule Not Working

### Question

You allowed port 8080 in an NSG, but the application is still unreachable. What do you check?

### Answer

> I verify the rule priority, source, destination, protocol and port. Then I check whether another NSG is also applied at the NIC or subnet level, and I verify the route, firewall and whether the application is actually listening on port 8080.

---

# 8. What Happens When Both Subnet and NIC Have NSGs?

### Question

A subnet has an NSG and the VM NIC also has an NSG. What happens?

### Answer

> Traffic has to satisfy the effective security rules from both levels. So when troubleshooting, I check both the subnet-level and NIC-level NSGs rather than assuming only one of them controls the traffic.

---

# 9. ASG vs NSG

### Question

What is an Application Security Group?

### Answer

> ASG is a logical way to group network interfaces based on application roles, such as Web, API, or DB. ASG itself doesn't allow or deny traffic; I use it with NSG rules to make those rules easier to manage.

### Memory

```text
ASG = Group
NSG = Filter
```

---

# 10. Public IP vs Private IP

### Question

When would you use public and private IPs?

### Answer

> Private IPs are used for communication inside private networks such as VNets. Public IPs are used when a resource or frontend needs public internet connectivity. In production, I generally keep backend resources on private IPs.

---

# 11. Application Gateway vs Load Balancer

### Question

What is the difference between Application Gateway and Azure Load Balancer?

### Answer

> Application Gateway works at Layer 7 and understands HTTP/HTTPS, so it can do host and path-based routing, TLS termination and WAF integration. Load Balancer works at Layer 4 and distributes TCP or UDP traffic.

### Memory

```text
Application Gateway → L7
Load Balancer       → L4
```

---

# 12. Application Gateway Returns 502

### Question

Your Application Gateway is returning 502. How do you troubleshoot?

### Answer

> I first check backend health and the health probe. Then I verify the backend port, HTTP settings, NSG rules, routing, and whether the backend application is actually listening on the configured port.

### Flow

```text
Client
  |
  v
Application Gateway
  |
  v
Health Probe
  |
  X
Backend unhealthy
  |
  v
502
```

---

# 13. Application Gateway Health Probe Fails

### Question

The backend VM is running, but Application Gateway shows it as unhealthy. Why?

### Answer

> The VM being up doesn't guarantee the application is healthy. I check the probe port, protocol, host/path settings, NSG rules and whether the application responds correctly to the probe request.

---

# 14. When Would You Use Internal Load Balancer?

### Question

Why would you use an internal Load Balancer?

### Answer

> I use an internal Load Balancer when backend services need load balancing through private IPs. For example, Application Gateway can receive external HTTPS traffic and forward requests to an internal load balancer in front of multiple application VMs.

### Example

```text
Internet
   |
   v
Application Gateway
   |
   v
Internal Load Balancer
   |
   +-- App VM1
   +-- App VM2
```

---

# 15. NAT Gateway vs Load Balancer

### Question

What is the difference between NAT Gateway and Load Balancer?

### Answer

> NAT Gateway mainly provides outbound internet connectivity with predictable public IPs for private resources. Load Balancer distributes incoming or internal TCP/UDP traffic across backend instances.

### Memory

```text
NAT Gateway → Outbound
Load Balancer → Traffic distribution
```

---

# 16. Vendor Requires Static Public IP

### Question

Your private application needs to call a third-party API, and the vendor requires a static source IP. What will you do?

### Answer

> I would keep the application servers private and use NAT Gateway with a static public IP. The vendor can allowlist that public IP while the backend servers remain without public IPs.

---

# 17. Private Endpoint vs Service Endpoint

### Question

What is the difference between Private Endpoint and Service Endpoint?

### Answer

> Service Endpoint allows a subnet to access supported Azure services through the Azure backbone while the service still uses its normal endpoint. Private Endpoint gives the service a private IP inside the VNet, so applications can access it privately.

### Memory

```text
Service Endpoint
→ Subnet-based private access path

Private Endpoint
→ Private IP
```

---

# 18. Private Endpoint Exists but Application Cannot Connect

### Question

You created a Private Endpoint for Storage, but the application cannot connect. What will you check?

### Answer

> I first check DNS resolution because the application still uses the Storage hostname. Then I verify the Private DNS zone, VNet link and DNS configuration. After confirming the hostname resolves to the private IP, I check NSG, routing and connectivity.

---

# 19. Private Endpoint Resolves to Public IP

### Question

The Private Endpoint exists, but `nslookup` returns a public IP. What could be wrong?

### Answer

> I would check whether the correct Private DNS zone exists and whether it is linked to the application's VNet. I would also verify the Private DNS zone integration with the Private Endpoint and the VNet DNS configuration.

### Flow

```text
Application
    |
    v
DNS Query
    |
    +---- Public IP
    |       |
    |       X
    |
    +---- Expected private IP
            |
            v
      Private Endpoint
```

---

# 20. Private DNS vs Public DNS

### Question

What is Private DNS used for?

### Answer

> Private DNS provides name resolution inside private networks. A common Azure use case is resolving a Private Endpoint hostname to its private IP so applications don't need to use public connectivity.

---

# 21. VNet Peering

### Question

What is VNet peering?

### Answer

> VNet peering connects two Azure VNets using Microsoft's private network. It allows resources in the VNets to communicate using private IPs, assuming routing and security rules allow the traffic.

---

# 22. VNet Peering Is Configured but Traffic Doesn't Work

### Question

Two VNets are peered, but the VMs cannot communicate. What will you check?

### Answer

> I check that both peering directions are configured correctly and that the address spaces don't overlap. Then I check NSGs, routes, firewalls and whether the destination service is listening on the required port.

---

# 23. Is VNet Peering Transitive?

### Question

You have:

```text
VNet A ↔ VNet B
VNet B ↔ VNet C
```

Can A automatically communicate with C?

### Answer

> No, VNet peering is not automatically transitive. If A needs to communicate with C, the architecture needs an appropriate connectivity design such as direct peering or a supported routing solution.

---

# 24. Hub-Spoke Architecture

### Question

Explain Hub-Spoke architecture with an example.

### Answer

> I use a central Hub VNet for shared networking services such as Azure Firewall, VPN or ExpressRoute connectivity and DNS. Production and development workloads are deployed in separate Spoke VNets and connect to the Hub as required.

### Diagram

```text
                  HUB
                   |
        +----------+----------+
        |                     |
        v                     v
     PROD                   DEV
    SPOKE                  SPOKE
```

---

# 25. Why Use Hub-Spoke?

### Question

What problem does Hub-Spoke solve?

### Answer

> It centralizes common network services and separates application environments. This makes security, connectivity and network management easier to standardize across multiple workloads.

---

# 26. Route Table / UDR Scenario

### Question

You want application traffic to pass through a network virtual appliance. How can you do it?

### Answer

> I can create a user-defined route with the required destination prefix and configure the next hop as a virtual appliance. Then I associate the route table with the relevant subnet.

### Example

```text
Application
    |
    v
Route Table
    |
    v
Virtual Appliance
    |
    v
Destination
```

---

# 27. Traffic Goes to the Wrong Destination

### Question

A VM is sending traffic through the wrong network path. What will you check?

### Answer

> I check the effective routes for the subnet or NIC and look for custom UDRs, system routes, peering routes and gateway routes. Then I verify whether a firewall or virtual appliance is intended to be the next hop.

---

# 28. DNS vs Route

### Question

What is the difference between DNS and routing?

### Answer

> DNS converts a hostname into an IP address. Routing determines where packets should go to reach that destination IP.

### Memory

```text
DNS:
Name → IP

Routing:
IP → Next Hop
```

---

# 29. VM Can Resolve DNS but Cannot Connect

### Question

`nslookup` works, but the application connection fails. What does that tell you?

### Answer

> It tells me DNS resolution is working, but it doesn't prove network connectivity. I would move to checking routes, NSGs, firewall rules, destination ports and the application.

---

# 30. DNS Resolution Completely Fails

### Question

A VM cannot resolve any hostname. How will you troubleshoot?

### Answer

> I check the VM's DNS configuration first, then test the configured DNS server using `nslookup` or `dig`. In Azure, I also check VNet DNS settings, custom DNS servers, Private DNS links and DNS forwarding if applicable.

---

# 31. Backend VM Has Public IP

### Question

You find that backend VMs have public IPs in production. What would you review?

### Answer

> I would first understand whether the public IP is actually required. If not, I would keep the backend private and provide controlled inbound access through the application frontend and controlled outbound access through NAT Gateway or another approved network path.

---

# 32. Internet → Database

### Question

How would you prevent direct internet access to a database?

### Answer

> I would keep the database private and avoid assigning a public IP where possible. I would allow database traffic only from the required application tier using network security controls and private connectivity.

---

# 33. Application Needs Key Vault

### Question

How would a private application access Key Vault?

### Answer

> If private connectivity is required, I would use a Key Vault Private Endpoint and configure the appropriate Private DNS integration. The application would resolve the Key Vault hostname to the private endpoint IP.

---

# 34. Application Needs Storage

### Question

How would you provide private access from a VM or AKS workload to Azure Storage?

### Answer

> I can use a Private Endpoint for Storage and configure Private DNS so the normal Storage hostname resolves to the private IP. I would then verify network and identity permissions separately.

---

# 35. Network Is Working but Application Still Fails

### Question

You verified DNS, routing, NSG and connectivity. The application still fails. What next?

### Answer

> I would move to the application layer. I check whether the service is running, listening on the expected port, accepting the correct protocol, and whether there are application, TLS or authentication errors.

### Important mental model

```text
DNS
 ↓
Network
 ↓
Security
 ↓
Port
 ↓
Application
```

---

# 36. Complete Production Troubleshooting Scenario

### Question

Users report that the application is unavailable.

How would you troubleshoot from end to end?

### Answer

> I start from the client side and check DNS resolution and the Application Gateway frontend. Then I check Application Gateway listener and backend health, followed by NSGs, routes, firewall rules and backend connectivity. Finally, I check the application logs and service health.

### Flow

```text
User
 |
 v
DNS
 |
 v
Application Gateway
 |
 v
Backend Health
 |
 v
NSG
 |
 v
Route
 |
 v
Firewall
 |
 v
Application
```

---

# 37. Application Gateway vs Front Door

### Question

When would you use Application Gateway versus Azure Front Door?

### Answer

> Application Gateway is a regional Layer 7 service used inside Azure application architectures, while Front Door is a global edge service designed for global HTTP/HTTPS traffic and application acceleration. The choice depends on the application's global and regional architecture.

---

# 38. Load Balancer vs Application Gateway vs Front Door

### Question

How do you decide between these services?

### Answer

> For Layer 4 TCP/UDP load balancing, I use Load Balancer. For regional Layer 7 HTTP/HTTPS routing, TLS termination and WAF integration, Application Gateway is suitable. For global HTTP/HTTPS entry and edge routing, Front Door is designed for that use case.

### Memory

```text
Load Balancer
→ L4

Application Gateway
→ Regional L7

Front Door
→ Global HTTP/HTTPS edge
```

---

# 39. Production Architecture Design Question

### Question

Design a secure Azure network for a web application.

### Answer

> I would create a VNet with separate subnets for the frontend, application tier and private endpoints. Public traffic would enter through Application Gateway with WAF, while backend workloads remain private. I would use NSGs for segmentation, Private Endpoints and Private DNS for Azure services, and NAT Gateway for controlled outbound connectivity.

### Diagram

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
                       Web / App Tier
                             |
             +---------------+---------------+
             |                               |
             v                               v
       Private Database                Private Endpoint
                                             |
                                             v
                                       Private DNS
                                             |
                                             v
                                        Azure PaaS
```

---

# 40. Five-Year-Level Scenario

### Question

Your company has multiple production and development VNets. How would you design networking?

### Answer

> I would consider a Hub-Spoke architecture with centralized services such as firewall, DNS and hybrid connectivity in the Hub. Workloads would remain isolated in Spokes, with controlled connectivity through peering and routing. Within each Spoke, I would use subnet segmentation, NSGs, Private Endpoints and appropriate outbound controls.

---

# 41. Scenario — Security Team Wants No Public IPs

### Question

Security says backend VMs cannot have public IPs, but they still need outbound internet access. What would you propose?

### Answer

> I would keep the VMs private and use NAT Gateway for outbound internet access. This provides a predictable public source IP without exposing the individual VMs directly to inbound internet traffic.

---

# 42. Scenario — Vendor Allowlisting

### Question

The vendor has allowlisted your NAT IP, but requests still fail. What do you check?

### Answer

> I verify that the application subnet is actually associated with the NAT Gateway and that the expected public IP is being used. Then I check DNS, NSGs, routes, firewall rules and the vendor-side allowlist.

---

# 43. Scenario — Private Endpoint DNS

### Question

Private Endpoint is healthy but application resolves the service publicly.

### Answer

> I would focus on DNS rather than the Private Endpoint itself. I check the Private DNS zone, the VNet link, the DNS zone integration and the VNet's DNS configuration, then test resolution again.

---

# 44. Scenario — NSG vs Application Problem

### Question

How do you know whether an issue is caused by NSG or the application?

### Answer

> I first verify connectivity to the destination port. If the network connection reaches the destination but the application returns an error, I move to application troubleshooting. If the connection itself is blocked or times out, I investigate NSG, routes and firewalls.

---

# 45. Scenario — Multiple Network Controls

### Question

A request is failing even though the NSG allows it. What else could block it?

### Answer

> I would check the route, Azure Firewall or another network virtual appliance, destination-side NSG, service-level network restrictions and whether the application is listening on the expected port. An NSG allow rule alone doesn't guarantee end-to-end connectivity.

---

# 46. Scenario — Network Change in Production

### Question

You need to modify an NSG rule in production. What would you do?

### Answer

> I would first understand the required traffic flow and verify the current rule set. Then I would make the change through the organization's approved IaC or change-management process, validate the impact in a lower environment where possible, and monitor the production traffic after deployment.

---

# 47. Scenario — Terraform Networking

### Question

How would you manage this network using Terraform?

### Answer

> I would create reusable modules for the VNet, subnets, NSGs, route tables, private endpoints and other network components. Environment-specific values would come through variables or tfvars, while the module structure remains reusable across development, UAT and production.

### Example

```text
Terraform
   |
   +-- VNet Module
   |
   +-- Subnet Module
   |
   +-- NSG Module
   |
   +-- Route Table Module
   |
   +-- Private Endpoint Module
```

---

# 48. Scenario — Dev/UAT/Prod

### Question

How would you maintain networking across Dev, UAT and Prod?

### Answer

> I would use reusable Terraform modules and separate environment configurations. Each environment can have its own CIDR ranges, VNets, subnets and security rules while following the same architectural standards.

```text
Terraform Modules
       |
       +-- DEV
       |
       +-- UAT
       |
       +-- PROD
```

---

# 49. Rapid-Fire Interview Revision

### Q: VNet?

> Private network boundary in Azure.

### Q: Subnet?

> Smaller network segment inside a VNet.

### Q: NSG?

> Controls allowed and denied network traffic.

### Q: ASG?

> Groups NICs by application role for easier NSG rules.

### Q: Route Table?

> Controls the next hop for network traffic.

### Q: VNet Peering?

> Private connectivity between VNets.

### Q: Load Balancer?

> Layer 4 TCP/UDP traffic distribution.

### Q: Application Gateway?

> Layer 7 HTTP/HTTPS routing with features such as TLS termination and WAF integration.

### Q: NAT Gateway?

> Predictable outbound internet connectivity for private resources.

### Q: Private Endpoint?

> Provides a private IP for private access to supported Azure services.

### Q: Private DNS?

> Resolves private service names to private IP addresses.

### Q: Service Endpoint?

> Provides subnet-based access to supported Azure services over the Azure backbone.

---

# 50. Most Important Interview Memory

Don't memorize 15 separate services.

Think about the traffic:

```text
                  USER
                   |
                   v
                  DNS
                   |
                   v
             ENTRY POINT
                   |
          +--------+--------+
          |                 |
          v                 v
     App Gateway        Load Balancer
          |                 |
          +--------+--------+
                   |
                   v
              APPLICATION
                   |
        +----------+----------+
        |                     |
        v                     v
     DATABASE            AZURE PaaS
                              |
                              v
                       Private Endpoint
                              |
                              v
                        Private DNS
```

For outbound traffic:

```text
APPLICATION
     |
     v
NAT GATEWAY
     |
     v
INTERNET
```

For enterprise connectivity:

```text
                    HUB
                     |
          +----------+----------+
          |                     |
       PROD                   DEV
      SPOKE                  SPOKE
```

---

# 51. Final Senior-Level Mental Model

When an interviewer gives you any Azure networking problem, think in this order:

```text
1. DNS
   ↓
2. Destination IP
   ↓
3. Route
   ↓
4. NSG
   ↓
5. Firewall / Security
   ↓
6. Port
   ↓
7. Application
```

And for Private Endpoint:

```text
Hostname
   ↓
Private DNS
   ↓
Private IP
   ↓
Private Endpoint
   ↓
Azure PaaS
```

And for outbound internet:

```text
Private Resource
   ↓
NAT Gateway
   ↓
Static Public IP
   ↓
Internet
```

### Final interview line

> **When troubleshooting Azure networking, I don't randomly change rules. I trace the traffic flow from DNS to destination IP, routing, security controls, port connectivity and finally the application.**
