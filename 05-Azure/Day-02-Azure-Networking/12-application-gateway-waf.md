# Azure Application Gateway and WAF

## 1. What is Azure Application Gateway?

Azure Application Gateway is an **Azure Layer 7 (L7) web traffic load balancer**.

Unlike Azure Load Balancer, Application Gateway understands **HTTP and HTTPS traffic**.

It can make routing decisions based on:

* Host name
* URL path
* HTTP headers
* HTTP/HTTPS requests

Example:

```text
                    Internet
                       |
                       v
              Application Gateway
                    L7
                       |
          +------------+------------+
          |                         |
       /api/*                    /web/*
          |                         |
          v                         v
      API Backend              Web Backend
       VM/VMSS                   VM/VMSS
```

---

# 2. Simple Real-World Example

Suppose we have one public domain:

```text
myapp.example.com
```

And the application has:

```text
myapp.example.com/
        |
        +---- /api
        |
        +---- /payment
        |
        +---- /admin
```

We can use Application Gateway to route requests:

```text
/api/*
    ↓
API Servers

/payment/*
    ↓
Payment Servers

/admin/*
    ↓
Admin Servers
```

The client still uses the same public endpoint.

---

# 3. Why Do We Need Application Gateway?

Suppose we have:

```text
Client
   |
   v
Application Gateway
   |
   +---- /api      → API backend
   |
   +---- /frontend → Frontend backend
   |
   +---- /payment  → Payment backend
```

Without application-level routing, we would need separate entry points or another routing mechanism.

Application Gateway gives us:

* Layer 7 routing
* TLS termination
* Host-based routing
* Path-based routing
* Health probes
* Backend pools
* WAF capability

---

# 4. Main Application Gateway Components

Think about Application Gateway like this:

```text
Application Gateway
│
├── Frontend IP
│
├── Listener
│
├── Routing Rule
│
├── Backend Pool
│
├── Backend HTTP Settings
│
├── Health Probe
│
└── WAF Policy
```

These components work together.

---

# 5. Frontend IP

The frontend IP is where clients connect.

Example:

```text
Internet
   |
   v
Public IP
   |
   v
Application Gateway
```

Application Gateway can also have a private frontend IP depending on the architecture.

For a public web application, a common design is:

```text
Internet
   |
   v
Public IP
   |
   v
Application Gateway
```

---

# 6. Listener

A listener receives incoming HTTP/HTTPS requests.

For example:

```text
https://www.example.com
```

The listener can be configured for:

```text
Protocol = HTTPS
Port = 443
Hostname = www.example.com
```

Flow:

```text
Client
   |
   | HTTPS :443
   v
Listener
```

The listener basically answers:

> "What type of incoming request am I accepting?"

---

# 7. Backend Pool

Backend pool contains the application servers.

Example:

```text
Application Gateway
        |
        v
Backend Pool
   /     |     \
 VM1   VM2    VM3
```

Backend targets can include resources such as:

* VMs
* VM Scale Sets
* IP addresses
* App Services
* Other supported backend targets

---

# 8. Backend HTTP Settings

Backend HTTP settings define how Application Gateway communicates with the backend.

For example:

```text
Frontend
HTTPS :443

        |
        v

Application Gateway

        |
        | HTTP :8080
        v

Backend Application
```

The frontend and backend protocols do not necessarily have to be the same.

For example:

```text
Client
HTTPS :443
    |
    v
Application Gateway
    |
    | HTTP :8080
    v
Backend
```

This is a common place where **TLS termination** becomes useful.

---

# 9. TLS Termination

Suppose the client sends:

```text
HTTPS
   |
   v
Application Gateway
```

Application Gateway can terminate TLS at the gateway.

Then it can forward traffic to the backend.

Example:

```text
Client
   |
   | HTTPS
   v
Application Gateway
   |
   | HTTP
   v
Backend
```

The gateway handles the client-side TLS connection.

Depending on security requirements, you can also configure HTTPS between Application Gateway and the backend.

---

# 10. Path-Based Routing

This is one of the most important Application Gateway features.

Suppose:

```text
https://example.com/api/*
https://example.com/images/*
https://example.com/admin/*
```

We can configure:

```text
/api/*
     ↓
API Backend

/images/*
     ↓
Image Backend

/admin/*
     ↓
Admin Backend
```

Example:

```text
Client
   |
   v
Application Gateway
   |
   +---- /api/* ------> API Pool
   |
   +---- /images/* ---> Image Pool
   |
   +---- /admin/* ----> Admin Pool
```

This is called **path-based routing**.

---

# 11. Host-Based Routing

Application Gateway can also route based on hostname.

Suppose the company has:

```text
api.example.com
www.example.com
admin.example.com
```

We can configure:

```text
api.example.com
       ↓
API Backend

www.example.com
       ↓
Web Backend

admin.example.com
       ↓
Admin Backend
```

Example:

```text
                 Application Gateway
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
   api.example.com  www.example.com  admin.example.com
          |             |             |
          v             v             v
       API Pool      Web Pool      Admin Pool
```

This is called **host-based routing**.

---

# 12. Health Probe

Application Gateway uses health probes to determine whether backend servers are healthy.

Example:

```text
Application Gateway
       |
       +---- VM1 → Healthy
       |
       +---- VM2 → Healthy
       |
       +---- VM3 → Unhealthy
```

If VM3 fails the health probe, Application Gateway stops sending new requests to that backend.

A probe can check a specific endpoint such as:

```text
/health
```

Example:

```text
GET /health
```

Expected response:

```text
200 OK
```

---

# 13. Why Custom Health Probes Are Useful

Suppose the application is running but the actual business service is broken.

For example:

```text
VM = Running
Application process = Running
Database connection = Failed
```

A simple TCP probe might still consider the server available.

Instead, we can use:

```text
/health
```

and make the application return failure when critical dependencies are unavailable.

Example:

```text
GET /health

Database = OK
Cache = OK
Application = OK

Response:
200 OK
```

If the database is unavailable:

```text
Database = Failed

Response:
503 Service Unavailable
```

Then the gateway can stop sending traffic to that backend.

---

# 14. WAF — Web Application Firewall

WAF stands for **Web Application Firewall**.

Azure Application Gateway can be used with WAF capability to help protect web applications against common web attacks.

Example:

```text
Internet
   |
   v
Application Gateway + WAF
   |
   v
Application
```

WAF inspects HTTP/HTTPS requests.

It can help protect against threats such as:

* SQL injection
* Cross-site scripting (XSS)
* Other common web attack patterns

---

# 15. Normal Application Gateway vs WAF

Think of it like this:

```text
Application Gateway
        |
        +---- Traffic routing
        +---- Health probes
        +---- TLS termination
        +---- HTTP/HTTPS handling


Application Gateway + WAF
        |
        +---- Everything above
        |
        +---- Web request inspection
        +---- WAF protection
```

WAF is therefore a **security capability on top of Application Gateway**, not a replacement for network security controls.

---

# 16. WAF Detection vs Prevention

A WAF policy can operate in different modes.

### Detection

The WAF detects and logs suspicious requests.

```text
Request
   |
   v
WAF
   |
   +---- Suspicious → Log
   |
   v
Backend
```

### Prevention

The WAF can block requests that match configured protection rules.

```text
Request
   |
   v
WAF
   |
   +---- Malicious → BLOCK
   |
   +---- Valid → Backend
```

For production, the exact mode and rule configuration should be chosen according to the application's requirements and tested before rollout.

---

# 17. Application Gateway Architecture

A typical production architecture:

```text
                         Internet
                            |
                            v
                     Public IP :443
                            |
                            v
                 Application Gateway
                         + WAF
                            |
              +-------------+-------------+
              |             |             |
           /api/*       /web/*        /admin/*
              |             |             |
              v             v             v
          API Pool       Web Pool      Admin Pool
          VM/VMSS        VM/VMSS        VM/VMSS
```

Backend servers can remain on private IPs.

This is a common production design.

---

# 18. Application Gateway vs Load Balancer

This is one of the most common interview questions.

| Feature         | Azure Load Balancer  | Application Gateway      |
| --------------- | -------------------- | ------------------------ |
| Layer           | L4                   | L7                       |
| Traffic         | TCP/UDP              | HTTP/HTTPS               |
| URL routing     | No                   | Yes                      |
| Path routing    | No                   | Yes                      |
| Host routing    | No                   | Yes                      |
| WAF             | No                   | Yes, with WAF capability |
| TLS termination | Not its primary role | Yes                      |
| Web-aware       | No                   | Yes                      |

Memory trick:

```text
Load Balancer
→ L4
→ TCP/UDP

Application Gateway
→ L7
→ HTTP/HTTPS
→ URL/host routing
```

---

# 19. Application Gateway vs Azure Front Door

Both can handle HTTP/HTTPS traffic, but their typical placement and scope differ.

```text
Application Gateway
→ Regional application traffic
→ VNet-integrated architecture
→ Private backend access

Front Door
→ Global HTTP/HTTPS entry point
→ Microsoft global edge network
→ Global routing and acceleration
```

Example:

```text
Users around the world
          |
          v
     Azure Front Door
          |
     +----+----+
     |         |
     v         v
 Region 1   Region 2
     |         |
     v         v
App Gateway App Gateway
```

Application Gateway can then handle regional application routing.

---

# 20. Application Gateway vs Traffic Manager

Traffic Manager is primarily **DNS-based traffic routing**.

Application Gateway is an **HTTP/HTTPS reverse proxy and Layer 7 load balancer**.

Example:

```text
Traffic Manager
      |
      +---- Region 1
      |
      +---- Region 2
```

Application Gateway:

```text
Application Gateway
      |
      +---- /api
      |
      +---- /web
      |
      +---- /admin
```

Simple memory:

```text
Traffic Manager → DNS routing

Application Gateway → HTTP/HTTPS routing
```

---

# 21. CLI — Check Application Gateways

List Application Gateways:

```bash
az network application-gateway list -o table
```

Show a specific Application Gateway:

```bash
az network application-gateway show \
  --resource-group <resource-group> \
  --name <application-gateway-name>
```

List frontend IP configurations:

```bash
az network application-gateway frontend-ip list \
  --resource-group <resource-group> \
  --gateway-name <application-gateway-name> \
  -o table
```

List listeners:

```bash
az network application-gateway frontend-port list \
  --resource-group <resource-group> \
  --gateway-name <application-gateway-name> \
  -o table
```

Inspect backend pools:

```bash
az network application-gateway address-pool list \
  --resource-group <resource-group> \
  --gateway-name <application-gateway-name> \
  -o table
```

Check backend health:

```bash
az network application-gateway show-backend-health \
  --resource-group <resource-group> \
  --name <application-gateway-name>
```

This command is particularly useful during troubleshooting.

---

# 22. Production Troubleshooting Scenario

### Problem

Users report:

```text
https://example.com/api
```

returns:

```text
502 Bad Gateway
```

Do not immediately assume the Application Gateway itself is broken.

Check the complete path:

```text
Client
  |
  v
Application Gateway
  |
  v
Listener
  |
  v
Routing Rule
  |
  v
Backend Pool
  |
  v
Backend HTTP Settings
  |
  v
Health Probe
  |
  v
Application
```

---

# 23. 502 Troubleshooting Flow

### Step 1 — Check backend health

```bash
az network application-gateway show-backend-health \
  --resource-group <resource-group> \
  --name <application-gateway-name>
```

### Step 2 — Check application

On the backend:

```bash
ss -lntp
```

Verify the expected port is listening.

Example:

```text
0.0.0.0:8080
```

### Step 3 — Check backend HTTP settings

Verify:

```text
Protocol
Port
Host name
Timeout
```

### Step 4 — Check health probe

Verify:

```text
Probe protocol
Probe port
Probe path
Expected response
```

Example:

```text
GET /health
→ 200 OK
```

### Step 5 — Check NSG

Verify traffic from the Application Gateway subnet to the backend is allowed.

### Step 6 — Check routing

Check route tables and any custom routes affecting the Application Gateway subnet or backend subnet.

---

# 24. Common Production Issue — Wrong Backend Port

Suppose:

```text
Application Gateway
       |
       | HTTP :80
       v
Backend
       |
       X
Application actually listening on :8080
```

The health probe fails.

Correct configuration:

```text
Application Gateway
       |
       | HTTP :8080
       v
Backend Application
```

This is a common configuration issue.

---

# 25. Common Production Issue — NSG

Suppose the backend VM is healthy locally:

```text
curl localhost:8080
→ 200 OK
```

But Application Gateway reports the backend as unhealthy.

Then check:

```text
Application Gateway subnet
        |
        | Traffic
        v
Backend subnet / NIC
        |
        v
NSG
```

The NSG must allow the required application traffic.

---

# 26. Scenario-Based Interview Question

### Interviewer:

You have two applications:

```text
example.com/api
example.com/web
```

You want `/api` to go to API servers and `/web` to go to web servers. Which Azure service would you use?

### Answer:

> I would use Azure Application Gateway because it works at Layer 7 and supports path-based routing. I can configure `/api/*` to the API backend pool and `/web/*` to the web backend pool.

---

# 27. Scenario-Based Interview Question

### Interviewer:

Why would you use Application Gateway instead of Azure Load Balancer?

### Answer:

> If I only need TCP or UDP load balancing, I would use Azure Load Balancer. If I need HTTP or HTTPS features like URL-based routing, host-based routing, TLS termination, or WAF, I would use Application Gateway.

---

# 28. Scenario-Based Interview Question

### Interviewer:

Your Application Gateway backend shows unhealthy. What will you check?

### Answer:

> First I check the Application Gateway backend health and health probe configuration. Then I verify the backend application is listening on the correct port and path, followed by NSG rules, routing, and backend HTTP settings.

---

# 29. Scenario-Based Interview Question

### Interviewer:

The application works directly on the VM but returns 502 through Application Gateway. What could be wrong?

### Answer:

> I would check the backend health first. A common cause is a mismatch in backend port, protocol, host header, or health probe configuration. I would also verify NSG and routing between the Application Gateway subnet and backend subnet.

---

# 30. Scenario-Based Interview Question

### Interviewer:

Why do we need a health probe?

### Answer:

> The health probe tells Application Gateway whether a backend can actually receive traffic. If a VM is running but its application or health endpoint is unavailable, the probe can mark it unhealthy and prevent new requests from being sent there.

---

# 31. Scenario-Based Interview Question

### Interviewer:

What is WAF and why would you use it?

### Answer:

> WAF is Web Application Firewall capability used with Application Gateway to inspect web requests and protect applications against common web attacks. For example, it can detect or block suspicious SQL injection or XSS patterns based on the configured WAF policy.

---

# 32. Scenario-Based Interview Question

### Interviewer:

Can Application Gateway route traffic based on hostname?

### Answer:

> Yes. Application Gateway supports host-based routing. For example, `api.example.com` can route to an API backend pool while `www.example.com` routes to a frontend backend pool.

---

# 33. Scenario-Based Interview Question

### Interviewer:

Can Application Gateway route based on URL path?

### Answer:

> Yes. It supports path-based routing. For example, `/api/*` can go to the API backend and `/payment/*` can go to the payment backend.

---

# 34. Scenario-Based Interview Question

### Interviewer:

Where would you keep the backend VMs in a production architecture?

### Answer:

> I would normally keep the backend VMs on private IPs inside the VNet and expose only the Application Gateway frontend. This reduces direct internet exposure of the backend servers.

---

# 35. Real Production Architecture

A common Azure web architecture can look like this:

```text
                         Internet
                            |
                            v
                     Azure Front Door
                    Global HTTP/HTTPS
                            |
                            v
                Application Gateway + WAF
                            |
                  +---------+---------+
                  |                   |
                  v                   v
              Web Pool             API Pool
              Private               Private
              VM/VMSS               VM/VMSS
                  |                   |
                  +---------+---------+
                            |
                            v
                       Data Layer
                     Private Access
```

For a single-region application, Front Door may not be required.

A simpler design could be:

```text
Internet
   |
   v
Application Gateway + WAF
   |
   +---- Web Tier
   |
   +---- API Tier
   |
   v
Database
```

---

# 36. DevOps Engineer's Responsibility

As a DevOps engineer, I would typically work with Application Gateway for:

```text
1. Listener configuration
2. HTTPS certificates
3. Routing rules
4. Backend pools
5. Health probes
6. Backend HTTP settings
7. WAF policies
8. Monitoring
9. Troubleshooting 4xx/5xx errors
10. Infrastructure automation using Terraform
```

For Terraform-based environments, Application Gateway configuration should normally be managed as code rather than manually recreated in the portal.

---

# 37. Interview Mental Model

Remember the Application Gateway flow:

```text
CLIENT
  ↓
FRONTEND IP
  ↓
LISTENER
  ↓
ROUTING RULE
  ↓
BACKEND POOL
  ↓
HTTP SETTINGS
  ↓
HEALTH PROBE
  ↓
APPLICATION
```

And for WAF:

```text
CLIENT
  ↓
WAF
  ↓
APPLICATION GATEWAY
  ↓
BACKEND
```

---

# 38. Final Memory Trick

```text
Load Balancer
= L4
= TCP/UDP

Application Gateway
= L7
= HTTP/HTTPS
= Host/Path Routing
= TLS Termination
= WAF Capability
```

### One-line interview summary

> Application Gateway is Azure's Layer 7 web traffic service that provides HTTP/HTTPS routing, backend health checks, TLS termination, and WAF capability for web applications.
