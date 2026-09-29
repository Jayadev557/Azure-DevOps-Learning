# Azure Load Balancer

## 1. What is Azure Load Balancer?

Azure Load Balancer is a **Layer 4 (L4) load balancer**.

It distributes incoming TCP or UDP traffic across multiple backend VMs or VM Scale Set instances.

Simple flow:

```text
Client
   |
   v
Load Balancer Frontend IP
   |
   v
Load Balancing Rule
   |
   v
Backend Pool
   |
   +----> VM1
   |
   +----> VM2
   |
   +----> VM3
```

The main purpose is to provide:

* High availability
* Traffic distribution
* Backend health checking
* Scalability

---

# 2. Main Components

An Azure Load Balancer commonly contains these components:

```text
Load Balancer
│
├── Frontend IP
│
├── Backend Pool
│
├── Health Probe
│
├── Load Balancing Rule
│
└── Optional NAT / Outbound Rules
```

### Frontend IP

This is the IP address clients connect to.

Example:

```text
Internet
   |
   v
20.x.x.x
   |
   v
Azure Load Balancer
```

The frontend can use:

* Public IP
* Private IP

---

# 3. Backend Pool

Backend pool contains the resources that receive traffic.

Example:

```text
Load Balancer
      |
      v
Backend Pool
   /    |    \
 VM1   VM2   VM3
```

Backend targets can commonly be:

* Virtual Machines
* VM Scale Set instances

The Load Balancer distributes traffic among healthy backend instances.

---

# 4. Health Probe

Health probe checks whether a backend is available.

Example:

```text
Load Balancer
      |
      | TCP 80 probe
      v
   VM1 ---> Healthy
   VM2 ---> Healthy
   VM3 ---> Unhealthy
```

If VM3 fails the health probe, the Load Balancer stops sending new traffic to that unhealthy backend.

Example:

```text
Health Probe
    |
    +---- VM1 → Healthy
    |
    +---- VM2 → Healthy
    |
    +---- VM3 → Unhealthy
```

This is important because a VM can be running while the actual application is unavailable.

For example:

```text
VM = Running
Application = Down
Port 80 = Not listening
```

The health probe helps identify this condition.

---

# 5. Load Balancing Rule

The load balancing rule defines how traffic arriving on the frontend is sent to the backend pool.

Example:

```text
Frontend:
20.x.x.x:80

        |
        v

Load Balancing Rule
TCP 80 → Backend TCP 80

        |
        v

Backend Pool
VM1
VM2
VM3
```

A rule normally defines things such as:

* Frontend IP
* Frontend port
* Backend pool
* Backend port
* Protocol
* Health probe

---

# 6. Public Load Balancer

A Public Load Balancer uses a public frontend IP.

Typical architecture:

```text
Internet
    |
    v
Public IP
    |
    v
Azure Load Balancer
    |
    v
Backend Pool
  /   |   \
 VM1 VM2 VM3
```

Use it when the application needs an internet-facing Layer 4 endpoint.

Example:

```text
Customer
   |
   v
Public Load Balancer
   |
   +---- Web VM1
   |
   +---- Web VM2
```

---

# 7. Internal Load Balancer

An Internal Load Balancer uses a private IP.

It is normally used for internal applications and services.

Example:

```text
Frontend Tier
      |
      v
Internal Load Balancer
      |
      v
Backend/API Tier
   /       \
 API VM1   API VM2
```

The frontend can communicate with the API using the private IP of the Internal Load Balancer.

---

# 8. Public vs Internal Load Balancer

| Feature         | Public LB          | Internal LB          |
| --------------- | ------------------ | -------------------- |
| Frontend IP     | Public             | Private              |
| Internet-facing | Yes                | No                   |
| Common use      | Public application | Internal application |
| Backend         | VM/VMSS            | VM/VMSS              |
| Layer           | L4                 | L4                   |

Simple memory trick:

```text
Public LB   → Internet → Backend
Internal LB → Private Network → Backend
```

---

# 9. Load Balancer vs Application Gateway

This is a common interview question.

| Feature            | Load Balancer                | Application Gateway      |
| ------------------ | ---------------------------- | ------------------------ |
| Layer              | L4                           | L7                       |
| Protocol           | TCP/UDP                      | HTTP/HTTPS               |
| URL routing        | No                           | Yes                      |
| Path-based routing | No                           | Yes                      |
| Host-based routing | No                           | Yes                      |
| WAF                | No                           | Yes, with WAF capability |
| TLS termination    | Not its primary role         | Yes                      |
| Typical use        | Network-level load balancing | Web application routing  |

Example:

```text
Load Balancer:

Client
  |
  v
TCP/UDP
  |
  v
Backend
```

Application Gateway:

```text
Client
  |
  v
HTTP/HTTPS
  |
  v
Application Gateway
  |
  +---- /api  → API
  |
  +---- /web  → Web
```

Memory trick:

```text
LB  = L4
AGW = L7
```

---

# 10. Load Balancer vs Front Door vs Traffic Manager

| Service             | Main Layer/Method      | Typical Purpose                      |
| ------------------- | ---------------------- | ------------------------------------ |
| Load Balancer       | L4                     | TCP/UDP traffic to backend resources |
| Application Gateway | L7                     | HTTP/HTTPS application routing       |
| Front Door          | Global HTTP/HTTPS edge | Global web application delivery      |
| Traffic Manager     | DNS-based              | DNS traffic distribution             |

Simple mental model:

```text
Load Balancer
→ Inside Azure region/network
→ L4

Application Gateway
→ Web application
→ L7

Front Door
→ Global web traffic
→ HTTP/HTTPS edge

Traffic Manager
→ DNS-based traffic routing
```

---

# 11. Azure Load Balancer Traffic Flow

Suppose we have three backend VMs:

```text
Client
   |
   v
Frontend IP
   |
   v
Load Balancer
   |
   v
Health Probe
   |
   v
Backend Pool
  /   |   \
VM1  VM2  VM3
```

Suppose:

```text
VM1 → Healthy
VM2 → Healthy
VM3 → Unhealthy
```

Traffic is distributed only among the healthy backend instances.

```text
Client
   |
   v
Load Balancer
   |
   +----> VM1
   |
   +----> VM2
   |
   X----> VM3
```

---

# 12. Azure CLI — Load Balancer

List Load Balancers:

```bash
az network lb list -o table
```

Show a specific Load Balancer:

```bash
az network lb show \
  --resource-group <resource-group> \
  --name <load-balancer-name>
```

List frontend IP configurations:

```bash
az network lb frontend-ip list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  -o table
```

List backend pools:

```bash
az network lb address-pool list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  -o table
```

List health probes:

```bash
az network lb probe list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  -o table
```

List load balancing rules:

```bash
az network lb rule list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  -o table
```

---

# 13. Create a Load Balancer

For a public Load Balancer, create a frontend public IP first.

Example:

```bash
az network public-ip create \
  --resource-group <resource-group> \
  --name <public-ip-name> \
  --sku Standard
```

Create the Load Balancer:

```bash
az network lb create \
  --resource-group <resource-group> \
  --name <load-balancer-name> \
  --sku Standard \
  --public-ip-address <public-ip-name> \
  --frontend-ip-name <frontend-name> \
  --backend-pool-name <backend-pool-name>
```

The exact backend resources and rule configuration depend on the architecture.

---

# 14. Create a Health Probe

Example TCP probe:

```bash
az network lb probe create \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  --name <probe-name> \
  --protocol Tcp \
  --port 80
```

For HTTP-based health checking:

```bash
az network lb probe create \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  --name <probe-name> \
  --protocol Http \
  --port 80 \
  --path /
```

The application must actually respond on the configured port/path.

---

# 15. Create a Load Balancing Rule

Example:

```bash
az network lb rule create \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  --name <rule-name> \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name <frontend-name> \
  --backend-pool-name <backend-pool-name> \
  --probe-name <probe-name>
```

This means:

```text
Frontend TCP 80
      |
      v
Load Balancing Rule
      |
      v
Backend TCP 80
```

---

# 16. Production Example

Suppose a customer-facing application has three web VMs.

```text
                    Internet
                       |
                       v
               Public IP
                       |
                       v
              Azure Load Balancer
                       |
                Load Balancing Rule
                       |
                       v
                Backend Pool
              /       |       \
             /        |        \
           VM1       VM2       VM3
          Web       Web       Web
```

If VM2 becomes unhealthy:

```text
VM1 → Healthy
VM2 → Unhealthy
VM3 → Healthy
```

Traffic continues to:

```text
VM1
VM3
```

This improves availability without requiring the client to know individual VM IP addresses.

---

# 17. Troubleshooting Load Balancer

Suppose the user reports:

```text
Load Balancer public IP is not responding.
```

I would troubleshoot in this order:

### Step 1 — Check frontend IP

```bash
az network lb frontend-ip list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name>
```

### Step 2 — Check backend pool

```bash
az network lb address-pool list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name>
```

### Step 3 — Check health probe

```bash
az network lb probe list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name>
```

### Step 4 — Check application

On the backend VM:

```bash
ss -lntp
```

Verify that the application is listening on the expected port.

Example:

```text
0.0.0.0:80
```

### Step 5 — Check NSG

Verify that the NSG allows the required traffic.

### Step 6 — Check Load Balancing Rule

```bash
az network lb rule list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name>
```

Verify:

```text
Frontend port
Backend port
Backend pool
Health probe
Protocol
```

---

# 18. Common Production Issue

### Problem

Load Balancer is created, but backend VMs are not receiving traffic.

### Possible causes

```text
Health probe failing
       ↓
Backend marked unhealthy
       ↓
No new traffic to backend
```

Check:

```text
1. Application is running
2. Correct backend port
3. Health probe port
4. Health probe path if HTTP
5. NSG rules
6. Backend pool membership
7. Load balancing rule
```

---

# 19. Interview Answer — What is Azure Load Balancer?

> Azure Load Balancer is a Layer 4 load balancing service that distributes TCP or UDP traffic across healthy backend VMs or VM Scale Sets. It uses a frontend IP, backend pool, health probes, and load-balancing rules. In production, I use it when I need highly available network-level traffic distribution.

---

# 20. Interview Answer — Public vs Internal Load Balancer

> A Public Load Balancer has a public frontend IP and is used for internet-facing traffic. An Internal Load Balancer uses a private IP and is mainly used for communication inside the Azure network. For example, I can use a public LB for the web tier and an internal LB for backend services.

---

# 21. Interview Answer — Load Balancer vs Application Gateway

> Azure Load Balancer works at Layer 4 and handles TCP or UDP traffic. Application Gateway works at Layer 7 and understands HTTP/HTTPS, so it can do host-based and path-based routing and WAF functions. I choose based on whether I need network-level or web-application-level traffic management.

---

# 22. Scenario-Based Interview Question

### Interviewer:

Your Load Balancer is working, but one backend VM is not receiving traffic. What will you check?

### Answer:

> First I check the health probe status and whether the backend is marked healthy. Then I verify the application is listening on the expected port, followed by NSG rules, backend pool membership, and the load-balancing rule. If the probe is failing, I fix the application or probe configuration first.

---

# 23. Hands-On Practice

Use existing Azure resources where possible.

### Step 1 — Check existing Load Balancers

```bash
az network lb list -o table
```

### Step 2 — Inspect the Load Balancer

```bash
az network lb show \
  --resource-group <resource-group> \
  --name <load-balancer-name>
```

### Step 3 — Check backend pools

```bash
az network lb address-pool list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  -o table
```

### Step 4 — Check health probes

```bash
az network lb probe list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  -o table
```

### Step 5 — Check rules

```bash
az network lb rule list \
  --resource-group <resource-group> \
  --lb-name <load-balancer-name> \
  -o table
```

### Step 6 — Understand the complete flow

Draw this yourself:

```text
Client
  |
  v
Frontend IP
  |
  v
Load Balancing Rule
  |
  v
Backend Pool
  |
  +----> VM1
  |
  +----> VM2
  |
  +----> VM3

Health Probe
  |
  +----> VM1 ✓
  +----> VM2 ✓
  +----> VM3 ✗
```

---

# 24. Final Mental Model

Remember Azure Load Balancer using four words:

```text
FRONTEND
    ↓
RULE
    ↓
POOL
    ↓
HEALTH
```

Or simply:

```text
Client
  ↓
Frontend IP
  ↓
Load Balancing Rule
  ↓
Healthy Backend Pool
  ↓
VM / VMSS
```

### One-line memory trick

```text
Azure Load Balancer = L4 traffic distribution across healthy backends.
```
