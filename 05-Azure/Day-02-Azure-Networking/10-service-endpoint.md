# Azure Service Endpoint

## 1. What is a Service Endpoint?

An **Azure Service Endpoint** allows a VNet subnet to access supported Azure services through Azure's backbone network.

Example:

```text id="9x8q4m"
VM
 |
 v
Subnet
 |
 | Service Endpoint
 v
Azure Storage
```

It allows the Azure service to recognize traffic coming from the configured VNet/subnet.

---

# 2. Why Do We Need Service Endpoints?

Suppose an application is running inside:

```text id="g6p3y1"
VNet
 |
 +---- App Subnet
          |
          v
       Backend VM
```

The application needs to access Azure Storage.

A Service Endpoint can be enabled on the subnet:

```text id="x2m7r9"
App Subnet
     |
     | Microsoft.Storage
     v
Azure Storage
```

---

# 3. Simple Mental Model

Think of a Service Endpoint as:

**"Allow this subnet to access this Azure service."**

Example:

```text id="4r8k2v"
App Subnet
     |
     | Microsoft.Storage
     v
Azure Storage
```

---

# 4. Service Endpoint Is Enabled on a Subnet

This is important.

You don't normally configure the Service Endpoint directly on the VM.

Instead:

```text id="m7w4p1"
VNet
 |
 +---- App Subnet
          |
          +---- Service Endpoint
```

Any supported resource in that subnet can use the service endpoint.

---

# 5. Common Service Endpoint Services

Common examples include:

```text id="z1c8n5"
Microsoft.Storage
Microsoft.KeyVault
Microsoft.Sql
Microsoft.AzureCosmosDB
Microsoft.ServiceBus
```

The exact supported services depend on Azure service capabilities.

---

# 6. Service Endpoint Example

Suppose:

```text id="j5n9s3"
VNet
10.10.0.0/16
 |
 +---- App Subnet
      10.10.1.0/24
          |
          +---- VM
          |
          +---- Service Endpoint
                   |
                   v
             Azure Storage
```

The VM can access the supported Azure Storage service through the configured service endpoint.

---

# 7. Service Endpoint and Storage Firewall

A common production use case is Azure Storage network restrictions.

Suppose Storage is configured to allow selected VNets/subnets.

Architecture:

```text id="v6q3b2"
Application VM
     |
     v
App Subnet
     |
     | Service Endpoint
     v
Azure Storage
```

Storage can be configured to allow traffic from the required virtual network/subnet.

---

# 8. Service Endpoint vs Public Access

Without network restrictions:

```text id="k4x7p9"
Application
     |
     v
Storage Endpoint
```

With a Service Endpoint and network rules:

```text id="q9m2w6"
Application
     |
     v
VNet Subnet
     |
     | Service Endpoint
     v
Azure Storage
```

The Storage firewall/network rules can then restrict access to approved VNets/subnets.

---

# 9. Service Endpoint vs Private Endpoint

This is a very important interview topic.

### Service Endpoint

```text id="8c5v1r"
VM
 |
 v
Subnet
 |
 | Service Endpoint
 v
Azure Storage
```

The Azure service is still accessed through its service endpoint.

### Private Endpoint

```text id="3n7k5q"
VM
 |
 v
Subnet
 |
 v
Private Endpoint
 |
 | Private IP
 v
Azure Storage
```

The Azure service is represented by a private IP in your VNet.

---

# 10. Key Difference

| Feature                              | Service Endpoint                       | Private Endpoint          |
| ------------------------------------ | -------------------------------------- | ------------------------- |
| Configured on                        | Subnet                                 | Private Endpoint resource |
| Private IP in your VNet              | No                                     | Yes                       |
| Uses service endpoint                | Yes                                    | No                        |
| DNS changes commonly required        | No                                     | Yes                       |
| Common use                           | Restrict PaaS access to selected VNets | Private PaaS connectivity |
| PaaS service gets private IP in VNet | No                                     | Yes                       |

Simple memory:

```text id="k8w3p5"
Service Endpoint
→ Subnet access

Private Endpoint
→ Private IP
```

---

# 11. Enable Service Endpoint Using Azure CLI

Example for Storage:

```bash id="y4r6t2"
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --service-endpoints Microsoft.Storage
```

Now the subnet has:

```text id="e6n2v9"
Microsoft.Storage
```

enabled.

---

# 12. Enable Multiple Service Endpoints

For example:

```bash id="q3w8k1"
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --service-endpoints Microsoft.Storage Microsoft.KeyVault
```

Now the subnet can use the configured service endpoints for those services.

---

# 13. Check Service Endpoints

```bash id="n5h7x3"
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query "serviceEndpoints"
```

Example output:

```text id="w4k9m2"
[
  {
    "service": "Microsoft.Storage"
  }
]
```

---

# 14. List Subnets

```bash id="r7p2c5"
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --output table
```

---

# 15. Remove a Service Endpoint

If required:

```bash id="s6j4q8"
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --remove serviceEndpoints
```

Be careful with this in a production subnet because existing workloads may depend on the configuration.

---

# 16. Service Endpoint + Storage Example

Suppose we have:

```text id="e2x7m4"
VNet
 |
 +---- App Subnet
 |       |
 |       +---- Backend VM
 |       |
 |       +---- Microsoft.Storage
 |
 v
Azure Storage
```

Storage network rules can be configured to allow the required subnet.

Traffic:

```text id="b8q3k6"
Backend VM
    |
    v
App Subnet
    |
    | Service Endpoint
    v
Azure Storage
```

---

# 17. Production Scenario

Suppose a company has a backend application:

```text id="t9v5p2"
Backend VM
     |
     v
Application Subnet
     |
     | Microsoft.Storage
     v
Azure Storage
```

The security requirement is:

> Only the application subnet should access the Storage account.

A Service Endpoint can be enabled on the application subnet, and Storage network rules can allow the required subnet.

The architecture becomes:

```text id="h4n8s1"
Backend
   |
   v
App Subnet
   |
   | Service Endpoint
   v
Storage
```

---

# 18. Troubleshooting Scenario

### Problem

The application cannot access Storage.

Check:

```text id="z6k3p8"
1. Is the Service Endpoint enabled?
2. Is the VM actually inside the expected subnet?
3. Does Storage allow the required VNet/subnet?
4. Is the Storage firewall configuration correct?
5. Are NSG/routing rules affecting the traffic?
6. Is the application using the correct Storage endpoint?
```

Check the subnet:

```bash id="u2v7m5"
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query "serviceEndpoints"
```

---

# 19. Important Production Point

Service Endpoint and Private Endpoint are **not interchangeable**.

Use the architecture requirement to decide.

Example:

```text id="j3x9q6"
Need subnet-based access control
        |
        v
Service Endpoint
```

Where:

```text id="c5m8r2"
Need private IP connectivity to PaaS
        |
        v
Private Endpoint
```

---

# 20. Service Endpoint and DNS

One useful difference is DNS.

With a Service Endpoint:

```text id="p7x2k4"
Application
   |
   v
Storage hostname
   |
   v
Storage service endpoint
```

You generally don't need the Private DNS Zone pattern used with Private Endpoints.

With Private Endpoint:

```text id="w9m3c7"
Application
   |
   v
Private DNS
   |
   v
Private IP
   |
   v
Private Endpoint
```

---

# 21. Interview Question

### What is a Service Endpoint?

### Answer:

> A Service Endpoint allows a subnet in my Azure VNet to access supported Azure services through Azure's backbone network. For example, I can enable Microsoft.Storage on an application subnet and then restrict the Storage account to that subnet.

---

# 22. Interview Question

### Service Endpoint vs Private Endpoint?

### Answer:

> Service Endpoint is configured on a subnet and provides controlled access to a supported Azure service. Private Endpoint creates a private IP inside my VNet for the service. So the simple difference is subnet-based access versus private IP-based connectivity.

---

# 23. Interview Scenario

### How would you secure Azure Storage access from a VM?

### Answer:

> If the requirement is subnet-based access control, I can enable the Microsoft.Storage service endpoint on the application subnet and configure Storage network rules to allow that subnet. If private IP connectivity is required, I would use a Private Endpoint.

---

# 24. Hands-On Practice

Use an existing VNet and subnet from your Azure lab.

### Step 1 — List VNets

```bash id="j5x8m3"
az network vnet list \
  --output table
```

### Step 2 — List subnets

```bash id="w2n6p9"
az network vnet subnet list \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --output table
```

### Step 3 — Check existing Service Endpoints

```bash id="a7q4k1"
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query "serviceEndpoints"
```

### Step 4 — If appropriate, enable Storage endpoint

```bash id="m8v3c6"
az network vnet subnet update \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --service-endpoints Microsoft.Storage
```

### Step 5 — Verify

```bash id="q6r2t8"
az network vnet subnet show \
  --resource-group <resource-group> \
  --vnet-name <vnet-name> \
  --name <subnet-name> \
  --query "serviceEndpoints"
```

---

# 25. Final Mental Model

```text id="e9x3m7"
              VNet
                |
                v
             Subnet
                |
       Service Endpoint
                |
                v
          Azure Storage
```

### Memory Trick

**Service Endpoint = Subnet access**

**Private Endpoint = Private IP**

**NSG = Traffic filtering**

**Route Table = Traffic direction**
