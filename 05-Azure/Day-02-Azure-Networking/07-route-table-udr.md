# Azure Route Table and User Defined Routes (UDR)

## 1. What is Routing?

Routing decides **where network traffic should go**.

Simple example:

```text
Frontend VM
    |
    | Request
    v
Backend VM
```

Azure needs to know which network path should be used to reach the backend.

Think:

```text
IP Destination
      |
      v
Route Table
      |
      v
Next Hop
```

---

# 2. What is a Route Table?

An Azure route table contains routes that tell Azure how to forward network traffic.

Example:

```text
Route Table
     |
     +---- Destination: 10.10.2.0/24
     |
     +---- Next Hop: Virtual Network
```

A route table can be associated with a subnet.

```text
VNet
 |
 +---- Frontend Subnet
 |
 +---- Backend Subnet
          |
          +---- Route Table
```

---

# 3. Why Do We Need Route Tables?

Azure already provides system routes automatically.

But sometimes production architecture requires custom traffic paths.

For example:

```text
Application Subnet
       |
       v
Azure Firewall
       |
       v
Internet
```

Instead of allowing traffic to directly follow the default path, we can create a custom route that sends traffic through a firewall.

This is where **UDR** is useful.

---

# 4. What is UDR?

UDR means **User Defined Route**.

It is a custom route created by the user.

Example:

```text
Destination: 0.0.0.0/0
Next Hop: Virtual Appliance
Next Hop IP: 10.10.10.4
```

Meaning:

```text
All destinations
     |
     v
Azure Firewall / NVA
     |
     v
Next destination
```

---

# 5. System Route vs User Defined Route

Azure automatically creates **system routes**.

Example:

```text
VNet
10.10.0.0/16
```

Azure knows how to route traffic between subnets inside that VNet.

You can add custom routes when the default system routing is not enough.

```text
System Route
    ↓
Azure default routing

UDR
    ↓
Custom routing requirement
```

---

# 6. Common Route Types

A route contains:

```text
Address Prefix
Next Hop Type
Next Hop IP
```

Common next-hop types include:

* Virtual network
* Internet
* Virtual appliance
* Virtual network gateway
* None

---

# 7. Virtual Network Next Hop

Example:

```text
Destination:
10.10.2.0/24

Next Hop:
Virtual network
```

This tells Azure to route traffic through the VNet.

---

# 8. Internet Next Hop

Example:

```text
Destination:
0.0.0.0/0

Next Hop:
Internet
```

This represents a route toward the internet.

The actual ability to communicate still depends on other Azure networking and security configuration.

---

# 9. Virtual Appliance

A virtual appliance is a network appliance such as:

* Azure Firewall
* Network Virtual Appliance (NVA)

Example:

```text
Application Subnet
       |
       v
Route Table
       |
       | UDR
       v
Azure Firewall
       |
       v
Internet
```

The UDR can force traffic through the firewall.

---

# 10. Example: Hub and Spoke

A common enterprise architecture is:

```text
                  HUB VNET
              +-------------+
              | Firewall    |
              | VPN Gateway |
              +-------------+
                    |
          +---------+---------+
          |                   |
          v                   v
     Spoke VNet 1        Spoke VNet 2
      Application          Application
```

Traffic from a spoke can be routed through the hub firewall.

Example:

```text
Spoke VM
   |
   v
UDR
   |
   v
Hub Firewall
   |
   v
Destination
```

---

# 11. Route Table Association

A route table is normally associated with a subnet.

Example:

```text
VNet
 |
 +---- app-subnet
 |        |
 |        +---- Route Table
 |
 +---- db-subnet
```

The routes in the route table affect traffic from resources in the associated subnet.

---

# 12. Create a Route Table

Create a route table:

```bash
az network route-table create \
  --resource-group <resource-group> \
  --name <route-table-name> \
  --location <location>
```

Example:

```text
Route table:
rt-app
```

---

# 13. List Route Tables

```bash
az network route-table list \
  --resource-group <resource-group> \
  --output table
```

---

# 14. Show Route Table

```bash
az network route-table show \
  --resource-group <resource-group> \
  --name <route-table-name>
```

---

# 15. Create a User Defined Route

Example:

```bash
az network route-table route create \
  --resource-group <resource-group> \
  --route-table-name <route-table-name> \
  --name <route-name> \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address <firewall-private-ip>
```

This means:

```text
0.0.0.0/0
    |
    v
Virtual Appliance
    |
    v
Next destination
```

---

# 16. List Routes

```bash
az network route-table route list \
  --resource-group <resource-group> \
  --route-table-name <route-table-name> \
  --output table
```

---

# 17. Associate Route Table With Subnet

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --route-table <route-table-name>
```

Now:

```text
Subnet
   |
   v
Route Table
   |
   v
Custom Routes
```

---

# 18. Remove Route Table From Subnet

If required:

```bash
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --remove routeTable
```

---

# 19. Important Difference: NSG vs Route Table

This is a common interview question.

### NSG

NSG decides:

```text
ALLOW or DENY
```

Example:

```text
TCP 443 → Allow
TCP 22  → Deny
```

### Route Table

Route table decides:

```text
WHERE should traffic go?
```

Example:

```text
Internet traffic
      |
      v
Azure Firewall
```

Simple memory:

```text
NSG = Can traffic pass?

Route = Where should traffic go?
```

---

# 20. Example: NSG + Route Table

Suppose:

```text
Application VM
      |
      +---- NSG
      |
      +---- Route Table
```

Traffic first needs an allowed path.

The NSG determines whether traffic is allowed.

The routing system determines the destination path.

Think:

```text
Traffic
   |
   v
Routing
   |
   v
Destination
   |
   v
NSG / other security controls
```

The exact packet-processing behavior depends on the Azure networking path, so don't memorize it as "NSG always runs before routing."

The important distinction is:

```text
NSG  → Security decision
Route → Forwarding decision
```

---

# 21. Troubleshooting Scenario

### Problem

A VM has internet connectivity, but your organization wants all outbound traffic to pass through Azure Firewall.

Current flow:

```text
VM
 |
 v
Internet
```

Required flow:

```text
VM
 |
 v
UDR
 |
 v
Azure Firewall
 |
 v
Internet
```

Check:

```text
1. Is the route table created?
2. Is the UDR present?
3. Is the route table associated with the correct subnet?
4. Is the firewall reachable?
5. Is the next-hop IP correct?
6. Are NSG/firewall rules allowing the traffic?
```

---

# 22. Another Troubleshooting Scenario

### Problem

Two VMs cannot communicate.

You check the NSG and see that the required port is allowed.

But communication still fails.

Don't stop at NSG.

Check:

```text
VM
 ↓
NIC
 ↓
Subnet
 ↓
Route Table
 ↓
Next Hop
 ↓
Destination
```

A bad custom route can send traffic to the wrong destination.

---

# 23. Production Example

Imagine a company has:

```text
                    Internet
                       |
                       v
                Azure Firewall
                       |
                 Hub VNet
                       |
          +------------+------------+
          |                         |
          v                         v
     Spoke VNet 1              Spoke VNet 2
      Application               Application
```

Each spoke subnet can have a route table.

Example:

```text
Spoke Subnet
     |
     v
UDR
     |
     v
Firewall Private IP
     |
     v
Hub Firewall
```

This gives the network team a controlled traffic path.

---

# 24. Route Table vs Azure Firewall

They are not the same thing.

### Route Table

Controls the traffic path.

```text
Traffic
   |
   v
Route Table
   |
   v
Firewall
```

### Azure Firewall

Inspects and controls network traffic according to firewall rules.

```text
Traffic
   |
   v
Azure Firewall
   |
   v
Allow / Deny
```

Simple memory:

**Route Table = Direction**

**Firewall = Security**

---

# 25. Interview Question

### What is a UDR in Azure?

### Answer:

> UDR stands for User Defined Route. It allows us to define a custom traffic path for a subnet, for example routing application traffic through Azure Firewall or another network virtual appliance.

---

# 26. Interview Question

### What is the difference between NSG and Route Table?

### Answer:

> NSG controls whether traffic is allowed or denied, while a route table controls where the traffic should go. For example, I can use an NSG to allow HTTPS and a UDR to route the traffic through Azure Firewall.

---

# 27. Interview Scenario

### How would you troubleshoot incorrect routing?

### Answer:

> I first check the subnet and associated route table, then verify the route destination and next-hop type or IP. After that I check the firewall, NSG and the destination to confirm the complete network path.

---

# 28. Hands-On Practice

For this topic, avoid creating a new firewall or NVA just for the lab because those resources can create unnecessary cost.

### Step 1 — List existing route tables

```bash
az network route-table list \
  --output table
```

### Step 2 — Inspect a route table

```bash
az network route-table show \
  --resource-group <resource-group> \
  --name <route-table-name>
```

### Step 3 — List routes

```bash
az network route-table route list \
  --resource-group <resource-group> \
  --route-table-name <route-table-name> \
  --output table
```

### Step 4 — Check subnet route-table association

```bash
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query "routeTable.id"
```

If there is no route table associated, the result can be:

```text
null
```

---

# 29. Final Mental Model

Remember:

```text
                Traffic
                   |
                   v
              Route Table
                   |
             Where to go?
                   |
                   v
              Next Hop
                   |
                   v
             Destination
```

And:

```text
NSG
 ↓
Allow / Deny

UDR
 ↓
Where should traffic go?
```

### Memory Trick

**NSG = Permission**

**UDR = Direction**

**Firewall = Inspection**
