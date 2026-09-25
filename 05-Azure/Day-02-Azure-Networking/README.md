# Day 02 - Azure Networking

## Objective

The goal of Day 2 is to understand Azure networking from a practical DevOps and production perspective.

By the end of this day, I should be able to understand how traffic moves between:

* Internet
* Azure VNet
* Subnets
* Virtual Machines
* AKS
* Load Balancer
* Application Gateway
* Private Endpoints
* Azure Services

I should also be able to troubleshoot common Azure networking issues using Azure Portal, Azure CLI, and practical scenarios.

---

# Day 02 Topics

## 01. Azure VNet

Learn:

* What is a VNet?
* Why do we need a VNet?
* Address space
* CIDR
* VNet and subnet relationship
* VNet design in production
* VNet CLI commands
* Real production scenario
* Interview questions

---

## 02. Azure Subnet

Learn:

* What is a subnet?
* Why do we divide a VNet into subnets?
* Frontend subnet
* Backend subnet
* Database subnet
* Subnet address ranges
* Subnet delegation
* Subnet troubleshooting
* Real production scenario

---

## 03. Network Security Group - NSG

Learn:

* What is NSG?
* Inbound rules
* Outbound rules
* Source and destination
* Port and protocol
* Priority
* NSG association
* Subnet vs NIC association
* Production troubleshooting

---

## 04. Application Security Group - ASG

Learn:

* What is ASG?
* Why use ASG?
* ASG with NSG
* Application-based network rules
* Production example

---

## 05. Public IP vs Private IP

Learn:

* Private IP
* Public IP
* Dynamic vs Static
* When a VM needs a public IP
* Why production backend servers normally don't need public IPs
* Internet traffic flow

---

## 06. Network Interface - NIC

Learn:

* What is NIC?
* NIC and VM relationship
* Private IP association
* Public IP association
* NSG association
* Troubleshooting NIC-related issues

---

## 07. Route Table / User Defined Route - UDR

Learn:

* How Azure decides where traffic goes
* System routes
* User Defined Routes
* Next hop
* Firewall routing
* Forced tunneling
* Production troubleshooting

---

## 08. VNet Peering

Learn:

* What is VNet Peering?
* Same-region peering
* Cross-region peering
* Traffic flow
* Hub-Spoke architecture
* Common peering problems

---

## 09. Private Endpoint

Learn:

* What is Private Endpoint?
* Private IP for Azure PaaS services
* Private access to Storage
* Private access to Key Vault
* Private DNS
* Production architecture
* Troubleshooting

---

## 10. Service Endpoint

Learn:

* What is Service Endpoint?
* How it differs from Private Endpoint
* Subnet-based access
* Storage/SQL examples
* When it is used

---

## 11. Azure Load Balancer

Learn:

* What is Load Balancer?
* Layer 4 traffic
* Frontend IP
* Backend pool
* Health probe
* Load balancing rule
* Inbound NAT
* Production scenario

---

## 12. Application Gateway + WAF

Learn:

* What is Application Gateway?
* Layer 7 routing
* Backend pools
* Listeners
* Routing rules
* Health probes
* TLS termination
* Path-based routing
* WAF
* Application Gateway vs Load Balancer

---

## 13. NAT Gateway

Learn:

* What is NAT Gateway?
* Why private resources need outbound connectivity
* SNAT
* Static outbound public IP
* Production use case
* NAT Gateway vs Load Balancer outbound connectivity

---

## 14. Azure DNS

Learn:

* DNS basics
* Public DNS
* Private DNS
* Private DNS Zone
* DNS resolution
* Private Endpoint + Private DNS
* Production troubleshooting

---

# 15. Production Network Architecture

Build a real-world architecture:

```text
                         Internet
                            |
                            v
                   Application Gateway
                         + WAF
                            |
                            v
                    Frontend Subnet
                            |
                            v
                     Backend Subnet
                            |
                            v
                      Private Endpoint
                            |
                            v
                 Azure Database / Storage
```

We will expand this architecture with:

* VNet
* Subnets
* NSG
* Route Table
* Application Gateway
* Load Balancer
* Private Endpoint
* Private DNS
* NAT Gateway
* Azure Firewall concepts
* Hub-Spoke networking

---

# 16. Scenario-Based Questions

Practice real production scenarios such as:

### Scenario 1

A VM is running but the application is not reachable from the internet.

How would you troubleshoot it?

### Scenario 2

Two VMs are in different subnets but cannot communicate.

What would you check?

### Scenario 3

An application can access the internet but cannot access Azure Storage privately.

What would you check?

### Scenario 4

Application Gateway shows backend servers as unhealthy.

How would you troubleshoot?

### Scenario 5

A private endpoint is created successfully but the application cannot resolve the service hostname.

What would you check?

### Scenario 6

A backend VM should have internet access for updates but should not have a public IP.

How would you design it?

---

# 17. Day-02 Mini Lab

The final lab will combine the important networking concepts.

Target architecture:

```text
                    Internet
                       |
                       v
              Application Gateway
                       |
                       v
                  Frontend
                   Subnet
                       |
                       v
                   Backend
                   Subnet
                       |
                       v
                Private Endpoint
                       |
                       v
                Azure Storage
```

The lab will include:

* Resource Group
* VNet
* Multiple Subnets
* NSG
* VM or test workload
* Private Endpoint
* Private DNS
* Public/Private IP validation
* Network troubleshooting
* Azure CLI commands
* Resource ID verification
* Cleanup

---

# Day-02 Learning Method

For every topic, follow this sequence:

```text
1. Understand the concept
        ↓
2. Understand a real production example
        ↓
3. Run Azure CLI commands
        ↓
4. Observe the output
        ↓
5. Troubleshoot a failure scenario
        ↓
6. Explain it verbally
        ↓
7. Save notes to GitHub
```

---

# Day-02 Interview Goal

After completing Day 2, I should be able to explain a production Azure network like this:

```text
User
 |
 v
Internet
 |
 v
Application Gateway + WAF
 |
 v
Frontend Subnet
 |
 v
Backend Subnet
 |
 v
Private Endpoint
 |
 v
Azure PaaS Service
```

And explain:

* How traffic enters Azure
* How traffic is routed
* How NSG controls traffic
* How private resources communicate
* How outbound internet access works
* How Application Gateway distributes HTTP/HTTPS traffic
* How Private Endpoint provides private access
* How DNS resolves private services
* How to troubleshoot connectivity problems

---

# Day-02 Final Mental Model

Remember:

```text
VNet       → Network boundary
Subnet     → Network segmentation
NSG        → Traffic filtering
ASG        → Application grouping
NIC        → Network connection for VM
IP         → Network address
UDR        → Custom traffic path
Peering    → VNet-to-VNet connectivity
LB         → Layer 4 load balancing
App GW     → Layer 7 HTTP/HTTPS routing
WAF        → Web attack protection
Private EP → Private access to Azure service
NAT GW     → Controlled outbound internet access
DNS        → Name-to-IP resolution
```

---

# Day-02 Outcome

At the end of Day 2, I should be able to look at an Azure networking architecture and understand:

```text
Who is sending traffic?
        ↓
Where is the traffic entering?
        ↓
Which subnet is involved?
        ↓
Is NSG allowing it?
        ↓
Which route will it take?
        ↓
Is the destination public or private?
        ↓
Is DNS resolving correctly?
        ↓
Is the backend healthy?
        ↓
Where exactly is the traffic failing?
```
