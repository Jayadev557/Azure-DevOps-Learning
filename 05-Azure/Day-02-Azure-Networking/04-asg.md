# Azure Application Security Group - ASG

## 1. What is an Application Security Group?

Application Security Group (ASG) is used to logically group Azure network interfaces based on their application role.

The important point is:

> **ASG does not itself allow or deny traffic. NSG does that. ASG helps us define NSG rules using application groups instead of individual IP addresses.**

Example:

```text
                    NSG
                     |
          +----------+----------+
          |                     |
          v                     v
     Web-ASG                API-ASG
      |  |                   |  |
      v  v                   v  v
    Web1 Web2              API1 API2
```

Then we can create an NSG rule like:

```text
Web-ASG
   |
   | TCP 8080
   v
API-ASG
```

This means:

> Allow traffic from the web servers to the API servers on port 8080.

---

# 2. Easy Memory Trick

Remember:

```text
ASG → Group resources by application role
NSG → Allow or deny traffic
```

So:

```text
ASG = Who belongs to the application group?

NSG = What traffic is allowed?
```

---

# 3. Why Do We Need ASG?

Imagine we have:

```text
Web Servers:
10.10.1.10
10.10.1.11
10.10.1.12

API Servers:
10.10.2.10
10.10.2.11
10.10.2.12
```

Without ASG, we may end up maintaining rules based on individual IP addresses.

That becomes difficult when servers are added or removed.

With ASG:

```text
Web-ASG
 |
 +-- Web1
 +-- Web2
 +-- Web3

API-ASG
 |
 +-- API1
 +-- API2
 +-- API3
```

The NSG rule can reference:

```text
Source: Web-ASG
Destination: API-ASG
Port: 8080
```

Now the rule represents the application architecture instead of hard-coded server IPs.

---

# 4. ASG Works With NSG

This is the most important concept.

ASG alone does not provide traffic filtering.

The relationship is:

```text
              NSG
               |
        +------+------+
        |             |
        v             v
     Web-ASG       API-ASG
        |             |
      Web VMs       API VMs
```

The NSG uses ASGs as source and/or destination in its security rules.

---

# 5. Real Production Example

Suppose we have a three-tier application:

```text
                  Internet
                     |
                     v
              Application Gateway
                     |
                     v
                  Web Tier
                     |
                     v
                  API Tier
                     |
                     v
                Database Tier
```

We can create:

```text
Web-ASG
API-ASG
DB-ASG
```

Architecture:

```text
Web-ASG
   |
   | TCP 8080
   v
API-ASG
   |
   | TCP 1433
   v
DB-ASG
```

NSG rules:

```text
Web-ASG → API-ASG → 8080 → Allow

API-ASG → DB-ASG → 1433 → Allow
```

And we don't need to write every VM IP into the rule.

---

# 6. ASG and NIC

ASGs are associated with network interfaces.

Conceptually:

```text
VM
 |
NIC
 |
ASG
```

Example:

```text
Web VM 1
   |
 NIC
   |
Web-ASG


Web VM 2
   |
 NIC
   |
Web-ASG
```

The ASG represents the role of those NICs.

---

# 7. ASG Does Not Replace NSG

This is a common interview mistake.

Wrong:

> "ASG controls network traffic."

Better:

> "ASG groups network interfaces by application role, and NSG uses those groups when defining traffic rules."

Remember:

```text
ASG → Grouping
NSG → Filtering
```

---

# 8. ASG vs NSG

| ASG                              | NSG                      |
| -------------------------------- | ------------------------ |
| Logical grouping                 | Traffic filtering        |
| Groups NICs                      | Contains security rules  |
| Represents application role      | Allows or denies traffic |
| Used by NSG rules                | Enforces security rules  |
| Doesn't filter traffic by itself | Filters traffic          |

---

# 9. ASG Example

Suppose:

```text
Web Servers
----------------
web-01
web-02
web-03

API Servers
----------------
api-01
api-02
```

Create:

```text
Web-ASG
API-ASG
```

Then:

```text
Web-ASG
   |
   | TCP 8080
   v
API-ASG
```

This is much easier to understand than:

```text
10.10.1.10 → 10.10.2.10 → 8080
10.10.1.10 → 10.10.2.11 → 8080
10.10.1.11 → 10.10.2.10 → 8080
10.10.1.11 → 10.10.2.11 → 8080
```

---

# 10. Create ASG Using Azure CLI

Create a Web ASG:

```bash
az network asg create \
  --resource-group <resource-group> \
  --name web-asg
```

Create an API ASG:

```bash
az network asg create \
  --resource-group <resource-group> \
  --name api-asg
```

List ASGs:

```bash
az network asg list \
  --resource-group <resource-group> \
  -o table
```

---

# 11. Show ASG

```bash
az network asg show \
  --resource-group <resource-group> \
  --name web-asg \
  -o json
```

---

# 12. Associate NIC With ASG

Suppose we have a NIC:

```text
<nic-name>
```

Associate it with Web-ASG:

```bash
az network nic ip-config update \
  --resource-group <resource-group> \
  --nic-name <nic-name> \
  --name ipconfig1 \
  --application-security-groups web-asg
```

The exact IP configuration name should match the NIC configuration.

Check the NIC:

```bash
az network nic show \
  --resource-group <resource-group> \
  --name <nic-name> \
  -o json
```

---

# 13. Create NSG Rule Using ASGs

Suppose we already have:

```text
NSG: backend-nsg

Source:
web-asg

Destination:
api-asg

Port:
8080
```

Example:

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name backend-nsg \
  --name allow-web-to-api \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs web-asg \
  --destination-asgs api-asg \
  --destination-port-ranges 8080
```

Check the rule:

```bash
az network nsg rule show \
  --resource-group <resource-group> \
  --nsg-name backend-nsg \
  --name allow-web-to-api \
  -o json
```

---

# 14. Production Traffic Flow

Suppose a user accesses the application:

```text
User
 |
 v
Application Gateway
 |
 v
Web Server
 |
 | TCP 8080
 v
API Server
 |
 | TCP 1433
 v
Database
```

With ASGs:

```text
Web Server
    |
    v
 Web-ASG
    |
    | 8080
    v
 API-ASG
    |
    | 1433
    v
 DB-ASG
```

NSG rules define which traffic is allowed between these groups.

---

# 15. Why ASG Helps in Dynamic Environments

Imagine the API tier initially has:

```text
api-01
api-02
```

Later the application scales:

```text
api-01
api-02
api-03
api-04
api-05
```

If the security rule is designed around the API-ASG, we don't need to redesign the rule for every new API NIC.

We add the new NICs to the appropriate ASG.

Conceptually:

```text
API-ASG
 |
 +-- API-01
 +-- API-02
 +-- API-03
 +-- API-04
 +-- API-05
```

The application role remains the same.

---

# 16. Important Limitation

ASG is primarily useful when the workloads are represented by Azure network interfaces that can be associated with an ASG.

It is not a replacement for:

* NSG
* Azure Firewall
* Application Gateway WAF
* Network routing
* Identity-based access control

Each service solves a different problem.

---

# 17. ASG and NSG Production Design

A practical architecture could look like:

```text
                         Internet
                            |
                            v
                  Application Gateway
                            |
                            v
                       Web-ASG
                      /        \
                   Web1        Web2
                      \        /
                       \      /
                        v    v
                       API-ASG
                      /       \
                   API1       API2
                       \       /
                        v     v
                         DB-ASG
                            |
                            v
                         Database
```

NSG rules:

```text
Internet
   |
   | HTTPS 443
   v
Web-ASG

Web-ASG
   |
   | TCP 8080
   v
API-ASG

API-ASG
   |
   | TCP 1433
   v
DB-ASG
```

---

# 18. Troubleshooting Scenario

### Problem

The Web application cannot call the API on port 8080.

You know:

```text
Web server → Web-ASG
API server → API-ASG
```

I would check:

```text
1. Is the Web NIC associated with Web-ASG?
2. Is the API NIC associated with API-ASG?
3. Is the NSG associated correctly?
4. Does the NSG rule reference the correct ASGs?
5. Is port 8080 correct?
6. Is the application listening on 8080?
7. Is another NSG rule blocking the traffic?
8. Is routing correct?
9. Is the OS firewall blocking the port?
```

---

# 19. Common ASG Mistake

### Mistake

Creating an ASG but expecting traffic to automatically become allowed.

Example:

```text
Web-ASG
   |
API-ASG
```

This does **not** automatically allow communication.

We still need an NSG rule.

Correct:

```text
Web-ASG
   |
   | NSG rule
   | TCP 8080
   v
API-ASG
```

---

# 20. ASG vs IP-Based NSG Rules

### IP-based rule

```text
Source:
10.10.1.10

Destination:
10.10.2.10

Port:
8080
```

### ASG-based rule

```text
Source:
Web-ASG

Destination:
API-ASG

Port:
8080
```

The ASG approach expresses the **application relationship** more clearly.

---

# 21. Interview Answer

### Interviewer:

**What is an Application Security Group?**

### Answer:

> "ASG is used to logically group network interfaces based on their application role, such as web or API servers. ASG itself doesn't filter traffic; I use it with NSG rules to allow communication between application groups without hard-coding individual IP addresses."

---

# 22. Interview Scenario

### Interviewer:

**Why would you use ASG instead of IP addresses in an NSG rule?**

### Answer:

> "In a dynamic environment, server IPs can change when workloads are added or replaced. With ASGs, I can define the rule based on application roles like Web-ASG to API-ASG, so the security policy is easier to maintain as the environment scales."

---

# 23. Interview Scenario

### Interviewer:

**Does ASG replace NSG?**

### Answer:

> "No. ASG is only used for logical grouping of network interfaces. NSG is responsible for allowing or denying traffic. ASGs are referenced by NSG rules to make the rules easier to manage."

---

# 24. Hands-On Lab

We will use the networking resources created in the previous topics.

### Step 1 - Create Web ASG

```bash
az network asg create \
  --resource-group <resource-group> \
  --name web-asg
```

### Step 2 - Create API ASG

```bash
az network asg create \
  --resource-group <resource-group> \
  --name api-asg
```

### Step 3 - Verify

```bash
az network asg list \
  --resource-group <resource-group> \
  -o table
```

### Step 4 - Create an NSG rule using the ASGs

```bash
az network nsg rule create \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name allow-web-to-api \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-asgs web-asg \
  --destination-asgs api-asg \
  --destination-port-ranges 8080
```

### Step 5 - Verify the rule

```bash
az network nsg rule show \
  --resource-group <resource-group> \
  --nsg-name <nsg-name> \
  --name allow-web-to-api \
  -o table
```

If you don't currently have two suitable NICs/VMs for the full ASG association lab, complete the **ASG creation + NSG rule exercise** and don't create extra paid resources just for this topic.

---

# 25. Final Mental Model

Remember this flow:

```text
NIC
 |
 v
ASG
 |
 | Application role
 v
NSG
 |
 | Allow / Deny
 v
Traffic
```

Example:

```text
Web NIC
   |
   v
Web-ASG
   |
   | TCP 8080
   v
API-ASG
   |
   v
API NIC
```

### Final memory trick

> **ASG groups the application; NSG controls the traffic.**
