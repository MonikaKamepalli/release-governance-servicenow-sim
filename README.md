# 🚀 Release Governance & ServiceNow Simulation

> A self-built project simulating enterprise-grade release management, ITSM workflows, and operational reporting — built to understand how production software is safely planned, approved, deployed, and recovered.

---

## 📌 Why I Built This

I wanted to understand what actually happens *behind the scenes* when a company releases software to production. As someone learning ITSM and IT operations, I kept seeing terms like **CAB approval**, **Change records**, **SLA compliance**, and **CMDB** in job descriptions and certifications — but no hands-on context for what they look like in practice.

I didn't have access to a real ServiceNow instance or a live production environment, so I built my own simulation from scratch. The goal was simple: **learn by doing**, not just reading.

This project helped me understand:
- Why companies have formal release processes (and what goes wrong without them)
- How ServiceNow ticket types (Change, Incident, Request) are structured and flow through approval stages
- What a real deployment checklist looks like and why each gate exists
- How SLA compliance is tracked and reported
- What a CMDB actually is and why CI relationships matter during an incident

---

## 🎯 What I Wanted to Learn

| Area | What I Was Trying to Understand |
|---|---|
| **Release Management** | How do teams safely plan and execute production deployments? |
| **CAB Process** | Who approves changes, and what criteria do they evaluate? |
| **ServiceNow Workflows** | How do CHG, INC, and REQ tickets move through their lifecycle? |
| **CMDB** | What is a Configuration Item, and how do CI relationships affect incident impact? |
| **SLA Reporting** | How is SLA compliance measured and reported in an Excel dashboard? |
| **Rollback Planning** | What does a real rollback decision process look like under pressure? |
| **Stakeholder Comms** | How are affected parties notified before, during, and after a release? |

---

## 🗂️ Project Structure

```
release-governance-servicenow-sim/
│
├── index.html                  # Interactive portfolio page (full project walkthrough)
├── README.md                   # This file
│
├── release-calendar/
│   └── weekly-release-calendar.xlsx    # Mock deployment calendar with release windows
│
├── servicenow-workflows/
│   ├── change-ticket-CHG0012847.md     # Standard Change record simulation
│   ├── incident-ticket-INC0043201.md   # P1 Incident record simulation
│   └── request-ticket-REQ0009512.md    # Service Request simulation
│
├── dashboard/
│   └── release-dashboard.xlsx          # Release success rate, incident severity, SLA compliance
│
├── cmdb/
│   ├── ci-relationship-map.md          # CMDB tier diagram and CI dependencies
│   └── ci-ownership-register.xlsx      # CI owners, environments, health status
│
├── rollback-plan/
│   └── rollback-procedure-REL003.md    # Step-by-step rollback timeline
│
└── comms/
    └── stakeholder-communication-log.md  # Pre/during/post release comms templates
```

---

## 📋 What's Included

### 1. Weekly Release Calendar
A mock monthly calendar showing how release teams schedule deployment windows, CAB review sessions, and freeze periods. Includes color-coded event types (standard release, CAB review, emergency hotfix) and a pre-deployment checklist with gate statuses (passed / pending / blocked).

**Why this matters:** Release calendars prevent conflicts between teams, align stakeholders, and ensure deployments happen in approved low-risk windows.

---

### 2. ServiceNow Workflow Simulation

Three ticket types simulated with realistic field data, workflow stages, and approvals:

**CHG0012847 — Standard Change**
A planned upgrade to a payment gateway API. Includes risk assessment, CAB approval, scheduled deployment window, and a Post-Implementation Review (PIR) stage.

**INC0043201 — P1 Critical Incident**
A production outage caused by a memory leak introduced in a hotfix. Tracks detection, triage, assignment, investigation, and resolution with SLA breach tracking.

**REQ0009512 — Service Request**
A new production access request for a QA team lead. Includes manager approval, security review, and access provisioning stages.

**Why this matters:** Understanding how tickets are structured, prioritized, and routed is foundational to working in any ITSM environment. ServiceNow is the #1 platform used in enterprise IT operations.

---

### 3. Excel Release Dashboard

A reporting dashboard tracking three core ITSM KPIs:

- **Release Success Rate** — Per-release pass/fail tracked across 6 releases
- **Incidents by Severity** — P1–P4 incident counts with MTTR and MTTD metrics
- **SLA Compliance** — Percentage compliance for Change, Incident, and Request SLAs

**Why this matters:** Managers and stakeholders don't read raw ticket data — they read dashboards. Building this taught me how to translate operational data into business-readable reporting.

---

### 4. CMDB · CI Relationships & Ownership

A three-tier Configuration Item map showing how production services depend on each other:

```
Tier 1 (Application):    checkout-svc-prod   payment-api-prod
Tier 2 (Middleware):     db-postgres-prod    redis-cache-prod    lb-prod-02
Tier 3 (Infrastructure): k8s-cluster-prod    aws-vpc-prod
```

Each CI has an assigned owner, environment tag, and current health status.

**Why this matters:** During a P1 incident, the CMDB tells you *what depends on what*. If `lb-prod-02` goes down, you instantly know it affects both Tier 1 applications. Without CMDB, you're guessing. CI ownership also makes it clear who to call.

---

### 5. Rollback Plan

A timestamped rollback procedure for REL-003, covering:
- Trigger conditions (error rate threshold breach)
- Decision point at T+10 minutes
- Automated rollback execution via blue/green traffic switch
- Smoke test validation
- Post-incident review scheduling

**Why this matters:** Every deployment should have a documented rollback path *before* go-live, not during a live outage. This simulation shows how rollback decisions are made under time pressure and what the recovery steps look like.

---

### 6. Stakeholder Communication Log

Four-stage communication plan used across the release lifecycle:
- Pre-release notice (T–48h) to all stakeholders via email + Slack
- Deployment start alert (T–2h) to tech leads via PagerDuty
- Incident bridge notification to senior leadership during a P1
- All-clear and resolution notice with status page update

**Why this matters:** Poor communication during a release or incident causes more chaos than the technical problem itself. Having templated, pre-written comms removes one decision from an already stressful situation.

---

### 7. Release Signoff

A formal approval record signed by six roles: Release Manager, CAB Chairperson, QA Lead, Security Officer, Product Owner, and DevOps Lead. This mirrors the gatekeeping process used in regulated industries (finance, healthcare, government) before any code touches production.

**Why this matters:** Signoff creates accountability and an auditable record. If something goes wrong after deployment, you can trace who approved what and when.

---

## 🛠️ Tools Used

| Tool | How It Was Used |
|---|---|
| **HTML / CSS / JavaScript** | Built the interactive portfolio page (index.html) |
| **Microsoft Excel** | Created the release dashboard and CI ownership register |
| **Markdown** | Documented ticket simulations, CMDB maps, and rollback procedures |
| **ServiceNow (conceptual)** | Referenced official ITSM workflow structures to simulate realistic tickets |

---

## 📚 What I Learned

**Technical:**
- How Change, Incident, and Request records are structured differently in ServiceNow and why each field exists
- Why CMDB relationships are maintained and how they're used during major incidents
- How SLA targets are defined per priority level and how breach timers work
- What blue/green deployments are and why they enable fast rollbacks

**Process:**
- Why CAB exists — it's not bureaucracy, it's a structured risk assessment with multiple perspectives
- Why code freeze windows matter and how they protect release stability
- How a war room bridge works during a P1 and who needs to be on it
- What a Post-Implementation Review (PIR) covers and why it closes the feedback loop

**Professional skills:**
- Thinking about risk before deployment, not after
- Writing structured operational documentation that others can follow under pressure
- Translating technical metrics into business-readable dashboards

---

## 🎓 Certifications This Supports

This project was built to reinforce concepts from:
- **ITIL 4 Foundation** — Service management lifecycle, change management, incident management
- **CompTIA Project+** — Project documentation, stakeholder communication, risk management
- **ServiceNow CSA (Certified System Administrator)** — Ticket structure, workflows, CMDB concepts

---

## 📬 About

Built by a learner interested in IT operations, ITSM, and release engineering. This project is intentionally self-directed — no tutorial followed, no pre-built template used. The goal was to understand the *why* behind enterprise processes, not just memorize definitions.

If you're studying for ITIL, exploring ServiceNow, or trying to break into IT operations — feel free to fork this, adapt it, or use it as a reference.

---

*Last updated: 2025 · Simulated data only — no real production systems or personal data involved.*
