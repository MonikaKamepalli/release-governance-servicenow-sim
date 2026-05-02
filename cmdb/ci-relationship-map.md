# CMDB · Configuration Item Relationship Map

> This document maps the CI dependencies across the production environment. It is used during incident triage to understand blast radius, during change management to assess risk, and during capacity planning to understand service coupling.

---

## Why the CMDB Exists

A Configuration Management Database (CMDB) is a repository that holds information about all the hardware, software, and services (Configuration Items, or CIs) that make up the IT environment — and critically, **how they relate to each other**.

Without a CMDB:
- During a P1 incident, you don't know which services depend on the broken one
- During a change review, you can't assess downstream impact
- During audits, you can't prove what's in production and who owns it

With a CMDB, a single CI record tells you: what it is, who owns it, what it depends on, what depends on it, and its current health status.

---

## CI Tier Model

CIs are organized into three tiers based on customer proximity:

| Tier | Description | Examples |
|------|-------------|---------|
| **Tier 1 — Application** | Customer-facing or revenue-critical services | checkout-svc, payment-api |
| **Tier 2 — Middleware** | Supporting infrastructure consumed by Tier 1 | Databases, caches, load balancers |
| **Tier 3 — Infrastructure** | Foundational compute, network, and cloud resources | Kubernetes clusters, VPCs |

---

## Production CI Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    TIER 1 — APPLICATION                         │
│                                                                 │
│   ┌─────────────────────────┐   ┌─────────────────────────┐   │
│   │    checkout-svc-prod    │   │    payment-api-prod      │   │
│   │  Owner: Platform Eng    │   │  Owner: Payments Team    │   │
│   │  Health: ⚠ Incident     │   │  Health: ✅ Healthy      │   │
│   └────────────┬────────────┘   └───────────┬─────────────┘   │
└────────────────┼──────────────────────────── ┼────────────────┘
                 │ depends_on                   │ depends_on
                 ▼                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   TIER 2 — MIDDLEWARE                           │
│                                                                 │
│  ┌──────────────────┐  ┌──────────────┐  ┌──────────────────┐ │
│  │  db-postgres-prod│  │redis-cache   │  │   lb-prod-02     │ │
│  │  Owner: DBA Team │  │Owner: Infra  │  │ Owner: NetOps    │ │
│  │  Health: ✅       │  │Health: ✅    │  │ Health: ✅        │ │
│  └────────┬─────────┘  └──────┬───────┘  └────────┬─────────┘ │
└───────────┼────────────────── ┼────────────────────┼───────────┘
            │ hosted_on         │ hosted_on           │ hosted_on
            ▼                   ▼                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                  TIER 3 — INFRASTRUCTURE                        │
│                                                                 │
│         ┌────────────────────────┐  ┌──────────────────┐       │
│         │   k8s-cluster-prod     │  │   aws-vpc-prod   │       │
│         │   Owner: DevOps        │  │   Owner: CloudOps│       │
│         │   Health: 🔄 Patching  │  │   Health: ✅      │       │
│         └────────────────────────┘  └──────────────────┘       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Relationship Types

| Relationship       | Meaning                                                             |
|--------------------|---------------------------------------------------------------------|
| `depends_on`       | CI A requires CI B to function correctly                           |
| `hosted_on`        | CI A runs within or on top of CI B                                 |
| `connects_to`      | CI A communicates with CI B (network path)                         |
| `backed_up_by`     | CI A has a backup/failover managed by CI B                         |
| `monitored_by`     | CI A is observed by monitoring tool CI B                           |

---

## Individual CI Records

---

### CI: checkout-svc-prod

| Field               | Value                                                   |
|---------------------|---------------------------------------------------------|
| **CI Name**         | checkout-svc-prod                                       |
| **CI Type**         | Application Service                                     |
| **Tier**            | 1 (Customer-Facing)                                     |
| **Owner**           | Platform Engineering                                    |
| **Technical Contact** | James Lee (james.lee@company.internal)               |
| **Environment**     | Production                                              |
| **Version**         | v2.8.1 (rolled back from v2.8.2 post-incident)         |
| **Hosting**         | k8s-cluster-prod · Namespace: checkout                 |
| **Replicas**        | 6 pods (auto-scaling 4–12)                              |
| **Health Status**   | ⚠ Incident Active — INC0043201                         |
| **SLA Class**       | Gold (99.9% uptime target)                              |
| **Depends On**      | db-postgres-prod, redis-cache-prod, lb-prod-02          |
| **Used By**         | Web frontend, mobile apps, order-notification-svc       |
| **Monitored By**    | Datadog, Grafana, PagerDuty                             |
| **Last Changed By** | CHG0012842 (hotfix — rolled back)                       |

---

### CI: payment-api-prod

| Field               | Value                                                   |
|---------------------|---------------------------------------------------------|
| **CI Name**         | payment-api-prod                                        |
| **CI Type**         | Application Service / API                               |
| **Tier**            | 1 (Revenue Critical)                                    |
| **Owner**           | Payments Team                                           |
| **Technical Contact** | Sarah Mitchell (s.mitchell@company.internal)         |
| **Environment**     | Production                                              |
| **Version**         | v3.1.4 (v3.2.1 upgrade scheduled: CHG0012847)          |
| **Hosting**         | k8s-cluster-prod · Namespace: payments                 |
| **Replicas**        | 4 pods (fixed)                                          |
| **Health Status**   | ✅ Healthy                                               |
| **SLA Class**       | Platinum (99.99% uptime target — PCI-DSS regulated)    |
| **Depends On**      | db-postgres-prod, lb-prod-02                            |
| **Used By**         | checkout-svc-prod                                       |
| **Monitored By**    | Datadog, Grafana, PCI monitoring suite                  |
| **Last Changed By** | CHG0012831 (routine config update — 2025-01-08)        |
| **Compliance Tags** | PCI-DSS, SOC 2                                          |

---

### CI: db-postgres-prod

| Field               | Value                                                   |
|---------------------|---------------------------------------------------------|
| **CI Name**         | db-postgres-prod                                        |
| **CI Type**         | Database                                                |
| **Tier**            | 2 (Middleware)                                          |
| **Owner**           | DBA Team                                                |
| **Technical Contact** | Maria Okonkwo (m.okonkwo@company.internal)           |
| **Environment**     | Production                                              |
| **Version**         | PostgreSQL 15.3                                         |
| **Hosting**         | AWS RDS · eu-west-1 (Multi-AZ)                         |
| **Health Status**   | ✅ Healthy                                               |
| **SLA Class**       | Platinum (99.99%)                                       |
| **Depends On**      | aws-vpc-prod                                            |
| **Used By**         | checkout-svc-prod, payment-api-prod                     |
| **Backup**          | Daily automated snapshots · 30-day retention           |
| **RPO / RTO**       | RPO: 1 hour · RTO: 30 minutes                          |

---

### CI: redis-cache-prod

| Field               | Value                                                   |
|---------------------|---------------------------------------------------------|
| **CI Name**         | redis-cache-prod                                        |
| **CI Type**         | Cache / In-Memory Data Store                            |
| **Tier**            | 2 (Middleware)                                          |
| **Owner**           | Infrastructure Team                                     |
| **Technical Contact** | DevOps On-Call Rotation                              |
| **Environment**     | Production                                              |
| **Version**         | Redis 7.2.3 · AWS ElastiCache                          |
| **Health Status**   | ✅ Healthy                                               |
| **Used By**         | checkout-svc-prod (session cache, cart data)            |
| **Notes**           | Non-persistent cache — data loss on restart acceptable  |

---

### CI: lb-prod-02

| Field               | Value                                                   |
|---------------------|---------------------------------------------------------|
| **CI Name**         | lb-prod-02                                              |
| **CI Type**         | Load Balancer                                           |
| **Tier**            | 2 (Middleware)                                          |
| **Owner**           | Network Operations (NetOps)                             |
| **Technical Contact** | NetOps On-Call                                       |
| **Environment**     | Production                                              |
| **Type**            | AWS Application Load Balancer (ALB)                    |
| **Health Status**   | ✅ Healthy                                               |
| **Used By**         | checkout-svc-prod, payment-api-prod                     |
| **Notes**           | Replaces lb-prod-01 (decommissioned 2024-11-01)        |

---

### CI: k8s-cluster-prod

| Field               | Value                                                   |
|---------------------|---------------------------------------------------------|
| **CI Name**         | k8s-cluster-prod                                        |
| **CI Type**         | Container Orchestration Platform                        |
| **Tier**            | 3 (Infrastructure)                                      |
| **Owner**           | DevOps Team                                             |
| **Technical Contact** | Luis Perez (l.perez@company.internal)               |
| **Environment**     | Production                                              |
| **Version**         | Kubernetes 1.28.4 · AWS EKS                            |
| **Health Status**   | 🔄 Patching (scheduled maintenance — CHG0012850)       |
| **Hosts**           | checkout-svc-prod, payment-api-prod, redis-cache-prod   |
| **Node Count**      | 12 worker nodes (m5.xlarge)                            |

---

### CI: aws-vpc-prod

| Field               | Value                                                   |
|---------------------|---------------------------------------------------------|
| **CI Name**         | aws-vpc-prod                                            |
| **CI Type**         | Network / Cloud Infrastructure                          |
| **Tier**            | 3 (Infrastructure)                                      |
| **Owner**           | Cloud Operations                                        |
| **Environment**     | Production · AWS eu-west-1                              |
| **Health Status**   | ✅ Healthy                                               |
| **CIDR**            | 10.0.0.0/16                                             |
| **Subnets**         | Public (3), Private (3), DB-isolated (2)               |

---

## Incident Impact Analysis Example

**Scenario:** `lb-prod-02` goes down

Using the CI relationship map, the blast radius is immediately clear:

```
lb-prod-02 DOWN
  └── checkout-svc-prod IMPACTED (depends_on lb-prod-02)
       └── Web frontend IMPACTED (used_by checkout-svc-prod)
       └── Mobile apps IMPACTED (used_by checkout-svc-prod)
  └── payment-api-prod IMPACTED (depends_on lb-prod-02)
       └── checkout-svc-prod DOUBLY IMPACTED

Result: Complete checkout flow unavailable → P1 incident
```

**Without CMDB:** You'd discover impact by getting customer complaints one by one.  
**With CMDB:** You know the full blast radius in < 60 seconds.

---

*Last updated: 2025-01-17 · Maintained by: Configuration Management Team · Review cycle: Monthly*
