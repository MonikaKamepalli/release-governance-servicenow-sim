# Incident Record · INC0043201

> **Priority:** P1 — Critical · **State:** Resolved · **SLA:** Breached (T+1h 12min)

---

## Header Information

| Field                  | Value                                             |
|------------------------|---------------------------------------------------|
| **Incident Number**    | INC0043201                                        |
| **Short Description**  | Production Outage — Checkout Service 503 Errors   |
| **Priority**           | P1 — Critical                                     |
| **State**              | Resolved                                          |
| **Category**           | Application / Service                             |
| **Subcategory**        | Service Unavailability                            |
| **Impact**             | High — Revenue-affecting                          |
| **Urgency**            | Critical                                          |
| **Source**             | Automated Monitoring Alert (Datadog)              |

---

## Detection & Assignment

| Field                | Value                                           |
|----------------------|-------------------------------------------------|
| **Detected At**      | 2025-01-14 · 14:32 UTC                          |
| **Reported By**      | Datadog Alert · ALT-CHECKOUT-503-PROD           |
| **Assigned Group**   | DevOps On-Call                                  |
| **Incident Commander** | James Lee (On-Call Engineer)                  |
| **Bridge ID**        | #inc-43201 (Zoom Bridge — 12 attendees)         |
| **Parent Change**    | CHG0012842 (hotfix deployed at 14:15 UTC)       |

---

## SLA Targets

| SLA Type         | Target     | Actual       | Status          |
|------------------|------------|--------------|-----------------|
| Time to Detect   | < 5 min    | 2 min        | ✅ Met          |
| Time to Acknowledge | < 15 min | 8 min       | ✅ Met          |
| Time to Resolve  | < 1 hour   | 1h 12 min    | ❌ Breached     |
| Time to Notify Stakeholders | < 30 min | 18 min | ✅ Met      |

**SLA Breach Reason:** Root cause identification delayed by misleading log noise from an unrelated cron job. Added 22 minutes to investigation.

---

## Incident Timeline

| Time (UTC)  | Event                                                                              | Actor              |
|-------------|------------------------------------------------------------------------------------|--------------------|
| 14:15       | CHG0012842 hotfix deployed to production (memory limit config change)              | Platform Eng       |
| 14:32       | Datadog alert fires: checkout-svc-prod error rate >10% (503 responses)            | Automated          |
| 14:34       | PagerDuty pages on-call engineer James Lee                                         | Automated          |
| 14:40       | James Lee acknowledges, opens war room bridge, notifies Release Manager            | James Lee          |
| 14:44       | Incident Commander declares P1, stakeholder notification sent                      | James Lee          |
| 14:50       | Initial investigation: CPU & memory graphs reviewed — memory spike identified      | DevOps Team        |
| 15:02       | False lead: cron job log noise investigated and ruled out (+22 min delay)          | DevOps Team        |
| 15:18       | Root cause confirmed: memory leak in hotfix config — JVM heap exhausted            | James Lee          |
| 15:22       | Rollback of CHG0012842 authorized by Release Manager Rachel Morgan                 | Rachel Morgan      |
| 15:26       | Rollback executed — traffic shifted back to pre-hotfix deployment                  | Luis Perez         |
| 15:30       | Error rate returns to baseline (<0.1%) — service healthy                           | Automated          |
| 15:44       | All-clear confirmed, bridge closed, stakeholders notified of resolution            | Rachel Morgan      |
| 15:50       | Incident state → Resolved · Resolution notes captured                              | James Lee          |
| 16:00       | PIR scheduled for 2025-01-16 10:00 UTC                                             | Rachel Morgan      |

---

## Root Cause Analysis

**Root Cause:** The hotfix deployed in CHG0012842 introduced an incorrect JVM heap configuration (`-Xmx512m` instead of `-Xmx2048m`). Under normal production load, the checkout service exhausted available heap memory within 17 minutes of deployment, triggering `OutOfMemoryError` exceptions and causing the pod to return 503 responses.

**Contributing Factor:** The misconfiguration was not caught in staging because the staging environment runs with reduced load (< 5% of production traffic), which did not trigger the heap exhaustion during smoke testing.

**Why Monitoring Didn't Catch It Earlier:** Memory metrics were being collected but the JVM heap alert threshold was set at 95% (for a 512MB heap), which was reached only after 17 minutes under load.

---

## Impact Assessment

| Metric                      | Value                                           |
|-----------------------------|-------------------------------------------------|
| **Duration**                | 58 minutes (14:32 – 15:30 UTC)                  |
| **Affected Users**          | ~12,400 unique sessions (estimated)             |
| **Failed Transactions**     | ~3,200 checkout attempts                        |
| **Estimated Revenue Impact**| £24,800 (avg order value × failed transactions) |
| **Error Rate Peak**         | 34.7% (503 responses)                           |
| **Services Affected**       | checkout-svc-prod, order-notification-svc       |

---

## Affected Configuration Items (CMDB)

| CI Name                    | Type        | Impact Level |
|----------------------------|-------------|--------------|
| checkout-svc-prod          | Application | Direct       |
| lb-prod-02                 | Load Balancer | Routing 503s |
| order-notification-svc     | Application | Downstream   |
| k8s-cluster-prod           | Infrastructure | Pod restarts |

---

## Resolution

**Resolution Action:** Rolled back CHG0012842 to restore pre-hotfix pod configuration. JVM heap restored to `-Xmx2048m`. All pods restarted and health checks confirmed green.

**Resolution Category:** Rollback  
**Resolution Code:** Software — Configuration Error  
**Resolved By:** Luis Perez (DevOps Lead) + James Lee (On-Call)

---

## Post-Incident Review (PIR)

**PIR Date:** 2025-01-16 · 10:00 UTC  
**Attendees:** James Lee, Luis Perez, Rachel Morgan, Sarah Mitchell, Tom Keller, Sunita Agarwal

### Findings

| Finding | Severity | Owner | Due Date |
|---------|----------|-------|----------|
| JVM heap config not validated in CD pipeline | High | Platform Eng | 2025-01-24 |
| Staging load insufficient to catch memory issues | High | DevOps | 2025-01-28 |
| JVM heap alert threshold too high (95%) | Medium | Monitoring Team | 2025-01-21 |
| Hotfix change process lacked mandatory checklist | Medium | Release Manager | 2025-01-24 |
| Log noise from cron job caused 22-min investigation delay | Low | DevOps | 2025-01-31 |

### Action Items

1. **Add JVM heap validation step** to CI/CD pipeline — fail build if heap config deviates from baseline  
2. **Increase staging load test** to minimum 20% of production traffic for all hotfix deployments  
3. **Lower JVM heap alert threshold** from 95% to 75% to provide earlier warning  
4. **Create mandatory hotfix deployment checklist** with required config diff review  
5. **Suppress cron job logs** from incident investigation dashboards (or label them clearly)

---

## Workflow History

```
[2025-01-14 14:32]  New             → Auto-created from Datadog alert
[2025-01-14 14:40]  In Progress     → Assigned to James Lee (DevOps On-Call)
[2025-01-14 14:44]  P1 Declared     → Bridge opened, stakeholders notified
[2025-01-14 15:26]  Resolving       → Rollback executed
[2025-01-14 15:44]  Resolved        → All-clear confirmed
[2025-01-16 10:45]  PIR Completed   → Action items logged
[2025-01-17 09:00]  Closed          → All immediate actions assigned
```

---

## Related Records

| Record Type | Number       | Description                                    |
|-------------|--------------|------------------------------------------------|
| Change      | CHG0012842   | Hotfix that caused the incident                |
| Change      | CHG0012847   | Planned v3.2 upgrade (separate, not affected)  |
| Problem     | PRB0004781   | Root problem record: Hotfix process gaps       |
| Release     | REL-003      | Next scheduled release — PIR findings included |

---

*Record created: 2025-01-14 · Resolved: 2025-01-14 · PIR Closed: 2025-01-17 · Owner: DevOps On-Call*
