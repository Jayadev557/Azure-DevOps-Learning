# Azure Availability Zones

## What is an Availability Zone?

An Availability Zone (AZ) is a **physically separate location within an Azure region** designed to provide isolation from failures affecting another zone.

Example:

```text
Azure Region
│
├── Availability Zone 1
├── Availability Zone 2
└── Availability Zone 3
```

### Easy Memory

> **Region = Geographic location**
> **AZ = Separate failure-isolated location inside that region**

---

## Why Do We Use Availability Zones?

The main purpose is **high availability and resilience**.

Suppose an application is running only in Zone 1:

```text
East US
│
└── Zone 1
      └── Application
```

If Zone 1 has an infrastructure failure, the application may become unavailable.

Instead, we can distribute workloads:

```text
East US
│
├── Zone 1 → Application instance
├── Zone 2 → Application instance
└── Zone 3 → Application instance
```

Now the application has protection against a failure affecting a single zone, assuming the application and Azure services are designed to use the zones correctly.

---

# Availability Zone vs Region

This is an important interview concept.

```text
Region
│
├── Zone 1
├── Zone 2
└── Zone 3
```

### Region

A region is a **geographic Azure location**.

Example:

```text
East US
```

### Availability Zone

An AZ is a **separate physical/failure-isolated location inside a supported region**.

Example:

```text
East US
 ├── Zone 1
 ├── Zone 2
 └── Zone 3
```

### Memory Trick

> **AZ protects inside a region.**
> **Another region protects against region-level failure.**

---

# Zonal vs Zone-Redundant Resources

Azure services can use Availability Zones in different ways.

## Zonal Resource

A zonal resource is deployed into a **specific Availability Zone**.

Example:

```text
East US
│
└── Zone 1
      └── VM
```

The resource is associated with that particular zone.

---

## Zone-Redundant Resource

A zone-redundant service is designed to automatically distribute or replicate infrastructure across multiple Availability Zones, depending on the service.

Example:

```text
East US
│
├── Zone 1
├── Zone 2
└── Zone 3
       │
       └── Zone-redundant service
```

The exact behavior depends on the Azure service.

---

# Real Production Example — AKS

Suppose we have an AKS cluster running our production application.

Without zone distribution:

```text
AKS
│
└── Zone 1
    ├── Node
    ├── Node
    └── Node
```

If Zone 1 becomes unavailable, all those nodes can be affected.

With multiple zones:

```text
AKS Cluster
│
├── Zone 1
│    ├── Node
│    └── Node
│
├── Zone 2
│    ├── Node
│    └── Node
│
└── Zone 3
     ├── Node
     └── Node
```

Kubernetes can then schedule workloads across available nodes according to the cluster configuration and workload rules.

---

# Important Point

Availability Zones are **not available in every Azure region**, and not every Azure service or SKU supports Availability Zones.

So before using AZs in production, I check:

```text
Region
   ↓
Is AZ supported?
   ↓
Does required Azure service support AZ?
   ↓
Does required SKU support AZ?
   ↓
Design workload distribution
```

---

# Availability Zone Failure Scenario

Suppose an application has three instances:

```text
Zone 1 → App-1
Zone 2 → App-2
Zone 3 → App-3
```

If Zone 2 becomes unavailable:

```text
Zone 1 → App-1  ✅
Zone 2 → App-2  ❌
Zone 3 → App-3  ✅
```

The application can continue serving traffic from the remaining instances if the application architecture, load balancing and dependencies are properly designed.

---

# AZ vs Disaster Recovery

Availability Zones and Disaster Recovery solve different problems.

### Availability Zone

Protects against failures affecting a **zone within a region**.

```text
East US
├── Zone 1
├── Zone 2
└── Zone 3
```

### Disaster Recovery

Protects against a **larger regional failure** by using another region.

```text
Primary Region
East US
     │
     │ DR
     ▼
Secondary Region
West US
```

### Easy Memory

> **AZ = High Availability**
> **Another Region = Disaster Recovery**

---

# Interview Scenario

### Question:

**Your production AKS application is running in one Availability Zone. What would you change for better availability?**

### Answer:

> I would distribute the AKS nodes across multiple Availability Zones if the region and required SKUs support them. I would also make sure the application has multiple replicas and traffic can reach healthy pods across the nodes. For region-level disaster recovery, I would use a separate region.

---

# Another Interview Question

### Question:

**If one Availability Zone goes down, does Azure automatically move every resource to another zone?**

### Answer:

> No, not every resource automatically moves to another zone. It depends on the Azure service and how the resource was deployed. For high availability, I design the workload using zonal or zone-redundant capabilities supported by that service.

---

# Key Takeaways

```text
Availability Zone
        ↓
Separate failure-isolated location
        ↓
Inside an Azure Region
        ↓
Used for High Availability
        ↓
Multiple zones reduce single-zone failure impact
```

Remember:

```text
Region       → Geographic location
AZ           → Resilience inside a region
DR Region    → Protection against regional failure
```
