# Azure DNS

## 1. What is DNS?

DNS stands for:

```text
Domain Name System
```

DNS converts a hostname into an IP address.

Example:

```text
www.example.com
        |
        v
     DNS
        |
        v
20.x.x.x
```

Instead of remembering an IP address, users/application can use a hostname.

---

# 2. Simple Real-World Example

Suppose your application calls:

```text
https://api.example.com
```

The application first needs to know:

```text
api.example.com → Which IP address?
```

DNS provides the answer.

```text
Application
     |
     | DNS Query
     v
DNS Server
     |
     | IP Address
     v
20.x.x.x
```

Then the application connects to that IP.

---

# 3. Why DNS Is Important in Azure

DNS is involved in many Azure services:

```text
VM
AKS
Application Gateway
Load Balancer
Private Endpoint
Storage
Key Vault
Azure SQL
App Service
Private DNS
```

A networking issue is sometimes actually a DNS issue.

For example:

```text
Application
    |
    v
api.example.com
    |
    X
DNS resolution fails
```

The application may report:

```text
Connection timeout
Name resolution failed
Could not resolve host
```

So as a DevOps engineer, DNS troubleshooting is very important.

---

# 4. Public DNS vs Private DNS

Azure networking commonly involves two types of DNS requirements:

```text
Public DNS
Private DNS
```

---

# 5. Public DNS

Public DNS is used for names that need to resolve from the public internet.

Example:

```text
www.example.com
      |
      v
Public DNS
      |
      v
Public IP
```

Example architecture:

```text
Internet
   |
   v
www.example.com
   |
   v
Public DNS
   |
   v
20.x.x.x
   |
   v
Application Gateway
```

Public DNS is useful for internet-facing applications.

---

# 6. Azure DNS

Azure DNS is Microsoft's DNS hosting service.

You can host DNS zones in Azure.

Example:

```text
example.com
```

Inside the zone you can create records such as:

```text
www.example.com
api.example.com
dev.example.com
```

Example:

```text
example.com
   |
   +-- www
   |
   +-- api
   |
   +-- dev
```

---

# 7. DNS Zone

A DNS zone contains DNS records for a domain.

Example:

```text
Zone:
example.com
```

Records:

```text
www.example.com
api.example.com
mail.example.com
```

Think:

```text
DNS Zone
   |
   +-- DNS Records
```

---

# 8. Common DNS Record Types

The most important records for interviews are:

```text
A
AAAA
CNAME
MX
TXT
NS
```

---

# 9. A Record

An A record maps a hostname to an IPv4 address.

Example:

```text
api.example.com
        |
        v
20.10.10.10
```

Conceptually:

```text
api.example.com → 20.10.10.10
```

---

# 10. AAAA Record

AAAA maps a hostname to an IPv6 address.

Example:

```text
api.example.com
        |
        v
IPv6 Address
```

For most basic Azure DevOps scenarios, A and CNAME records are encountered more frequently.

---

# 11. CNAME Record

CNAME maps one hostname to another hostname.

Example:

```text
app.example.com
       |
       v
myapp.azurewebsites.net
```

So:

```text
app.example.com
      ↓
myapp.azurewebsites.net
```

CNAME does not directly represent an IPv4 address.

---

# 12. MX Record

MX is used for mail routing.

Example:

```text
example.com
    |
    v
MX Record
    |
    v
Mail Server
```

---

# 13. TXT Record

TXT records store text information.

Common uses include:

```text
Domain verification
SPF
Other DNS-related configuration
```

---

# 14. NS Record

NS records identify authoritative name servers for a DNS zone.

Conceptually:

```text
example.com
    |
    v
Name Servers
```

---

# 15. Azure Private DNS

Azure Private DNS is used for DNS resolution inside private Azure networks.

Example:

```text
Private VM
10.0.1.10
      |
      v
Private DNS
      |
      v
10.0.2.10
```

The IP does not need to be publicly reachable.

---

# 16. Why Private DNS Is Important

Private DNS becomes extremely important with:

```text
Private Endpoint
```

Suppose Azure Storage normally has a hostname such as:

```text
mystorage.blob.core.windows.net
```

With a Private Endpoint, we want the application to reach the Storage service using a private IP.

Architecture:

```text
Application
     |
     | mystorage.blob.core.windows.net
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

This is one of the most important Azure networking concepts.

---

# 17. Private Endpoint + Private DNS

Suppose:

```text
Storage Account
```

has a Private Endpoint:

```text
Private IP:
10.0.3.10
```

The application still uses:

```text
mystorage.blob.core.windows.net
```

DNS should resolve:

```text
mystorage.blob.core.windows.net
             |
             v
        10.0.3.10
```

Instead of resolving to the public endpoint.

This allows applications to continue using the normal Azure service hostname while traffic goes through the private endpoint.

---

# 18. Private DNS Zone Example

For Azure Blob Storage, a commonly used Private DNS zone is:

```text
privatelink.blob.core.windows.net
```

Example:

```text
Private DNS Zone
privatelink.blob.core.windows.net
             |
             +-- mystorage
                    |
                    v
                 10.0.3.10
```

The exact record management can be handled automatically when creating a Private Endpoint with suitable DNS integration.

---

# 19. DNS Resolution Flow with Private Endpoint

This is very important for interviews.

```text
Application
     |
     | DNS query
     v
mystorage.blob.core.windows.net
     |
     v
Private DNS resolution
     |
     v
Private IP
10.0.3.10
     |
     v
Private Endpoint
     |
     v
Azure Storage
```

The application does not have to hardcode:

```text
10.0.3.10
```

It continues using:

```text
mystorage.blob.core.windows.net
```

---

# 20. Private DNS Zone Link

Creating a Private DNS Zone alone is not enough.

The zone normally needs to be linked to the VNet where the clients live.

Example:

```text
Private DNS Zone
       |
       |
       v
VNet Link
       |
       v
Application VNet
       |
       +-- VM
       +-- AKS
       +-- Private Endpoint
```

This allows resources in the linked VNet to resolve records from the private DNS zone.

---

# 21. Hub-Spoke DNS Example

Suppose we have:

```text
             Hub VNet
                |
          DNS infrastructure
                |
       +--------+--------+
       |                 |
       v                 v
   Spoke VNet 1      Spoke VNet 2
       |                 |
      VM                AKS
```

A centralized DNS architecture can be used for larger environments.

For example:

```text
Application
    |
    v
Central DNS Resolver
    |
    +---- Private DNS
    |
    +---- Corporate DNS
    |
    +---- Public DNS
```

The exact implementation depends on the organization's DNS architecture.

---

# 22. Azure-Provided DNS

Azure provides a DNS service for VNet resources.

The well-known Azure-provided DNS IP is:

```text
168.63.129.16
```

It is a special Azure platform virtual IP used for several platform functions, including DNS.

For basic Azure VM DNS resolution:

```text
VM
 |
 v
Azure DNS
 |
 v
Hostname resolution
```

---

# 23. Custom DNS

You can configure a VNet to use custom DNS servers.

For example:

```text
VNet
 |
 +-- Custom DNS
       |
       +-- Corporate DNS
       +-- Domain Controller
       +-- DNS appliance
```

Example:

```text
VM
 |
 v
Custom DNS Server
 |
 +---- Corporate domain
 |
 +---- Azure/private names
 |
 +---- Internet names
```

This is common in enterprise environments.

---

# 24. Azure DNS vs Custom DNS

| Feature                     | Azure-provided DNS                 | Custom DNS |
| --------------------------- | ---------------------------------- | ---------- |
| Managed by Azure            | Yes                                | No         |
| Configuration effort        | Low                                | Higher     |
| Enterprise DNS integration  | Limited                            | Flexible   |
| Corporate domain resolution | Usually requires additional design | Yes        |
| Centralized DNS control     | Limited                            | Yes        |

---

# 25. DNS in a VM

On a Linux VM, check DNS configuration:

```bash
cat /etc/resolv.conf
```

You may see a DNS nameserver entry.

Test DNS:

```bash
nslookup google.com
```

or:

```bash
dig google.com
```

or:

```bash
getent hosts google.com
```

---

# 26. Basic DNS Troubleshooting

Suppose:

```text
Application cannot connect to:
api.example.com
```

Don't immediately troubleshoot the application.

First check:

```bash
nslookup api.example.com
```

Then:

```bash
dig api.example.com
```

If DNS does not return the expected IP, investigate DNS.

---

# 27. DNS Troubleshooting Flow

Use this mental model:

```text
Application
     |
     v
Hostname
     |
     v
DNS Resolution
     |
     +---- FAIL → DNS problem
     |
     v
IP Address
     |
     v
Network Connectivity
     |
     v
Application
```

This separates:

```text
DNS problem
```

from:

```text
Network problem
```

and:

```text
Application problem
```

---

# 28. Production Scenario — Private Endpoint

### Problem

An application cannot access Azure Key Vault after a Private Endpoint was created.

Architecture:

```text
Application
    |
    v
Key Vault hostname
    |
    X
DNS
```

First check:

```bash
nslookup <key-vault-hostname>
```

If the name resolves to the public IP instead of the Private Endpoint IP, investigate:

```text
Private DNS Zone
Private DNS Zone Link
VNet
DNS configuration
```

---

# 29. Production Scenario — DNS Works but Application Fails

Suppose:

```bash
nslookup api.example.com
```

returns:

```text
10.0.2.10
```

DNS is working.

But:

```bash
curl https://api.example.com
```

fails.

Now investigate:

```text
NSG
Route table
Firewall
Application Gateway
Backend application
Port
TLS
```

The important point:

> Successful DNS resolution does not mean the application connection will succeed.

---

# 30. Production Scenario — DNS Fails

Suppose:

```bash
nslookup api.example.com
```

returns:

```text
server can't find api.example.com
```

Then investigate:

```text
DNS server
DNS zone
DNS record
VNet DNS configuration
Private DNS zone link
DNS forwarding
```

Don't start by changing NSG rules because DNS resolution happens before the application can connect to the resolved destination.

---

# 31. Azure CLI — DNS Zones

List public DNS zones:

```bash
az network dns zone list -o table
```

Show a DNS zone:

```bash
az network dns zone show \
  --resource-group <resource-group> \
  --name <dns-zone>
```

List records:

```bash
az network dns record-set list \
  --resource-group <resource-group> \
  --zone-name <dns-zone> \
  -o table
```

---

# 32. Create an Azure Public DNS Zone

Example:

```bash
az network dns zone create \
  --resource-group <resource-group> \
  --name <dns-zone>
```

Example zone:

```text
example.com
```

---

# 33. Create an A Record

Example:

```bash
az network dns record-set a create \
  --resource-group <resource-group> \
  --zone-name <dns-zone> \
  --name api \
  --ttl 300
```

Then add the IP:

```bash
az network dns record-set a add-record \
  --resource-group <resource-group> \
  --zone-name <dns-zone> \
  --record-set-name api \
  --ipv4-address <ip-address>
```

Result:

```text
api.example.com
        |
        v
<ip-address>
```

---

# 34. Private DNS CLI

List Private DNS zones:

```bash
az network private-dns zone list \
  --resource-group <resource-group> \
  -o table
```

Show a Private DNS zone:

```bash
az network private-dns zone show \
  --resource-group <resource-group> \
  --name <private-dns-zone>
```

List records:

```bash
az network private-dns record-set list \
  --resource-group <resource-group> \
  --zone-name <private-dns-zone> \
  -o table
```

---

# 35. Create a Private DNS Zone

Example:

```bash
az network private-dns zone create \
  --resource-group <resource-group> \
  --name <private-dns-zone>
```

Example:

```text
privatelink.blob.core.windows.net
```

---

# 36. Link Private DNS Zone to VNet

Example:

```bash
az network private-dns link vnet create \
  --resource-group <resource-group> \
  --zone-name <private-dns-zone> \
  --name <vnet-link-name> \
  --virtual-network <vnet-resource-id> \
  --registration-enabled false
```

This creates the relationship:

```text
Private DNS Zone
        |
        v
VNet Link
        |
        v
Application VNet
```

---

# 37. Private DNS Zone Groups

Private Endpoint deployments can use a **Private DNS Zone Group** to integrate the Private Endpoint with the appropriate Private DNS zone.

Conceptually:

```text
Private Endpoint
       |
       v
Private DNS Zone Group
       |
       v
Private DNS Zone
       |
       v
DNS Record
```

This helps maintain the DNS relationship between the Private Endpoint and the private DNS zone.

---

# 38. DNS and Private Endpoint — Complete Architecture

```text
                  Application
                       |
                       | DNS Query
                       v
          mystorage.blob.core.windows.net
                       |
                       v
                Private DNS Zone
                       |
                       v
                  Private IP
                   10.0.3.10
                       |
                       v
                Private Endpoint
                       |
                       v
                 Azure Storage
```

This is one of the most important diagrams to remember.

---

# 39. DNS and Application Gateway

DNS is also used with Application Gateway.

Example:

```text
www.example.com
       |
       v
Public DNS
       |
       v
Application Gateway Public IP
       |
       v
Listener
       |
       v
Backend
```

The DNS record points the hostname to the Application Gateway frontend.

---

# 40. DNS and Load Balancer

Similarly:

```text
api.example.com
       |
       v
DNS
       |
       v
Load Balancer Public IP
       |
       v
Backend Pool
```

The client doesn't need to know individual backend VM IPs.

---

# 41. DNS and AKS

DNS is also important in AKS.

Inside Kubernetes:

```text
Pod
 |
 v
Kubernetes DNS
 |
 v
Service
```

AKS applications may also need to resolve:

```text
Azure services
Private Endpoints
Corporate services
External APIs
```

Example:

```text
AKS Pod
   |
   v
DNS
   |
   v
Private Endpoint
   |
   v
Azure Key Vault
```

Therefore, DNS configuration becomes important when AKS uses private Azure services.

---

# 42. DNS Scenario — AKS + Private Endpoint

Suppose:

```text
AKS
 |
 v
Azure Storage Private Endpoint
```

Application uses:

```text
storageaccount.blob.core.windows.net
```

If DNS is correctly configured:

```text
AKS Pod
   |
   v
DNS
   |
   v
Private IP
   |
   v
Private Endpoint
   |
   v
Storage
```

If DNS is incorrectly configured:

```text
AKS Pod
   |
   v
DNS
   |
   v
Public IP
```

The application may fail or violate the intended private network design.

---

# 43. DNS vs NSG

Very common interview question.

### DNS

Answers:

```text
"What IP address belongs to this hostname?"
```

### NSG

Answers:

```text
"Is this network traffic allowed?"
```

Example:

```text
api.example.com
       |
       v
DNS → 10.0.2.10
       |
       v
NSG → Allow/Deny
```

Memory:

```text
DNS = Name → IP

NSG = Allow/Deny
```

---

# 44. DNS vs Route Table

### DNS

```text
Name → IP
```

### Route Table

```text
IP → Next Hop
```

Example:

```text
api.example.com
      |
      v
DNS
      |
      v
10.0.2.10
      |
      v
Route Table
      |
      v
Next Hop
```

Memory:

```text
DNS = Where is the destination?

Route = Where should the packet go next?
```

---

# 45. Scenario-Based Interview Question

### Interviewer:

What happens when you enter `https://www.example.com` in a browser?

### Answer:

> First, the client resolves `www.example.com` through DNS to get the destination IP. Then it establishes network connectivity to that IP and starts the HTTPS connection. After the TLS handshake, the HTTP request is sent to the application endpoint.

---

# 46. Scenario-Based Interview Question

### Interviewer:

What is the difference between Public DNS and Private DNS?

### Answer:

> Public DNS resolves names that need to be reachable through public DNS infrastructure. Private DNS is used for private name resolution inside networks such as Azure VNets. Private DNS is especially important when using Private Endpoints.

---

# 47. Scenario-Based Interview Question

### Interviewer:

Why is Private DNS important with Private Endpoint?

### Answer:

> Private Endpoint gives an Azure service a private IP in the VNet, but applications normally continue using the service hostname. Private DNS makes that hostname resolve to the private endpoint IP instead of the public endpoint.

---

# 48. Scenario-Based Interview Question

### Interviewer:

Your Private Endpoint exists, but the application is still resolving the Azure service to a public IP. What will you check?

### Answer:

> I would check whether the correct Private DNS zone exists and whether it is linked to the application's VNet. I would also verify the DNS zone group and the VNet's DNS configuration, then test resolution using `nslookup` or `dig`.

---

# 49. Scenario-Based Interview Question

### Interviewer:

DNS resolution is successful, but the application cannot connect. What do you check next?

### Answer:

> If DNS returns the expected IP, I move to network troubleshooting. I check NSG rules, route tables, firewall or NVA rules, the destination port, and finally the application itself.

---

# 50. Scenario-Based Interview Question

### Interviewer:

How do you troubleshoot a DNS issue on a Linux VM?

### Answer:

> I first check `/etc/resolv.conf` to see which DNS server the VM is using. Then I use `nslookup`, `dig`, or `getent hosts` to test resolution. If it fails, I check the DNS server, private DNS zone, VNet link, custom DNS configuration, and forwarding configuration.

---

# 51. Scenario-Based Interview Question

### Interviewer:

What is an A record?

### Answer:

> An A record maps a hostname to an IPv4 address. For example, `api.example.com` can resolve to `10.0.2.10`.

---

# 52. Scenario-Based Interview Question

### Interviewer:

What is a CNAME?

### Answer:

> A CNAME maps one hostname to another hostname. For example, `app.example.com` can point to `myapp.azurewebsites.net`.

---

# 53. Scenario-Based Interview Question

### Interviewer:

Why would a company use custom DNS instead of Azure-provided DNS?

### Answer:

> Enterprise environments may need centralized DNS, corporate domain resolution, custom forwarding rules, or integration with on-premises DNS. In those cases, the VNet can be configured to use custom DNS servers.

---

# 54. Senior-Level Troubleshooting Scenario

### Problem

An application in Azure cannot access Key Vault through a Private Endpoint.

You know:

```text
Private Endpoint = Created
Key Vault = Healthy
Application = Running
```

What would you check?

### Troubleshooting flow

```text
Application
    |
    v
Key Vault hostname
    |
    v
DNS resolution
    |
    +---- Public IP? → Check Private DNS
    |
    v
Private IP
    |
    v
Private Endpoint
    |
    v
Network
    |
    +---- NSG
    +---- Route
    +---- Firewall
    |
    v
Key Vault
```

Commands:

```bash
nslookup <key-vault-hostname>
```

Then:

```bash
dig <key-vault-hostname>
```

Check Private DNS zones:

```bash
az network private-dns zone list \
  --resource-group <resource-group> \
  -o table
```

Check VNet links:

```bash
az network private-dns link vnet list \
  --resource-group <resource-group> \
  --zone-name <private-dns-zone> \
  -o table
```

---

# 55. Production DNS Troubleshooting Mental Model

When an application cannot reach a hostname, follow:

```text
1. Can I resolve the hostname?
             |
             v
2. Did DNS return the expected IP?
             |
             v
3. Can I reach that IP?
             |
             v
4. Is the required port open?
             |
             v
5. Is the application responding?
```

This prevents randomly changing Azure networking configurations.

---

# 56. Hands-On Lab

Use existing Azure resources where possible.

### Step 1 — Test DNS from a Linux VM

```bash
nslookup google.com
```

Then:

```bash
dig google.com
```

Then:

```bash
getent hosts google.com
```

---

### Step 2 — Check DNS configuration

```bash
cat /etc/resolv.conf
```

---

### Step 3 — Check Azure DNS zones

```bash
az network dns zone list -o table
```

---

### Step 4 — Check Private DNS zones

```bash
az network private-dns zone list -o table
```

---

### Step 5 — If you have a Private Endpoint

Find its DNS configuration and test the service hostname:

```bash
nslookup <service-hostname>
```

Check whether the result matches the expected private endpoint IP.

---

### Step 6 — Understand the complete flow

Draw:

```text
Application
    |
    v
Hostname
    |
    v
DNS
    |
    v
IP Address
    |
    v
Network
    |
    v
Application
```

For Private Endpoint:

```text
Application
    |
    v
Azure Service Hostname
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
Azure Service
```

---

# 57. Final Comparison

| Component             | Main Purpose                                      |
| --------------------- | ------------------------------------------------- |
| Public DNS            | Public hostname resolution                        |
| Azure DNS             | Host DNS zones in Azure                           |
| Private DNS           | Private name resolution                           |
| A Record              | Name → IPv4                                       |
| AAAA Record           | Name → IPv6                                       |
| CNAME                 | Name → Name                                       |
| NS                    | Authoritative name servers                        |
| MX                    | Mail routing                                      |
| TXT                   | Text/domain verification information              |
| Private DNS Zone Link | Makes private zone available to a VNet            |
| DNS Zone Group        | Helps integrate Private Endpoint with Private DNS |

---

# 58. Final Mental Model

Remember:

```text
DNS
↓
Name → IP
```

Public application:

```text
www.example.com
        ↓
Public DNS
        ↓
Public IP
        ↓
Application Gateway
```

Private application:

```text
app.internal.example.com
        ↓
Private DNS
        ↓
Private IP
        ↓
Private Application
```

Private Endpoint:

```text
Azure Service Hostname
        ↓
Private DNS
        ↓
Private Endpoint IP
        ↓
Azure PaaS Service
```

### One-line interview memory

> **DNS resolves the name, routing decides the path, NSG controls traffic, and the application handles the request.**
