# Azure VNet Peering

## 1. What is VNet Peering?

VNet Peering connects two Azure Virtual Networks so that resources in those VNets can communicate using private IP addresses.

Simple example:

```text
VNet-A
10.10.0.0/16
     |
     | VNet Peering
     |
     v
VNet-B
10.20.0.0/16
```

After peering, resources in both VNets can communicate through Azure's private network.

---

# 2. Why Do We Need VNet Peering?

By default, two separate VNets are isolated.

Example:

```text
VNet-A                    VNet-B
10.10.0.0/16             10.20.0.0/16

Backend VM                Database VM
10.10.2.10                10.20.2.10
```

Without connectivity between the VNets:

```text
Backend VM  X  Database VM
```

With VNet Peering:

```text
Backend VM
10.10.2.10
     |
     v
 VNet Peering
     |
     v
Database VM
10.20.2.10
```

---

# 3. How VNet Peering Works

Consider:

```text
VNet-A
10.10.0.0/16
     |
     | Peering
     |
VNet-B
10.20.0.0/16
```

A resource in VNet-A can communicate with a resource in VNet-B using its private IP.

Example:

```text
VM-A
10.10.1.10
     |
     | Private network
     v
VM-B
10.20.1.10
```

---

# 4. VNets Must Have Non-Overlapping Address Spaces

This is very important.

Example:

```text
VNet-A
10.10.0.0/16

VNet-B
10.20.0.0/16
```

These address spaces do not overlap.

But:

```text
VNet-A
10.10.0.0/16

VNet-B
10.10.0.0/16
```

They overlap.

Overlapping address spaces create routing problems and are not suitable for normal VNet peering.

---

# 5. Peering Is Configured on Both VNets

Suppose:

```text
VNet-A
   |
   | Peering
   |
VNet-B
```

Azure creates peering relationships in both directions.

Conceptually:

```text
VNet-A
  |
  +---- Peering ----> VNet-B
  |
  <---- Peering ----+
```

Both sides need the appropriate peering configuration.

---

# 6. Production Example

Suppose a company has:

```text
Hub VNet
10.0.0.0/16

Spoke VNet
10.10.0.0/16
```

Architecture:

```text
              Hub VNet
           10.0.0.0/16
                |
                | Peering
                |
                v
            Spoke VNet
           10.10.0.0/16
```

The hub can contain shared services such as:

* Azure Firewall
* VPN Gateway
* DNS infrastructure
* Shared management services

The spoke can contain application workloads.

---

# 7. Hub-and-Spoke Architecture

A common enterprise design is:

```text
                    HUB VNET
                 10.0.0.0/16
                       |
          +------------+------------+
          |            |            |
          v            v            v
       Spoke-1      Spoke-2      Spoke-3
       App VNet     App VNet     App VNet
```

Each spoke is separately connected to the hub.

This makes the hub a central networking location.

---

# 8. VNet Peering vs Internet

VNet peering does not require traffic to travel over the public internet.

Example:

```text
VM-A
  |
  v
VNet-A
  |
  | Azure private network
  v
VNet-B
  |
  v
VM-B
```

This is one reason peering is commonly used for communication between Azure VNets.

---

# 9. VNet Peering Across Regions

Azure also supports peering between VNets in different Azure regions.

Example:

```text
East US
VNet-A
   |
   | Global VNet Peering
   |
   v
West Europe
VNet-B
```

This is useful when an application has workloads distributed across regions.

Example:

```text
East US
Application
     |
     | Global VNet Peering
     |
West Europe
Database / Shared Service
```

---

# 10. Local vs Global VNet Peering

### VNet Peering

Used when VNets are in the same Azure region.

```text
East US
VNet-A ---- VNet-B
```

### Global VNet Peering

Used when VNets are in different Azure regions.

```text
East US
VNet-A
   |
   | Global Peering
   |
West Europe
VNet-B
```

---

# 11. Create VNet Peering

First, create the peering from VNet-A to VNet-B:

```bash id="5m8k9p"
az network vnet peering create \
  --resource-group <resource-group-a> \
  --vnet-name <vnet-a> \
  --name <peering-a-to-b> \
  --remote-vnet <vnet-b-resource-id> \
  --allow-vnet-access
```

Then create the reverse peering:

```bash id="4o5v5e"
az network vnet peering create \
  --resource-group <resource-group-b> \
  --vnet-name <vnet-b> \
  --name <peering-b-to-a> \
  --remote-vnet <vnet-a-resource-id> \
  --allow-vnet-access
```

---

# 12. List VNet Peerings

```bash id="xjz4gi"
az network vnet peering list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --output table
```

You can check:

```text
PeeringState
PeeringSyncLevel
AllowVirtualNetworkAccess
```

A healthy peering should show the expected connected state.

---

# 13. Show a Specific Peering

```bash id="xg1u1v"
az network vnet peering show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <peering-name>
```

---

# 14. Delete a Peering

If required:

```bash id="b9axq5"
az network vnet peering delete \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <peering-name>
```

Only remove a peering when you know it is not required by existing workloads.

---

# 15. Peering and NSG

VNet peering provides connectivity, but NSGs can still control traffic.

Example:

```text id="dj3m40"
VM-A
  |
  v
VNet-A
  |
  | Peering
  v
VNet-B
  |
  v
NSG
  |
  v
VM-B
```

If the required traffic is blocked by an NSG, communication can still fail.

So when troubleshooting peering, check both:

```text
Peering
+
NSG
```

---

# 16. Peering and Routing

Peering provides a network path, but routing configuration still matters.

Example:

```text id="x47a8d"
VM-A
 |
 v
Route Table
 |
 v
VNet Peering
 |
 v
VM-B
```

A custom UDR can change the traffic path.

Therefore, if communication fails, check:

```text
1. Peering state
2. Address spaces
3. Route tables / UDR
4. NSG
5. Destination
```

---

# 17. Important: Peering Is Not Transitive by Default

Consider:

```text id="d2t8bm"
VNet-A
   |
   | Peering
   |
VNet-B
   |
   | Peering
   |
VNet-C
```

Just because:

```text
A ↔ B
B ↔ C
```

does not automatically mean:

```text
A ↔ C
```

VNet peering is not automatically transitive.

For hub-and-spoke architectures, additional routing or network services may be required when spokes need to communicate through the hub.

---

# 18. Hub-and-Spoke Traffic Example

Suppose:

```text
Spoke-1
   |
   | Peering
   v
  HUB
   |
   | Peering
   v
Spoke-2
```

If Spoke-1 needs to communicate with Spoke-2 through a central firewall, the architecture may use:

```text
Spoke-1
   |
   v
Hub Firewall
   |
   v
Spoke-2
```

UDRs and firewall configuration can be used to control that path.

---

# 19. VNet Peering vs VPN Gateway

Both can connect networks, but they are used for different scenarios.

### VNet Peering

Commonly used for:

```text
Azure VNet
    |
    v
Azure VNet
```

### VPN Gateway

Commonly used for:

```text
On-premises
    |
    | VPN tunnel
    v
Azure VNet
```

Simple memory:

```text
Azure-to-Azure
→ VNet Peering

On-premises-to-Azure
→ VPN Gateway
```

---

# 20. Troubleshooting Scenario

### Problem

Two VMs are in different VNets and cannot communicate.

Check in this order:

```text id="u1f7r6"
1. Are the VNet address spaces non-overlapping?
2. Is peering configured on both sides?
3. Is the peering state connected?
4. Are route tables/UDRs correct?
5. Are NSGs allowing the traffic?
6. Is the application listening on the required port?
```

For example:

```text id="y1zjhz"
VM-A
10.10.1.10
   |
   v
VNet-A
   |
   | Peering
   |
VNet-B
   |
   v
VM-B
10.20.1.10
```

If the peering is connected but TCP 443 is blocked by an NSG, the application will still be unreachable.

---

# 21. Useful CLI Commands

List VNets:

```bash id="b6h5sj"
az network vnet list \
  --output table
```

Show VNet address space:

```bash id="f3h24u"
az network vnet show \
  --resource-group <resource-group> \
  --name <vnet-name> \
  --query "addressSpace.addressPrefixes"
```

List peerings:

```bash id="j8zj87"
az network vnet peering list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --output table
```

Show peering:

```bash id="9v0h7j"
az network vnet peering show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <peering-name>
```

---

# 22. Interview Question

### What is VNet Peering?

### Answer:

> VNet peering connects two Azure VNets so resources can communicate using private IP addresses. It is commonly used for Azure-to-Azure communication, such as connecting application workloads in a spoke VNet with shared services in a hub VNet.

---

# 23. Interview Question

### Is VNet Peering Transitive?

### Answer:

> No. VNet peering is not automatically transitive. If VNet-A is peered with VNet-B and VNet-B is peered with VNet-C, A cannot automatically communicate with C through B.

---

# 24. Interview Scenario

### Two VNets are peered, but the VMs cannot communicate. What will you check?

### Answer:

> I first check whether the peering state is connected and the VNet address spaces don't overlap. Then I check UDRs, NSG rules, and finally whether the application is listening on the required port.

---

# 25. Hands-On Practice

Use existing VNets only. Do not create additional paid resources just for this topic.

### Step 1 — List VNets

```bash id="19t1fi"
az network vnet list \
  --output table
```

### Step 2 — Check address spaces

```bash id="t3qcb1"
az network vnet show \
  --resource-group <resource-group> \
  --name <vnet-name> \
  --query "addressSpace.addressPrefixes"
```

### Step 3 — Check existing peerings

```bash id="7uhjvy"
az network vnet peering list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --output table
```

If you already have two suitable non-overlapping VNets, you can practice creating peering between them.

### Step 4 — Verify peering

```bash id="t2o7bd"
az network vnet peering show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <peering-name> \
  --query "{State:peeringState,Access:allowVirtualNetworkAccess}"
```

---

# 26. Final Mental Model

Remember:

```text
VNet-A
   |
   | Peering
   |
VNet-B
```

This provides:

```text
Private Azure-to-Azure connectivity
```

Production:

```text
                 HUB VNET
                    |
          +---------+---------+
          |                   |
          v                   v
      Spoke-1              Spoke-2
     Application          Application
```

### Memory Trick

**VNet Peering = Connect two VNets**

**NSG = Allow/Deny traffic**

**UDR = Control traffic path**

**VPN = Connect external/on-prem network**
