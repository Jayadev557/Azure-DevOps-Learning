# Azure Network Security Group - NSG

## 1. What is an NSG?

Network Security Group (NSG) is used to control network traffic to and from Azure resources.

In simple words:

> **NSG works like a traffic filter. It checks network traffic and decides whether to allow or deny it.**

Example:

```text
Internet
   |
   | TCP 443
   v
+----------------+
|      NSG       |
|                |
| Allow 443      |
| Deny 22        |
+----------------+
   |
   v
   VM
```

If traffic matches an allowed rule, it can continue.

If traffic matches a deny rule, it is blocked.

---

# 2. Easy Memory Trick

Remember:

```text
NSG = Network Traffic Filter
```

Think of it like a security guard at the entrance of a building.

```text
Traffic
   |
   v
Security Guard
   |
   +---- Allowed ----> Application
   |
   +---- Denied  ----> Blocked
```

---

# 3. Where Can We Associate an NSG?

An NSG can be associated with:

1. Subnet
2. Network Interface (NIC)

Example:

```text
VNet
 |
 +-- Backend Subnet
       |
       +-- NSG
       |
       +-- VM1
       +-- VM2
       +-- VM3
```

Or:

```text
VM
 |
NIC
 |
NSG
```

This gives us flexibility in how we apply network security.

---

# 4. Inbound and Outbound Traffic

NSG rules are divided into:

```text
Inbound
Outbound
```

### Inbound

Traffic coming toward the resource.

Example:

```text
Internet
   |
   | HTTPS 443
   v
NSG
   |
   v
Application
```

### Outbound

Traffic leaving the resource.

Example:

```text
Application
   |
   | HTTPS 443
   v
NSG
   |
   v
Internet
```

---

# 5. NSG Rule Components

An NSG rule can define things such as:

```text
Priority
Source
Source Port
Destination
Destination Port
Protocol
Action
```

Example:

```text
Priority:          100
Source:            Internet
Source Port:       *
Destination:       Backend subnet
Destination Port:  443
Protocol:          TCP
Action:             Allow
```

This means HTTPS traffic is allowed according to the rule.

---

# 6. Priority

NSG rules have priorities.

Example:

```text
Priority 100
Allow TCP 443

Priority 200
Deny TCP 443
```

The lower number has higher priority.

So:

```text
100 → evaluated before 200
```

If traffic matches the rule at priority 100, the later rule doesn't override it.

### Easy memory

> **Lower number = higher priority.**

---

# 7. Example: Allow HTTPS

Suppose a web server needs HTTPS access.

We can create an inbound rule:

```text
Source:      Internet
Destination: VM
Protocol:    TCP
Port:        443
Action:      Allow
```

Traffic:

```text
User
 |
 | HTTPS 443
 v
NSG
 |
 | Allowed
 v
Web Server
```

---

# 8. Example: Block SSH from Internet

Suppose SSH should not be directly exposed to the internet.

We can deny:

```text
Source:      Internet
Destination: VM
Protocol:    TCP
Port:        22
Action:      Deny
```

Traffic:

```text
Internet
   |
   | SSH 22
   v
  NSG
   |
   X
Blocked
```

In production, administration access may instead use controlled mechanisms such as Azure Bastion or private connectivity.

---

# 9. Default NSG Rules

Azure NSGs contain default rules.

Conceptually, they allow important Azure networking behavior and deny unsolicited inbound internet traffic.

For outbound traffic, default rules allow common outbound connectivity unless overridden by custom rules.

Example conceptual order:

```text
Custom rules
     ↓
Default rules
```

Custom rules are evaluated before the default rules.

---

# 10. NSG Rule Evaluation

Suppose we have:

```text
Priority 100 → Allow TCP 443
Priority 200 → Deny TCP 443
```

Traffic arrives on TCP 443.

Azure checks:

```text
Traffic
   |
   v
Priority 100
   |
   +-- Match → Allow
```

It does not continue to a later matching rule.

---

# 11. NSG and Subnet

Example:

```text
VNet
 |
 +-- Backend Subnet
       |
       +-- NSG
       |
       +-- VM1
       +-- VM2
       +-- VM3
```

If the NSG is associated with the subnet, it can apply to traffic involving resources in that subnet.

This is useful when several workloads need a common network security policy.

---

# 12. NSG and NIC

Example:

```text
VM
 |
NIC
 |
NSG
```

A NIC-level NSG lets us apply rules specifically to that network interface.

This can be useful when one workload needs additional network restrictions beyond the subnet-level policy.

---

# 13. What Happens When Both Subnet and NIC Have NSGs?

This is a very important interview topic.

Suppose:

```text
Subnet
  |
 NSG-A
  |
  v
 NIC
  |
 NSG-B
  |
  v
 VM
```

For traffic to be allowed, the effective security rules from both levels must permit the traffic.

For example:

```text
Subnet NSG → Allow 443
NIC NSG    → Deny 443
```

The traffic will be blocked.

So when troubleshooting, don't check only the subnet NSG.

Check both:

```text
Subnet NSG
+
NIC NSG
```

---

# 14. NSG Does Not Replace an Application Firewall

An NSG primarily works at the network traffic level.

For HTTP/HTTPS application-layer protection, services such as:

* Application Gateway WAF
* Azure Firewall
* Web Application Firewall policies

may be used depending on the architecture.

Example:

```text
Internet
   |
   v
Application Gateway
   |
  WAF
   |
   v
NSG
   |
   v
Backend
```

The exact architecture depends on the application's security requirements.

---

# 15. Create an NSG Using Azure CLI

Create an NSG:

```bash
az network nsg create \
  --resource-group <resource-group> \
  --name <nsg-name> \
  -o table
```

Example:

```text
<resource-group> → <resource-group>
<nsg-name>       → backend-nsg
```

---

# 16. List NSGs

```bash
az network nsg list \
  --resource-group <resource-group> \
  -o table
```

---

# 17. Show an NSG

```bash
az network nsg show \
  --resource-group <resource-group> \
  --name <nsg-name> \
  -o json
```

---

# 18. Create an Inbound Rule

Example: allow HTTPS.

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name allow-https \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes Internet \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443
```

---

# 19. Create an Inbound Deny Rule

Example:

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name deny-ssh \
  --priority 200 \
  --direction Inbound \
  --access Deny \
  --protocol Tcp \
  --source-address-prefixes Internet \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22
```

---

# 20. List NSG Rules

```bash
az network nsg rule list \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  -o table
```

This is one of the most useful commands during troubleshooting.

---

# 21. Associate NSG with a Subnet

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --network-security-group <nsg-name>
```

Verify:

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query networkSecurityGroup.id \
  -o tsv
```

---

# 22. Associate NSG with a NIC

First identify the NIC:

```bash
az network nic list \
  --resource-group <resource-group> \
  -o table
```

Then:

```bash
az network nic update \
  --resource-group <resource-group> \
  --name <nic-name> \
  --network-security-group <nsg-name>
```

Verify:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query networkSecurityGroup.id \
  -o tsv
```

---

# 23. Production Example

Suppose we have:

```text
                    Internet
                       |
                       v
                Application Gateway
                       |
                       v
              Frontend Subnet
                       |
                    NSG
                       |
                       v
               Backend Subnet
                       |
                    NSG
                       |
                       v
                Backend APIs
```

We may define:

```text
Internet → Frontend : TCP 443
Frontend → Backend  : TCP 8080
Internet → Backend  : Deny
Backend → Database  : TCP 1433
```

The objective is:

```text
Internet
   |
   | HTTPS
   v
Frontend
   |
   | API traffic
   v
Backend
   |
   | Database traffic
   v
Database
```

But:

```text
Internet
   |
   X
Backend
```

This reduces unnecessary exposure of backend services.

---

# 24. Real Production Troubleshooting Scenario

### Problem

Users report:

> "The application is not reachable."

You check the VM and it is running.

Instead of assuming the VM is the problem, follow the traffic path.

```text
User
 |
 v
Application Gateway
 |
 v
Frontend
 |
 v
Backend
```

Check:

```text
1. Is the frontend reachable?
2. Is the required port allowed?
3. Is an NSG blocking traffic?
4. Is there a subnet-level NSG?
5. Is there a NIC-level NSG?
6. Is the route correct?
7. Is the application listening?
8. Is the OS firewall blocking traffic?
```

---

# 25. Example: Backend API Not Reachable

Suppose:

```text
Frontend
   |
   | TCP 8080
   v
Backend
```

The backend application listens on:

```text
8080
```

But the NSG only allows:

```text
443
```

Traffic:

```text
Frontend
   |
   | TCP 8080
   v
NSG
   |
   X
Blocked
```

The solution is to create an appropriate rule allowing the required traffic, with a properly scoped source and destination.

---

# 26. NSG Troubleshooting Commands

List NSGs:

```bash
az network nsg list \
  --resource-group <resource-group> \
  -o table
```

List rules:

```bash
az network nsg rule list \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  -o table
```

Check subnet NSG:

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query networkSecurityGroup.id \
  -o tsv
```

Check NIC NSG:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query networkSecurityGroup.id \
  -o tsv
```

---

# 27. Effective Security Rules

When troubleshooting a VM, Azure can show the effective security rules applied to its NIC.

First identify the NIC:

```bash
az network nic list \
  --resource-group <resource-group> \
  -o table
```

Then:

```bash
az network nic list-effective-nsg \
  --resource-group <resource-group> \
  --name <nic-name>
```

This is useful because the effective result can include rules coming from different security configurations.

---

# 28. NSG vs Route Table

These two are commonly confused.

### NSG

Answers:

> **"Is this traffic allowed?"**

### Route Table

Answers:

> **"Where should this traffic go?"**

Example:

```text
Packet
  |
  +---- NSG ----> Allow / Deny
  |
  +---- Route --> Next hop
```

Memory trick:

```text
NSG   → Can traffic go?
UDR   → Where should traffic go?
```

---

# 29. NSG vs Azure Firewall

### NSG

Used for network traffic filtering at subnet/NIC scope.

### Azure Firewall

Provides centralized network security capabilities and can inspect/control traffic according to a broader firewall architecture.

Example:

```text
                Internet
                   |
                   v
             Azure Firewall
                   |
                   v
                  VNet
                   |
             +-----+-----+
             |           |
           Subnet      Subnet
             |           |
            NSG         NSG
```

They solve different parts of the security architecture and can be used together.

---

# 30. Interview Answer

### Interviewer:

**What is an NSG?**

### Answer:

> "NSG is a network traffic filtering mechanism in Azure. I can associate it with a subnet or NIC and define inbound and outbound rules based on source, destination, port, protocol, priority, and action. In production, I use it to control which workloads can communicate with each other."

---

# 31. Interview Scenario

### Interviewer:

**A VM is running but port 8080 is not reachable. What will you check?**

### Answer:

> "First I'll verify that the application is listening on port 8080. Then I'll check the subnet-level and NIC-level NSGs and their effective rules. After that I'll verify routing, the VM OS firewall, and whether the source is allowed to reach that destination port."

---

# 32. Interview Scenario

### Interviewer:

**What happens if subnet NSG allows traffic but NIC NSG denies it?**

### Answer:

> "The traffic will be blocked. When both subnet and NIC have NSGs, the effective security rules from both levels matter. So during troubleshooting I always check both levels instead of checking only the subnet NSG."

---

# 33. Hands-On Lab

Use the VNet and subnets from the previous topics.

### Step 1 - Create NSG

```bash
az network nsg create \
  --resource-group <resource-group> \
  --name backend-nsg \
  -o table
```

### Step 2 - List rules

```bash
az network nsg rule list \
  --resource-group <resource-group> \
  --nsg-name backend-nsg \
  -o table
```

### Step 3 - Create HTTPS rule

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name backend-nsg \
  --name allow-https \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes Internet \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443
```

### Step 4 - Verify

```bash
az network nsg rule list \
  --resource-group <resource-group> \
  --nsg-name backend-nsg \
  -o table
```

### Step 5 - Associate NSG with backend subnet

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name backend-subnet \
  --network-security-group backend-nsg
```

### Step 6 - Verify association

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name backend-subnet \
  --query networkSecurityGroup.id \
  -o tsv
```

---

# 34. Final Mental Model

When troubleshooting connectivity, think:

```text
Source
  |
  v
IP Address
  |
  v
Subnet
  |
  v
NSG
  |
  v
Route
  |
  v
Destination
  |
  v
Application Port
```

And remember:

```text
NSG
 |
 +-- Inbound rules
 |
 +-- Outbound rules
 |
 +-- Priority
 |
 +-- Allow
 |
 +-- Deny
```

### One-line memory trick

> **NSG answers: "Is this network traffic allowed or blocked?"**
