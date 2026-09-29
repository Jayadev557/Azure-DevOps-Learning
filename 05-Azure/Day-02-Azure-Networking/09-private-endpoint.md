# Azure Private Endpoint

## 1. What is a Private Endpoint?

An **Azure Private Endpoint** provides a private IP address from your VNet to an Azure service.

Simple example:

```text id="9f7t3a"
VNet
10.10.0.0/16
     |
     v
Private Endpoint
10.10.3.10
     |
     v
Azure Storage
```

The application can access the Azure service through a private IP instead of using the service's public endpoint.

---

# 2. Why Do We Need Private Endpoint?

Suppose an application running in a private subnet needs to access Azure Storage.

Without Private Endpoint:

```text id="8v7z5m"
Backend VM
   |
   v
Public Storage Endpoint
   |
   v
Azure Storage
```

With Private Endpoint:

```text id="4f6qz8"
Backend VM
   |
   v
Private Endpoint
10.10.3.10
   |
   v
Azure Storage
```

This allows the application to use private connectivity to the Azure service.

---

# 3. Simple Mental Model

Think of Private Endpoint as a **private door to an Azure service**.

```text id="c1n8av"
VNet
 |
 | Private IP
 v
Private Endpoint
 |
 v
Azure Service
```

The service itself is still an Azure PaaS service, but your VNet gets a private network interface for accessing it.

---

# 4. Private Endpoint Architecture

Example:

```text id="2e6b7v"
                 VNet
        10.10.0.0/16
              |
      +-------+-------+
      |               |
      v               v
 Application      Private Endpoint
    VM              10.10.3.10
                        |
                        v
                  Azure Storage
```

The Private Endpoint is deployed into a subnet in your VNet.

---

# 5. Private Endpoint Uses a Private IP

Example:

```text id="g2f4p9"
Private Endpoint
Private IP: 10.10.3.10
```

The application can connect to the service using the private network path.

The private IP comes from the subnet where the Private Endpoint is created.

---

# 6. Private Endpoint vs Public Endpoint

### Public Endpoint

```text id="q2y7r0"
Application
    |
    v
Public DNS
    |
    v
Public Service Endpoint
    |
    v
Azure Storage
```

### Private Endpoint

```text id="1g4w7e"
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
    |
    v
Azure Storage
```

The second architecture provides private access from the VNet.

---

# 7. Common Azure Services Using Private Endpoints

Private Endpoints are commonly used with Azure PaaS services such as:

* Azure Storage
* Azure Key Vault
* Azure SQL
* Azure Database services
* Azure Container Registry
* Azure Cosmos DB

Example:

```text id="x1l3o8"
AKS
 |
 +---- Private Endpoint → ACR
 |
 +---- Private Endpoint → Key Vault
 |
 +---- Private Endpoint → Storage
```

---

# 8. Private Endpoint and DNS

DNS is extremely important with Private Endpoint.

Suppose the application uses:

```text id="5xq4m7"
storageaccount.blob.core.windows.net
```

With Private Endpoint, DNS should resolve the service name to the private IP.

Example:

```text id="4l7q3s"
storageaccount.blob.core.windows.net
              |
              v
       Private DNS Zone
              |
              v
         10.10.3.10
```

Then:

```text id="g8t5c1"
Application
     |
     v
storageaccount.blob.core.windows.net
     |
     v
10.10.3.10
     |
     v
Private Endpoint
```

---

# 9. Private DNS Zone

A Private DNS Zone allows private name resolution for supported Azure services.

For example:

```text id="6d8m2j"
Private DNS Zone
privatelink.blob.core.windows.net
```

The Private Endpoint can be associated with the appropriate Private DNS Zone.

---

# 10. Why DNS Troubleshooting Is Important

A Private Endpoint may be correctly created, but the application can still fail if DNS is not configured correctly.

Example:

```text id="n7y1kp"
Application
    |
    v
storageaccount.blob.core.windows.net
    |
    X
Wrong DNS resolution
```

The application may resolve the service to a public address instead of the intended private endpoint.

Therefore, when troubleshooting Private Endpoint connectivity, check:

```text id="j8c4s2"
Private Endpoint
        +
Private DNS
        +
DNS resolution
        +
NSG / Routing
```

---

# 11. Private Endpoint vs Service Endpoint

This is a very common interview question.

### Service Endpoint

```text id="9m0s5w"
VNet
 |
 v
Service Endpoint
 |
 v
Azure Storage
```

The service remains accessed through its service endpoint, while the VNet is extended to the service over Azure's backbone.

### Private Endpoint

```text id="2k8r6v"
VNet
 |
 v
Private IP
 |
 v
Private Endpoint
 |
 v
Azure Storage
```

The Azure service gets a private IP in your VNet.

Simple memory:

```text id="k6w3j8"
Service Endpoint
→ VNet-based service access

Private Endpoint
→ Private IP for the service
```

---

# 12. Create a Private Endpoint

Example for an existing supported Azure resource:

```bash id="3x8z6q"
az network private-endpoint create \
  --resource-group <resource-group> \
  --name <private-endpoint-name> \
  --vnet-name <vnet-name> \
  --subnet <private-endpoint-subnet> \
  --private-connection-resource-id <resource-id> \
  --connection-name <connection-name>
```

The exact resource ID and connection parameters depend on the Azure service.

---

# 13. List Private Endpoints

```bash id="q7h3b1"
az network private-endpoint list \
  --resource-group <resource-group> \
  --output table
```

---

# 14. Show a Private Endpoint

```bash id="6m4q2z"
az network private-endpoint show \
  --resource-group <resource-group> \
  --name <private-endpoint-name>
```

You can inspect:

```text id="u9p2q5"
Network interfaces
Subnet
Private IP
Private link connections
```

---

# 15. Check the Private Endpoint NIC

A Private Endpoint creates a network interface in your VNet.

Conceptually:

```text id="1k9x4m"
Private Endpoint
       |
       v
Private Endpoint NIC
       |
       v
Private IP
```

List NICs:

```bash id="j6s5z2"
az network nic list \
  --resource-group <resource-group> \
  --output table
```

---

# 16. Production Example: Key Vault

Suppose AKS needs to access Key Vault.

Instead of:

```text id="e3c9k7"
AKS
 |
 v
Public Key Vault Endpoint
```

we can use:

```text id="5a7x2p"
AKS
 |
 v
Private Network
 |
 v
Private Endpoint
 |
 v
Key Vault
```

DNS resolves the Key Vault hostname to the private endpoint IP.

---

# 17. Production Example: Azure Container Registry

Suppose AKS needs to pull container images from ACR.

Architecture:

```text id="z4f8b1"
             AKS
              |
              v
       Private Network
              |
              v
       Private Endpoint
              |
              v
             ACR
```

This is useful when an organization wants registry access to remain private.

---

# 18. Private Endpoint and NSG

Private Endpoint traffic can still be affected by network security configuration depending on the architecture.

For troubleshooting, check:

```text id="p7s4v2"
Application
   |
   v
Private IP
   |
   v
Private Endpoint
   |
   v
Azure Service
```

Then verify relevant:

```text id="q8j2x5"
NSG
Route
DNS
Service configuration
```

---

# 19. Troubleshooting Scenario

### Problem

A VM cannot access Azure Storage through its Private Endpoint.

Check in this order:

```text id="6j3r8w"
1. Is the Private Endpoint created?
2. Does it have a private IP?
3. Is the Private Endpoint connected to the correct subnet?
4. Is DNS resolving the storage hostname to the private IP?
5. Is the Private DNS Zone configured correctly?
6. Are NSG/routing rules blocking traffic?
7. Is the application using the correct hostname?
```

A very common mistake is checking only the Private Endpoint and forgetting DNS.

---

# 20. Useful DNS Test

From a machine that can resolve the service hostname:

```bash id="m1q7d4"
nslookup <service-hostname>
```

or:

```bash id="w5x8k3"
dig <service-hostname>
```

You want to verify that the name resolves through the expected private DNS configuration.

---

# 21. Private Endpoint vs Public IP

Private Endpoint:

```text id="7p4h6c"
Private IP
    |
    v
Azure PaaS Service
```

Public access:

```text id="2m9q1z"
Public Endpoint
    |
    v
Azure PaaS Service
```

In a private production architecture, Private Endpoint can be used to provide private access to supported Azure services.

---

# 22. Interview Question

### What is a Private Endpoint in Azure?

### Answer:

> A Private Endpoint gives an Azure PaaS service a private IP address inside my VNet. For example, I can use it to access Key Vault, Storage or ACR privately instead of relying on the service's public endpoint.

---

# 23. Interview Question

### Why is DNS important with Private Endpoint?

### Answer:

> Private Endpoint connectivity depends heavily on DNS. The service hostname should resolve to the Private Endpoint's private IP, so I always check the Private DNS Zone and DNS resolution when troubleshooting connectivity.

---

# 24. Interview Question

### Private Endpoint vs Service Endpoint?

### Answer:

> With a Service Endpoint, the VNet is allowed to access the Azure service through its service endpoint. With a Private Endpoint, the service is accessed through a private IP in my VNet. Private Endpoint is commonly used when we need private connectivity to PaaS services.

---

# 25. Hands-On Practice

Use an existing supported Azure resource if you already have one.

Do not create a new PaaS resource only for this topic if it creates unnecessary cost.

### Step 1 — Check existing Private Endpoints

```bash id="4w7s9k"
az network private-endpoint list \
  --output table
```

### Step 2 — Inspect one

```bash id="2n6h4v"
az network private-endpoint show \
  --resource-group <resource-group> \
  --name <private-endpoint-name>
```

### Step 3 — Check its NIC

```bash id="7x5p3m"
az network private-endpoint show \
  --resource-group <resource-group> \
  --name <private-endpoint-name> \
  --query "networkInterfaces[].id"
```

### Step 4 — Check DNS

From a machine connected to the required network:

```bash id="3q8j5a"
nslookup <service-hostname>
```

Verify that the hostname resolves through the expected private endpoint configuration.

---

# 26. Final Mental Model

Remember:

```text id="f3k8z2"
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
     |
     v
Azure PaaS Service
```

### Memory Trick

**Private Endpoint = Private IP to an Azure service**

**Private DNS = Name → Private IP**

**Service Endpoint = VNet-based access to the service**
