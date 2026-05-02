# Rollback Procedure · REL-003

> **Release:** REL-003 · Payment Gateway API v3.2 Upgrade (CHG0012847)  
> **Status:** Rollback Executed · Triggered: 2025-01-17 02:27 UTC  
> **Incident:** INC0043201 (linked)

---

## Purpose

This document defines the step-by-step rollback procedure for REL-003. It is written **before deployment** and approved by the Release Manager as a mandatory pre-condition for go-live. During a live rollback, this document is the single source of truth — no improvisation.

**Rule:** If any engineer questions whether to roll back, the answer is: follow this document. The rollback decision criteria are pre-agreed. There is no ambiguity during an incident.

---

## Rollback Decision Criteria

A rollback is **automatically authorized** (no CAB approval required) if ANY of the following conditions are met during or after deployment:

| Trigger                            | Threshold           | Measurement Window |
|------------------------------------|---------------------|--------------------|
| HTTP 5xx error rate                | > 0.5%              | 5 consecutive minutes |
| P95 API latency                    | > 800ms             | 5 consecutive minutes |
| Pod crash loop (OOMKilled)         | Any occurrence      | Immediate          |
| Payment transaction failure rate   | > 0.1%              | 3 consecutive minutes |
| Manual declaration by Incident Commander | N/A           | Any time during window |

**Monitoring source:** Datadog dashboard `prod-release-watchdog` (link in deployment runbook)  
**Alert channel:** PagerDuty policy `P1-RELEASE-WATCH` → Pages Release Manager + On-Call DevOps

---

## Roles & Responsibilities During Rollback

| Role                   | Person (REL-003) | Responsibility                                      |
|------------------------|------------------|-----------------------------------------------------|
| **Incident Commander** | James Lee        | Declares rollback, runs bridge, owns timeline       |
| **Rollback Engineer**  | Luis Perez       | Executes all technical rollback steps               |
| **Release Manager**    | Rachel Morgan    | Authorizes rollback, communicates to stakeholders   |
| **QA Observer**        | Sunita Agarwal   | Validates health checks after rollback              |
| **Communications Lead**| Rachel Morgan    | Sends stakeholder notifications                     |

**War Room Bridge:** #inc-43201 (Zoom — all roles must join within 5 minutes of trigger)

---

## Pre-Rollback State Reference

Before deployment began, the following state was recorded and preserved:

| Item                          | Value                                       |
|-------------------------------|---------------------------------------------|
| **Previous stable version**   | payment-api v3.1.4                          |
| **Blue environment image**    | `payment-api:3.1.4-prod` (preserved)       |
| **Pod replica count**         | 4 pods                                      |
| **JVM heap config**           | `-Xmx2048m -Xms512m`                       |
| **Error rate baseline**       | 0.02%                                       |
| **P95 latency baseline**      | 235ms                                       |
| **Traffic routing**           | 100% → `payment-api-blue` (pre-deployment) |
| **CMDB version record**       | Updated pre-deployment (revert to v3.1.4 on rollback) |

---

## Rollback Execution Steps

### Phase 1: Detection & Decision (T+0 to T+10 min)

**Step 1.1 — Alert fires**
- Datadog alert `PROD-CHECKOUT-ERROR-RATE` triggers
- PagerDuty pages: James Lee (on-call), Luis Perez (backup), Rachel Morgan (release manager)
- **Time target:** All three paged within 2 minutes of threshold breach

**Step 1.2 — Open war room bridge**
- Incident Commander (James Lee) opens Zoom bridge: #inc-43201
- Posts in Slack `#release-notices`: `⚠️ REL-003 ALERT: Error rate threshold breached. Bridge open. Investigating.`
- **Time target:** Bridge open within 5 minutes of alert

**Step 1.3 — Assess and decide**
- Review Datadog dashboard `prod-release-watchdog`
- Check: Is the trigger condition confirmed? Is it sustained (not a spike)?
- **Decision rule:** If threshold met for 5+ consecutive minutes → rollback authorized
- Release Manager Rachel Morgan formally authorizes rollback on bridge
- Log decision in incident record: `[TIMESTAMP] Rollback authorized by Rachel Morgan`

---

### Phase 2: Execute Rollback (T+10 to T+20 min)

**Step 2.1 — Traffic shift to blue (Pre-deployment environment)**

Luis Perez executes the following:

```bash
# Verify blue environment is healthy
kubectl get pods -n payments -l version=3.1.4
kubectl rollout status deployment/payment-api-blue -n payments

# Shift 100% traffic back to blue via ALB target group weight
aws elbv2 modify-listener \
  --listener-arn $LISTENER_ARN \
  --default-actions Type=forward,TargetGroupArn=$BLUE_TG_ARN,Weight=100

# Confirm routing
aws elbv2 describe-rules --listener-arn $LISTENER_ARN
```

**Expected output:** All traffic on blue target group. Green receiving 0%.  
**Time target:** Traffic shifted within 60 seconds of step start.

**Step 2.2 — Confirm error rate drop**
- Watch Datadog for 2 minutes post-traffic-shift
- **Success criteria:** Error rate drops below 0.1% within 2 minutes
- If error rate does NOT drop → escalate (see Escalation Path below)

**Step 2.3 — Scale down green environment**
```bash
# Scale green pods to 0 (do not delete — preserve for investigation)
kubectl scale deployment payment-api-green --replicas=0 -n payments
```

---

### Phase 3: Validation (T+20 to T+30 min)

**Step 3.1 — Automated smoke tests**

```bash
# Run smoke test suite against production endpoint
./scripts/smoke-test.sh --env=prod --suite=payment-api --timeout=300
```

**Tests run:**
- `POST /api/v1/payments/initiate` → Expect 200 OK
- `GET /api/v1/payments/status/{id}` → Expect 200 OK
- `POST /api/v1/payments/refund` → Expect 200 OK
- Latency assertions: P95 < 400ms

**Step 3.2 — Manual validation by QA (Sunita Agarwal)**
- Confirm checkout flow end-to-end works (test transaction)
- Confirm payment dashboard shows healthy transaction counts
- Confirm monitoring shows no open alerts

**Step 3.3 — Confirm CMDB update**
- Release Manager updates CMDB CI record for `payment-api-prod`:
  - Version: Revert to `v3.1.4`
  - Last Changed By: Rollback of CHG0012847
  - Health Status: Healthy

---

### Phase 4: Communication & Closeout (T+30 min)

**Step 4.1 — All-clear communication**

Release Manager sends:
- Slack `#release-notices`: `✅ REL-003 ROLLBACK COMPLETE. payment-api-prod restored to v3.1.4. All services healthy. Monitoring continues.`
- Email to stakeholder distribution list (see comms log)
- Update status page: `Resolved — Payment API incident resolved at [TIME]`

**Step 4.2 — Close war room bridge**
- Confirm all roles acknowledge all-clear on bridge
- Log bridge close time in incident record
- Bridge participants: post final update to Slack thread

**Step 4.3 — Open incident record**
- INC record must be created/linked within 30 minutes of rollback decision
- Root cause: TBD (PIR within 48 hours)

**Step 4.4 — Schedule PIR**
- Book PIR within 24 hours of incident close
- Attendees: All roles above + Product Owner + Security Officer
- Agenda template: `pir-template-REL003.md`

---

## Escalation Path

If rollback does not restore service within 30 minutes:

| Step | Action                                                                 |
|------|------------------------------------------------------------------------|
| 1    | Escalate to DevOps Lead (Luis Perez) if not already on bridge          |
| 2    | Page CTO (on-call exec rotation) — Severity: Sev1                     |
| 3    | Engage AWS Support (Business Support case) if infrastructure suspected |
| 4    | Consider DNS failover to DR environment if full rollback fails         |

---

## Rollback Execution Log (Actual — REL-003)

| Time (UTC) | Action                                              | Actor        |
|------------|-----------------------------------------------------|--------------|
| 02:27      | Alert fired — error rate 34.7%                      | Datadog      |
| 02:29      | Bridge opened — all roles joined                    | James Lee    |
| 02:34      | Root cause suspected: memory config in hotfix       | James Lee    |
| 02:37      | Rollback authorized                                 | Rachel Morgan|
| 02:38      | Traffic shifted to blue (payment-api v3.1.4)        | Luis Perez   |
| 02:39      | Error rate dropped to 0.08%                         | Datadog      |
| 02:41      | Green pods scaled to 0                              | Luis Perez   |
| 02:47      | Smoke tests: 100% pass                              | Automated    |
| 02:50      | QA manual validation: Pass                          | S. Agarwal   |
| 02:52      | CMDB updated (v3.1.4 restored)                      | Rachel Morgan|
| 02:55      | All-clear sent to stakeholders                      | Rachel Morgan|
| 02:58      | Bridge closed                                       | James Lee    |
| 03:15      | INC0043201 created and linked                       | James Lee    |
| 03:30      | PIR booked: 2025-01-19 10:00 UTC                    | Rachel Morgan|

**Total rollback time: 31 minutes** (trigger to all-clear)

---

## Lessons Learned (Pre-PIR Notes)

1. Blue environment was preserved as designed — rollback was fast because of this
2. Automated smoke tests ran in 4 minutes — saved manual testing time
3. Need to investigate why the error was not caught in staging before this happened
4. All-clear comms was sent 3 minutes before QA sign-off — revise comms timing SOP

---

*Document prepared: 2025-01-15 · Executed: 2025-01-17 · Author: Rachel Morgan (Release Manager)*  
*Approval: James Chen (CAB) · Luis Perez (DevOps Lead) · Sunita Agarwal (QA Lead)*
