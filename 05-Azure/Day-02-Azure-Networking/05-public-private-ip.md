# Public IP vs Private IP in Azure

## 1. What is an IP Address?

An IP address identifies a network interface or service so that network traffic knows where to go.

In Azure, we commonly work with:

* Private IP
* Public IP

Simple mental model:

```text
Internet
   |
   | Public IP
   v
Frontend / Application Gateway
   |
   | Private IP
   v
Backend VM / API
   |
   | Private IP
   v
Database
```

---

# 2. Private IP

A **private IP** is used for communication inside private networks such as an Azure VNet.

Example:

```text
VNet
10.10.0.0/16

Frontend subnet
10.10.1.0/24

Backend subnet
10.10.2.0/24
```

A backend VM might have:

```text
Private IP: 10.10.2.10
```

Other Azure resources inside the VNet can communicate with this private IP.

Example:

```text
Frontend
10.10.1.10
     |
     | HTTP
     v
Backend
10.10.2.10
```

The backend does not need a public IP for this communication.

---

# 3. Public IP

A **public IP** provides an address that can be reached from the public internet, when the associated Azure service and network/security configuration allow it.

Example:

```text
Internet
    |
    | Public IP
    v
Application Gateway
    |
    | Private IP
    v
Backend VM
```

A public IP is commonly used with internet-facing components such as:

* Application Gateway
* Public Load Balancer
* Azure Firewall
* NAT Gateway
* VPN Gateway
* Bastion

---

# 4. Private IP vs Public IP

| Feature                    | Private IP | Public IP                    |
| -------------------------- | ---------- | ---------------------------- |
| Used inside VNet           | Yes        | No                           |
| Internet-facing            | No         | Yes                          |
| Backend communication      | Common     | Usually unnecessary          |
| Typical example            | VM NIC     | Application Gateway frontend |
| Source of internal traffic | Private IP | Usually private path         |
| Security exposure          | Lower      | Internet-facing              |

---

# 5. Example: Three-Tier Application

Consider an application with:

```text
Internet
   |
   v
Application Gateway
Public IP
   |
   | Private communication
   v
Frontend / Backend
Private IP
   |
   v
Database
Private IP
```

For example:

```text
Application Gateway
Public IP: <public-ip>

Backend VM
Private IP: 10.10.2.10

Database
Private IP: 10.10.3.10
```

Users access the application through the Application Gateway.

The backend VM communicates with the database using private networking.

---

# 6. Why Backend VMs Usually Don't Need Public IP

Suppose we have:

```text
Internet
    |
    v
Application Gateway
    |
    v
Backend VM
```

The backend VM can remain private.

This provides a cleaner architecture because users do not directly access the backend VM from the internet.

Instead:

```text
User
  |
  v
Public Application Gateway
  |
  v
Private Backend
```

This also allows the Application Gateway and NSG rules to control how traffic reaches the backend.

---

# 7. Private IP on an Azure VM

An Azure VM communicates through its network interface (NIC).

Example:

```text
VM
 |
 v
NIC
 |
 +---- Private IP: 10.10.2.10
 |
 +---- Public IP: optional
```

The private IP belongs to the NIC's IP configuration.

You can check it using Azure CLI:

```bash
az vm list-ip-addresses \
  --resource-group <resource-group> \
  --name <vm-name>
```

Example output can contain:

```text
PrivateIPAddress: 10.10.2.10
PublicIPAddress: 20.x.x.x
```

---

# 8. Check NIC IP Configuration

List NICs:

```bash
az network nic list \
  --resource-group <resource-group> \
  --output table
```

Show a specific NIC:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name>
```

Look for the IP configuration:

```text
ipConfigurations
```

It contains information such as:

```text
privateIPAddress
privateIPAllocationMethod
publicIPAddress
```

---

# 9. Dynamic vs Static Private IP

Azure private IP allocation can be:

```text
Dynamic
Static
```

### Dynamic

Azure assigns the private IP from the subnet.

Example:

```text
Backend subnet
10.10.2.0/24

VM
10.10.2.10
```

The address is managed by Azure.

### Static

You explicitly configure the private IP.

Example:

```text
Database VM
Private IP = 10.10.2.20
```

Static private IPs can be useful when a resource needs a predictable address.

---

# 10. Public IP Example

List public IP resources:

```bash
az network public-ip list \
  --resource-group <resource-group> \
  --output table
```

Show a specific public IP:

```bash
az network public-ip show \
  --resource-group <resource-group> \
  --name <public-ip-name>
```

You can check:

```text
ipAddress
publicIPAllocationMethod
sku
```

---

# 11. Public IP Does Not Automatically Mean Open Access

Having a public IP does not mean every port is automatically accessible.

Traffic can still be controlled by:

```text
Public IP
    |
    v
NSG / Firewall
    |
    v
Application
```

For example, a VM may have a public IP but SSH access can still be blocked by an NSG.

So when troubleshooting internet access, don't check only the IP.

Check:

```text
Public IP
   ↓
NSG
   ↓
Route
   ↓
Load Balancer / Application Gateway
   ↓
VM / Application
   ↓
Application port
```

---

# 12. Public IP vs Private IP in Production

A common production design is:

```text
                    Internet
                       |
                       v
              Public IP / DNS
                       |
                       v
             Application Gateway
                       |
                 Private network
                       |
              +--------+--------+
              |                 |
              v                 v
          Frontend           Backend
        Private IP          Private IP
                                |
                                v
                           Database
                          Private IP
```

The public-facing component handles internet traffic.

Internal application components use private IPs.

---

# 13. Administration Without Public IP

A backend VM does not necessarily need a public IP just because an administrator needs access.

For example:

```text
Administrator
      |
      v
Azure Bastion
      |
      | Private connection
      v
Backend VM
```

The VM can remain private.

This is a common production approach.

---

# 14. Public IP vs NAT Gateway

These two solve different problems.

### Public IP

Used when a service needs an internet-facing endpoint.

```text
Internet
   |
   v
Public IP
   |
   v
Application Gateway
```

### NAT Gateway

Primarily provides controlled outbound internet connectivity for resources in a subnet.

```text
Private VM
    |
    | Outbound
    v
NAT Gateway
    |
    v
Internet
```

The backend VM can therefore remain without its own public IP while still having outbound internet access through NAT Gateway.

---

# 15. Real Production Scenario

Suppose an application has:

```text
Application Gateway
Public IP

Backend VMSS
Private IPs

Azure Database
Private connectivity
```

User request:

```text
https://app.example.com
```

Traffic flows:

```text
User
  |
  v
DNS
  |
  v
Application Gateway
  |
  | Private IP
  v
Backend VMSS
  |
  | Private connection
  v
Database
```

The backend servers don't need to be directly exposed to the internet.

---

# 16. Troubleshooting Scenario

### Problem

The application is not accessible from the internet.

First check:

```text
1. Is DNS resolving correctly?
2. Does the frontend have the expected public endpoint?
3. Is Application Gateway / Load Balancer healthy?
4. Is the NSG allowing the required port?
5. Is routing correct?
6. Is the backend reachable?
7. Is the application actually listening on the required port?
```

For example:

```text
User
 |
 X
Public endpoint
```

Do not immediately assume the VM is down.

The problem could be at:

```text
DNS
 ↓
Public IP
 ↓
Load Balancer / App Gateway
 ↓
NSG
 ↓
Route
 ↓
Backend
 ↓
Application
```

---

# 17. Useful Azure CLI Commands

List VM IP addresses:

```bash
az vm list-ip-addresses \
  --resource-group <resource-group> \
  --name <vm-name>
```

List public IPs:

```bash
az network public-ip list \
  --resource-group <resource-group> \
  --output table
```

List NICs:

```bash
az network nic list \
  --resource-group <resource-group> \
  --output table
```

Show NIC:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name>
```

---

# 18. Interview Question: Public IP vs Private IP

### Question:

What is the difference between public and private IP in Azure?

### Answer:

> Private IP is used for communication inside the Azure VNet, while public IP is used for internet-facing connectivity. In production, I normally keep backend VMs private and expose only components like Application Gateway or Load Balancer to the internet.

---

# 19. Interview Question: Does Every VM Need a Public IP?

### Answer:

> No. A VM does not need a public IP for internal communication. For example, backend VMs can use private IPs and receive traffic through an Application Gateway or Load Balancer. For administration, we can use Azure Bastion instead of exposing the VM directly.

---

# 20. Interview Question: How Would You Troubleshoot an Unreachable VM?

### Answer:

> First I check whether the VM has the expected private or public IP. Then I check NSG rules, routing, Load Balancer or Application Gateway configuration, and finally whether the application is listening on the required port.

---

# 21. Hands-On Practice

Use an existing VM or NIC from your Azure lab.

### Step 1 — List VMs

```bash
az vm list \
  --output table
```

### Step 2 — Check VM IP addresses

```bash
az vm list-ip-addresses \
  --resource-group <resource-group> \
  --name <vm-name>
```

### Step 3 — List NICs

```bash
az network nic list \
  --resource-group <resource-group> \
  --output table
```

### Step 4 — Inspect the NIC

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name>
```

Identify:

```text
Private IP
Public IP association
Subnet
NSG association
```

### Step 5 — List Public IP resources

```bash
az network public-ip list \
  --resource-group <resource-group> \
  --output table
```

---

# 22. Final Mental Model

Remember it like this:

```text
PUBLIC IP
    ↓
Internet-facing access

PRIVATE IP
    ↓
Internal VNet communication
```

Production:

```text
             INTERNET
                 |
                 v
          Public Endpoint
                 |
                 v
       Application Gateway
                 |
                 | Private IP
                 v
          Backend VM / VMSS
                 |
                 | Private IP
                 v
             Database
```

### Memory Trick

**Public IP = Outside**

**Private IP = Inside**

And in production:

**Expose the frontend, keep the backend private.**
