# Azure Network Interface (NIC)

## 1. What is a NIC?

A **Network Interface Card (NIC)** is the networking component that connects an Azure VM to a VNet.

Simple flow:

```text
Azure VM
   |
   v
NIC
   |
   +---- Private IP
   |
   +---- Subnet
   |
   +---- NSG
   |
   v
VNet
```

Think of the NIC as the **network identity of the VM**.

---

# 2. Why Do We Need a NIC?

A VM needs a network interface to communicate with other resources.

For example:

```text
VM
 |
 v
NIC
 |
 v
Backend Subnet
 |
 v
VNet
```

The NIC provides the connection between the VM and the Azure network.

---

# 3. What Does a NIC Contain?

An Azure NIC can contain an IP configuration with:

```text
Private IP
Public IP association
Subnet
IP allocation method
NSG association
```

Example:

```text
NIC
 |
 +-- Private IP: 10.10.2.10
 |
 +-- Subnet: backend-subnet
 |
 +-- Public IP: optional
 |
 +-- NSG: backend-nsg
```

---

# 4. NIC and Private IP

The private IP is assigned to the NIC's IP configuration.

Example:

```text
Backend VM
     |
     v
backend-nic
     |
     v
Private IP: 10.10.2.10
```

Other resources inside the VNet can use this private IP to communicate with the VM.

---

# 5. NIC and Public IP

A NIC can also have a public IP association.

Example:

```text
VM
 |
 v
NIC
 |
 +---- Private IP: 10.10.2.10
 |
 +---- Public IP: <public-ip>
```

But a public IP is **not required** for every VM.

A backend VM can remain private:

```text
Application Gateway
       |
       | Private traffic
       v
Backend VM
       |
       v
NIC
       |
       v
Private IP
```

---

# 6. NIC and Subnet

Every NIC IP configuration is connected to a subnet.

Example:

```text
VNet: 10.10.0.0/16
       |
       +---- frontend-subnet
       |
       +---- backend-subnet
                    |
                    v
                 NIC
                    |
                    v
                   VM
```

The NIC therefore determines which subnet the VM is connected to.

---

# 7. NIC and NSG

An NSG can be associated with the NIC.

Example:

```text
Internet
   |
   v
NSG
   |
   v
NIC
   |
   v
VM
```

The NSG controls allowed and denied network traffic.

An NSG can also be associated at the subnet level.

```text
VNet
 |
 +---- Subnet ---- NSG
 |                  |
 |                  v
 |                NIC
 |                  |
 |                  v
 |                  VM
```

---

# 8. NIC vs VM

A VM is the compute resource.

A NIC is the networking resource.

Think:

```text
VM = Computer
NIC = Network connection
```

For example:

```text
VM
 |
 +---- OS
 +---- CPU
 +---- RAM
 |
 +---- NIC
        |
        +---- Private IP
        +---- Subnet
        +---- NSG
```

---

# 9. One VM Can Have Multiple NICs

Azure supports multiple NICs for supported VM sizes.

Example:

```text
             VM
          /     \
         /       \
      NIC-1     NIC-2
        |          |
   Frontend     Backend
   subnet       subnet
```

This can be useful for specific network architectures where a VM needs connectivity to different network segments.

---

# 10. Primary NIC

A VM normally has a **primary NIC**.

Example:

```text
VM
 |
 +---- NIC-1  ← Primary
 |
 +---- NIC-2
```

The primary NIC is used for the VM's primary network connectivity.

---

# 11. Multiple IP Configurations

A NIC can have multiple IP configurations.

Example:

```text
NIC
 |
 +---- IP Configuration 1
 |       |
 |       +-- Private IP
 |
 +---- IP Configuration 2
         |
         +-- Private IP
```

This allows a NIC to have multiple network identities when the architecture requires it.

---

# 12. Check NICs Using Azure CLI

List NICs:

```bash
az network nic list \
  --resource-group <resource-group> \
  --output table
```

Example output:

```text
Name              Location
----------------  --------
frontend-nic      eastus
backend-nic       eastus
```

---

# 13. Show a NIC

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name>
```

You can inspect:

```text
ipConfigurations
networkSecurityGroup
virtualMachine
location
```

---

# 14. Check NIC IP Configuration

You can query the private IP directly:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query "ipConfigurations[].privateIPAddress"
```

Example:

```text
[
  "10.10.2.10"
]
```

Check the subnet:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query "ipConfigurations[].subnet.id"
```

---

# 15. Check NSG Associated With NIC

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query "networkSecurityGroup.id"
```

If an NSG is associated, you will get its resource ID.

---

# 16. Find Which VM Uses a NIC

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query "virtualMachine.id"
```

This helps during troubleshooting.

Example:

```text
NIC
 |
 +---- VM
 |
 +---- Private IP
 |
 +---- Subnet
 |
 +---- NSG
```

---

# 17. Real Production Example

Suppose we have:

```text
VNet
 |
 +---- frontend-subnet
 |         |
 |         v
 |    frontend-nic
 |         |
 |         v
 |    frontend VM
 |
 +---- backend-subnet
           |
           v
      backend-nic
           |
           v
      backend VM
```

Each VM has its own NIC.

The NIC connects the VM to the appropriate subnet.

---

# 18. Troubleshooting Scenario

### Problem

A backend VM cannot communicate with another server.

Instead of checking only the VM, check the networking chain:

```text
VM
 ↓
NIC
 ↓
Private IP
 ↓
Subnet
 ↓
NSG
 ↓
Route
 ↓
Destination
```

Check the NIC:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name>
```

Verify:

```text
1. Correct subnet
2. Correct private IP
3. Correct NSG
4. Public IP association if required
5. VM association
```

---

# 19. Common Interview Question

### Question:

What is a NIC in Azure?

### Answer:

> NIC is the networking interface attached to an Azure VM. It connects the VM to a subnet and provides the private IP, and it can also have a public IP association and NSG configuration.

---

# 20. Interview Question: NIC vs Private IP

### Answer:

> NIC is the network interface attached to the VM, while the private IP is assigned through the NIC's IP configuration. So the NIC provides the network connection and the IP identifies the VM on that network.

---

# 21. Interview Scenario

### Question:

A VM is running, but it cannot communicate with another VM. What will you check?

### Answer:

> First I check the NIC and confirm the private IP and subnet. Then I check NSG rules, route configuration, and whether the destination VM is listening on the required port.

---

# 22. Hands-On Practice

Use an existing VM from your Azure lab.

### Step 1 — List VMs

```bash
az vm list \
  --output table
```

### Step 2 — List NICs

```bash
az network nic list \
  --resource-group <resource-group> \
  --output table
```

### Step 3 — Inspect the NIC

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name>
```

### Step 4 — Check Private IP

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query "ipConfigurations[].privateIPAddress"
```

### Step 5 — Check Subnet

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query "ipConfigurations[].subnet.id"
```

### Step 6 — Check NSG

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  --query "networkSecurityGroup.id"
```

---

# 23. Final Mental Model

Remember:

```text
VM
 |
 v
NIC
 |
 +---- Private IP
 |
 +---- Subnet
 |
 +---- NSG
 |
 +---- Optional Public IP
 |
 v
VNet
```

### Memory Trick

**VM = Computer**

**NIC = Network connection**

**Private IP = Network identity**

**Subnet = Network segment**

**NSG = Traffic filter**
