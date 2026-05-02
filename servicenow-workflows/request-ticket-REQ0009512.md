# Service Request · REQ0009512

> **Type:** Access Request · **State:** In Progress · **SLA:** On Track (Day 2 of 3)

---

## Header Information

| Field                  | Value                                              |
|------------------------|----------------------------------------------------|
| **Request Number**     | REQ0009512                                         |
| **Short Description**  | New Production Environment Access — QA Team Lead   |
| **Request Type**       | Access Management — New Access Grant               |
| **State**              | In Progress                                        |
| **Priority**           | P3 — Medium                                        |
| **Category**           | Access & Identity Management                       |
| **Subcategory**        | Production Environment Access                      |

---

## Requestor Information

| Field                | Value                                              |
|----------------------|----------------------------------------------------|
| **Requested By**     | Priya Nair                                         |
| **Job Title**        | QA Team Lead                                       |
| **Department**       | Quality Assurance                                  |
| **Email**            | p.nair@company.internal                            |
| **Employee ID**      | EMP-20847                                          |
| **Manager**          | Vikram Sharma (Engineering Director)               |

---

## Access Details

| Field                  | Value                                            |
|------------------------|--------------------------------------------------|
| **Environment**        | Production                                       |
| **Access Level**       | Read-Only                                        |
| **Access Type**        | Role-Based Access Control (RBAC)                 |
| **Requested Role**     | `prod-readonly-qa`                               |
| **Systems in Scope**   | Application logs, deployment dashboards, Datadog monitoring, Grafana, Kibana |
| **Systems Excluded**   | Database write access, Kubernetes admin, Secrets manager, CI/CD pipeline config |
| **Duration**           | Permanent (reviewed annually)                    |
| **Justification**      | Required for post-release validation activities and production smoke testing |

---

## Business Justification

Post-release validation currently requires QA Team Lead to request log access ad-hoc from DevOps engineers, creating bottlenecks during release windows. Standing read-only access to production logs and monitoring dashboards will:

1. Allow QA to independently validate deployments without waiting for DevOps availability
2. Reduce post-release sign-off time by an estimated 45 minutes per release
3. Support compliance requirement: QA sign-off requires direct evidence from production, not screenshots

**Risk:** Low. Read-only access prevents any configuration changes or data modification. Access is scoped to observability tooling only.

---

## Approval Workflow

| Step | Approver           | Role                    | Decision    | Date / Time          |
|------|--------------------|-------------------------|-------------|----------------------|
| 1    | Vikram Sharma      | Line Manager            | ✅ Approved | 2025-01-15 · 10:04   |
| 2    | Security Review    | InfoSec Team            | 🔄 In Progress | 2025-01-16 · ETA   |
| 3    | Access Provisioning| IT Operations           | ⏳ Pending  | —                    |
| 4    | User Notification  | IT Helpdesk             | ⏳ Pending  | —                    |

---

## SLA Tracking

| Metric              | Target         | Current Status           |
|---------------------|----------------|--------------------------|
| **SLA Target**      | 3 Business Days | Day 2 of 3 — On Track   |
| **Submitted**       | 2025-01-15 · 09:20 UTC | —               |
| **SLA Deadline**    | 2025-01-17 · 17:00 UTC | —               |
| **Time Remaining**  | ~20 hours (as of Day 2) | —              |

---

## Security Review Checklist

| Check                                              | Status     | Notes                               |
|----------------------------------------------------|------------|-------------------------------------|
| Access requested is least-privilege compliant       | ✅ Confirmed | Read-only RBAC role only           |
| No segregation-of-duties conflict                   | ✅ Confirmed | QA function does not conflict       |
| Employee currently active and in good standing      | ✅ Confirmed | HR verified                         |
| Production access policy acknowledges signed        | ⏳ Pending  | Email sent to Priya Nair            |
| MFA enforced on user account                        | ✅ Confirmed | TOTP configured since onboarding    |
| Access to sensitive data (PII, PCI) in scope?       | ✅ Reviewed  | No PII in log access scope         |

---

## Provisioning Steps (Pending Security Sign-Off)

Once security review is complete, IT Operations will:

1. Create RBAC role binding `prod-readonly-qa` for user `p.nair` in Kubernetes RBAC
2. Grant Datadog dashboard read access (Team: QA)
3. Grant Grafana read-only viewer role (Org: Production)
4. Grant Kibana read access to `checkout-*` and `payment-*` log indices
5. Add user to `prod-access-register` in CMDB
6. Set access expiry reminder for annual review (2026-01-15)

---

## Communication Log

| Date / Time         | Message                                                                 | From         |
|---------------------|-------------------------------------------------------------------------|--------------|
| 2025-01-15 · 09:20  | Request submitted via ServiceNow self-service portal                    | Priya Nair   |
| 2025-01-15 · 09:22  | Auto-acknowledgement sent (SLA: 3 BD)                                   | System       |
| 2025-01-15 · 09:30  | Manager approval request emailed to Vikram Sharma                       | System       |
| 2025-01-15 · 10:04  | Manager approved. Request routed to InfoSec for security review         | System       |
| 2025-01-15 · 10:10  | Security review assigned to InfoSec Team                                | System       |
| 2025-01-15 · 14:30  | Production Access Policy acknowledgement email sent to Priya Nair       | IT Helpdesk  |
| 2025-01-16 · 09:00  | SLA reminder: 1 business day remaining — security review in progress    | System       |

---

## Workflow History

```
[2025-01-15 09:20]  Submitted         → Priya Nair (Self-Service Portal)
[2025-01-15 09:22]  Awaiting Approval → Manager: Vikram Sharma
[2025-01-15 10:04]  Manager Approved  → Vikram Sharma
[2025-01-15 10:10]  Security Review   → InfoSec Team (In Progress)
[ Pending ]         Provisioning      → IT Operations
[ Pending ]         User Notification → IT Helpdesk
[ Pending ]         Closed
```

---

## Related Records

| Record Type | Number         | Description                                      |
|-------------|----------------|--------------------------------------------------|
| CI          | checkout-svc-prod | Application Priya will have log access to    |
| CI          | payment-api-prod  | Application Priya will have log access to    |
| Policy      | POL-PROD-ACCESS-002 | Production Access Policy (requires sign-off) |
| User Record | EMP-20847      | Priya Nair — HR System record                    |

---

## Closure Criteria

This request will be marked **Closed** when:
- [ ] Security review completed and approved
- [ ] All RBAC roles provisioned and verified
- [ ] Priya Nair confirms access works and acceptance email sent
- [ ] CMDB `prod-access-register` updated
- [ ] Annual review reminder set in access management calendar

---

*Record created: 2025-01-15 · Last updated: 2025-01-16 · Assigned Group: Access Management*
