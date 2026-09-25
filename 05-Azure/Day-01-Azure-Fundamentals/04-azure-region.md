# Azure Region

## What is an Azure Region?

An Azure Region is a **geographic location where Azure has datacenters and infrastructure** to run cloud resources.

Examples:

```text
East US
West Europe
Central India
Southeast Asia
```

### Easy Memory

> **Region = Geographic location of Azure infrastructure**

---

## Why Does Region Matter?

When deploying an Azure resource, region selection is important because of:

* Application latency
* Data residency requirements
* Compliance requirements
* Service availability
* Availability Zone support
* Cost
* Disaster Recovery

For example, if most users are located in India, deploying the application in an India region can help reduce network latency compared with deploying it in a distant region.

---

## Region vs Resource Group Location

A Resource Group has a location, but that does **not mean all resources inside the Resource Group must be deployed in the same region**.

Example:

```text
Resource Group: rg-payment-prod
Location: East US

Resources:
 ├── AKS       → East US
 ├── Storage   → West Europe
 └── Database  → East US
```

The resource group location mainly relates to the Resource Group's own metadata and management operations.

---

## How to Check Available Azure Regions

### List Azure Regions

```bash
az account list-locations -o table
```

### Filter a Specific Region

```bash
az account list-locations \
  --query "[?name=='<location>'].{Name:name,DisplayName:displayName}" \
  -o table
```

Example:

```bash
az account list-locations \
  --query "[?name=='eastus'].{Name:name,DisplayName:displayName}" \
  -o table
```

---

## Important Production Point

Not every Azure service, feature, or SKU is available in every region.

For example:

```text
Application
     ↓
Required Azure Service
     ↓
Check Region Availability
     ↓
Check SKU Availability
     ↓
Deploy
```

So before selecting a region for production, I check whether the required Azure services and SKUs are available there.

---

# Region Pairing and Resilience

Azure regions can be used as part of a **disaster recovery strategy**.

For a production application, we can have:

```text
Primary Region
     │
     │ Application running
     ▼
  East US
     │
     │ Disaster Recovery
     ▼
Secondary Region
     │
     ▼
  West US
```

The secondary region can be used for disaster recovery depending on the application's architecture and the Azure services being used.

### Example

Suppose the production application is running in:

```text
Primary:
East US
```

We can design a DR environment in another region:

```text
DR:
West US
```

If the primary region experiences a major outage, the application can be recovered or failed over to the DR region according to the application's DR design.

### Important

**Region pairing** is an Azure platform concept for certain regions and services. It does not mean that every resource automatically gets replicated to the paired region.

Replication and failover must be configured based on the specific Azure service.

### Memory Trick

> **AZ protects within a region.**
> **DR across regions protects against region-level failure.**

---

# Region vs Availability Zone vs Datacenter

These three terms are related but different.

```text
Azure Region
│
├── Availability Zone 1
│      └── Datacenter infrastructure
│
├── Availability Zone 2
│      └── Datacenter infrastructure
│
└── Availability Zone 3
       └── Datacenter infrastructure
```

### Region

A **Region** is a geographic Azure location containing Azure infrastructure.

Example:

```text
East US
West Europe
Central India
```

Memory:

> **Region = Geographic location**

---

### Availability Zone

An **Availability Zone (AZ)** is a physically separate location within a supported Azure region, designed to provide isolation from failures affecting another zone.

Example:

```text
East US Region
│
├── Zone 1
├── Zone 2
└── Zone 3
```

Not every Azure region supports Availability Zones, and not every Azure service supports them.

Memory:

> **Zone = Separate failure-isolated location inside a region**

---

### Datacenter

A **datacenter** is the physical infrastructure facility where computing, networking, storage and other Azure infrastructure operates.

A region can contain multiple datacenters, and Availability Zones are designed as separate physical locations within a region.

Memory:

> **Datacenter = Physical infrastructure facility**

---

## Simple Comparison

| Concept           | Meaning                                                      | Example          |
| ----------------- | ------------------------------------------------------------ | ---------------- |
| Region            | Geographic Azure location                                    | East US          |
| Availability Zone | Separate failure-isolated location inside a supported region | Zone 1           |
| Datacenter        | Physical facility/infrastructure location                    | Azure datacenter |

### Easy Way to Remember

```text
Region
  ↓
Contains Azure infrastructure
  ↓
Availability Zones
  ↓
Physical datacenter infrastructure
```

The exact physical mapping can vary, and Azure does not expose every physical datacenter detail to customers.

---

# Real Production Scenario

Suppose we have a production application running on AKS.

Instead of putting all workloads in one Availability Zone:

```text
East US
│
└── Zone 1
      └── All AKS nodes
```

we can design the cluster across multiple supported Availability Zones:

```text
East US
│
├── Zone 1 → AKS nodes
├── Zone 2 → AKS nodes
└── Zone 3 → AKS nodes
```

If one Availability Zone has an infrastructure failure, workloads may continue running in the other zones, depending on the AKS configuration and application design.

For protection against a **region-level disaster**, we would need a separate DR strategy involving another region.

---

# Interview Answer

> I select an Azure region based on latency, compliance, service availability, cost and disaster recovery requirements. For production, I also check whether the required services support Availability Zones. AZs provide resilience inside a region, while a secondary region can be used for disaster recovery against a region-level failure.

---

# Key Takeaways

```text
Region        → Geographic Azure location
AZ            → Failure-isolated location inside a region
Datacenter    → Physical infrastructure facility
Region Pairing → Supports regional resilience/DR strategy
AZ            → Intra-region resilience
Secondary Region → Region-level DR
```
