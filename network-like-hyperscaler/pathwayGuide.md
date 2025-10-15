# HCFO Pathway Guide

*(Working Name: “Network like a Hyperscaler with Hedgehog / Hedgehog Certified Fabric Operator (HCFO)”)*

---

## Overview & Framing

This guide defines the **core instructional design** of the HCFO pathway. Your technical content team will flesh out each module with actual CRD names, API schemas, commands, error cases, screenshots, and demo scripts.

**Pathway Title:** Network like a Hyperscaler with Hedgehog
**Certification:** Hedgehog Certified Fabric Operator (HCFO)
**Core Duration:** ~4 hours
**Delivery Modes:** Bootcamp (instructor-led) or Modular (self-paced)
**Lab Environment:** Preprovisioned with full Hedgehog fabric, Prometheus + Grafana dashboards, Argo CD / GitOps with YAML editing
**Capstone / Assessment:** Hands-on lab scenario combining provisioning, diagnostics, rollback, escalation, and postmortem

**Audience Challenge:**
Some learners bring strong cloud-native backgrounds (Kubernetes, GitOps, APIs) but minimal networking depth; others are network experts needing to shift into abstraction-based, declarative, software-driven fabrics. The narrative should walk the middle path, with optional background modules for gaps.

---

## Pathway Structure

```
HCFO Pathway
⮡ Course 1: Foundations & Interfaces (4 modules)
⮡ Course 2: Provisioning & Connectivity (4 modules)
⮡ Course 3: Observability & Fabric Health (4 modules)
⮡ Course 4: Troubleshooting, Change Recovery & Escalation (4 modules)
⮡ Capstone Lab / Assessment  
Optional / Background Modules (tagged; not required in core sequence)
```

Each **course** bundles ~4 modules. Each **module** is ~15 minutes core + optional lab portion. Modules may link to **background** modules for reinforcing fundamentals.

---

## Course & Module Scaffolding

Below is the detailed scaffolding for every course and module. The implementation team should convert these into full content (slides, prose, demo scripts, YAML examples, assessments, UI screens, etc.).

### Course 1: Foundations & Interfaces

**Goal:** Build mental models, understand architecture, and become familiar with the interfaces (CLI, API, UI) you’ll use.

| Module | Title / Theme                | Objectives                                                        | Narrative / Flow Sketch                                                                           | Lab Tie‑in                                                          | Assessment / Quiz Ideas                                                     | Background / Optional Tags                  |
| ------ | ---------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------- |
| 1.1    | Welcome & Mindset            | Understand role vs system, adopt confidence mindset               | Walk through a day’s tasks (health checks, onboarding, small fault, rollback)                     | Show lab topology / dashboard, no edits                             | T/F and short answer: “Which tasks are manual vs automated?”                | *Network Fundamentals*, *Declarative Infra* |
| 1.2    | Architecture & Control Model | Grasp CRD reconciliation, abstraction boundaries, drift detection | Diagram end-to-end: CRD → controller → fabric → forwarding; show change → reconciliation → status | Inspect CRD objects, status fields, mapping to topology             | Matching: CRD → purpose; scenario: change spec mid-flight                   | *CRD & Kubernetes Reconciliation*           |
| 1.3    | Interfaces: CLI / API / UI   | Learn workflows across CLI, YAML / CRD, UI, and GitOps edits      | Show `apply`, `status`, `inspect`, UI event view, edit via GitOps web UI                          | Edit a simple CRD via GitOps web editor, sync, inspect result in UI | Quiz: “Which CLI inspects HostAttachment?”, scenario of invalid spec change | *YAML 101*, *GitOps Basics*                 |
| 1.4    | Recap & Forward Map          | Reinforce mental images, map flow, prepare for next courses       | Review abstractions, tools, sample end-to-end thought path                                        | Learners sketch steps to onboard a host                             | Flashcards: key terms; quiz on conceptual flow                              | Glossary links                              |

---

### Course 2: Provisioning & Connectivity

**Goal:** Enable safe and confident creation, modification, connection, and removal of network resources.

| Module | Title / Theme               | Objectives                                                         | Narrative / Flow Sketch                                        | Lab Tie‑in                                                           | Assessment / Quiz Ideas                                 | Background / Optional Tags       |
| ------ | --------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------- | -------------------------------------------------------------------- | ------------------------------------------------------- | -------------------------------- |
| 2.1    | Define Tenant Network       | Learners will create a network abstraction (VPC‑style)             | Show YAML for TenantNetwork, apply → reconcile → status → UI   | Create a TenantNetwork CRD, sync via GitOps, inspect result          | Quiz: “What validation errors might show?”              | *Subnet & Network Primitives*    |
| 2.2    | Onboarding Hosts / Move‑Add | Attach hosts, assign IPs, integrate them into fabric               | Introduce HostAttachment CRD, spec → apply → status            | Write HostAttachment CRDs, sync, confirm host in topology            | Scenario: “Host stays offline — what to check?”         | *IP Addressing Primer*           |
| 2.3    | Connectivity Validation     | Run diagnostics / path tracing, interpret results                  | Show commands/CRD diagnostics (`inspect`, `trace`)             | Use diagnostics in lab (good path and induced failure)               | Quiz: “Which command shows path hops?”                  | *Observability Basics*           |
| 2.4    | Decommission & Cleanup      | Safely remove host attachments and networks, confirm state cleanup | Delete CRDs, monitor reconciliation, ensure no residual config | Delete resources via GitOps UI, observe cleanup in topology / status | MCQ: “What if dependency still exists during deletion?” | *Lifecycle / Garbage Collection* |

---

### Course 3: Observability & Fabric Health

**Goal:** Help admins monitor, detect, and act on fabric health signals — metrics, events, logs, alerts.

| Module | Title / Theme                    | Objectives                                                   | Narrative / Flow Sketch                                                                | Lab Tie‑in                                                         | Assessment / Quiz Ideas                               | Background / Optional Tags     |
| ------ | -------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------- | ------------------------------------------------------------------ | ----------------------------------------------------- | ------------------------------ |
| 3.1    | Fabric Telemetry Overview        | Understand how Hedgehog exposes metrics, logs, and events    | Introduce Prometheus integration, event streams, metric types                          | Explore Prometheus targets, metric names, logs in UI               | Quiz: “Which metric shows control plane lag?”         | *Observability Fundamentals*   |
| 3.2    | Dashboard Interpretation         | Read Grafana panels, correlate metrics, detect anomalies     | Walk through default dashboards: error rate, reconciliation latency, host connectivity | Trigger a small change, watch dashboards update                    | Scenario: “Error spike — what panels do you inspect?” | *Time-series Metrics Concepts* |
| 3.3    | Alerts & Event Handling          | Understand alert rules, event-to-alarm flows, tuning         | Show built-in alerts, event-to-notification flow                                       | Trigger alert in lab environment, view in UI, trace back to event  | Quiz: “What event triggers reconciliation alert?”     | *Alerting Best Practices*      |
| 3.4    | Pre‑Support Diagnostic Checklist | Know what diagnostics artifacts to collect before escalation | Define standard checklist: CRD dumps, recent events, log snapshots, metrics            | For simulated fault, students collect artifacts via CLI / UI / API | Quiz: “Which artifacts are essential?”                | *Log / Trace Fundamentals*     |

---

### Course 4: Troubleshooting, Recovery & Escalation

**Goal:** Teach structured fault diagnosis, rollback, escalation, and post-incident learning.

| Module | Title / Theme                              | Objectives                                                                    | Narrative / Flow Sketch                                                                          | Lab Tie‑in                                                                               | Assessment / Quiz Ideas                                        | Background / Optional Tags            |
| ------ | ------------------------------------------ | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------- |
| 4.1    | Diagnosing Fabric Issues                   | Walk through symptom → scope → root-cause workflow                            | Present faults (connectivity broken, segmentation failure, gateway error), show diagnostic steps | Lab injects fault; students use commands, status, metrics to isolate issue               | Scenario: “Symptom X — what checks do you run first?”          | *Network Troubleshooting Methodology* |
| 4.2    | Rollback & Recovery                        | Learn safe reversion practices                                                | Show rollback via GitOps revert, via CLI undo, plus necessary cleanup                            | Student reverts a faulty change, sync, confirm connectivity restores                     | Quiz: “What rollback method is preferred?”                     | *GitOps Rollback Concepts*            |
| 4.3    | Coordinating with Support                  | Best practices for escalating, diagnostic packaging                           | Teach how to write succinct but rich support tickets, what artifacts to attach                   | Students write a support summary from a simulated fault, attach logs, CRD dumps          | Short answer: “What 3 items are key in a ticket?”              | *Incident Management Basics*          |
| 4.4    | Post‑Incident Review & Continuous Learning | Learn how to do a blameless review, update runbooks, and close feedback loops | Walk through a postmortem: timeline, root cause, corrective actions, guardrails                  | After capstone / fault, students produce a short postmortem document and runbook updates | Quiz / reflection: “What would you change in your operations?” | *Postmortem / Learning Culture*       |

---

## Optional / Background Modules

These modules are **not required** in the core pathway but are available for learners who need reinforcement. They should be tagged (e.g. `background: true`) in the LMS for filtering.

* CRD & Kubernetes Reconciliation Fundamentals
* Basic Networking Primitives (routing, forwarding, segmentation)
* Subnetting & IP Addressing Primer
* YAML Syntax & Best Practices
* GitOps & Argo CD Workflow Basics
* Observability, Logs, Metrics, Traces 101
* Alerting and Notification Fundamentals
* Network Troubleshooting Methodology
* Incident / Blameless Postmortem Practices

These can be surfaced as “Suggested background modules if you need them” in the module header or catalog.

---

## Capstone Lab / Assessment Scenario (Draft)

This is a **sketch** for your technical team to refine.

### Scenario Setup (Pre-Lab)

* Lab fabric is preconfigured with baseline networks, host attachments, gateways, metrics, and dashboards.
* Argo CD / GitOps pipeline is live with CRD yml files mapped to the fabric.
* Observability, logging, events, status APIs are functioning.

### Learner Tasks (Typical Workflow)

1. **Define a new TenantNetwork** with a given IP range and segmentation rules.
2. **Attach multiple hosts** to that network via HostAttachment CRDs (or equivalent), assign addresses, wait reconciliation.
3. **Validate connectivity** (host-to-host, and perhaps host to external endpoint), using diagnostics or path tracing.
4. **Inject a controlled fault** (e.g. miswired CRD, missing route, disabled gateway) — either via lab or learner action.
5. **Diagnose the fault** using logs, events, metrics, diagnostic commands.
6. **Recover / rollback** to working state via YAML revert or configuration correction, confirm connectivity.
7. **Prepare diagnostic package / support summary**, containing CRD dumps, events, logs, metrics snapshots.
8. **Draft brief postmortem / runbook update** describing root cause, lessons learned, recommended guardrails.

### Success Criteria & Grading

* All tasks completed within target time (e.g. 60 minutes)
* Connectivity is restored and validated
* Correct artifacts submitted (logs, CRDs, event logs, metrics)
* Postmortem shows understanding and recommendations
* Partial credit possible; weighting for correctness, clarity, smooth rollback

### Variants / Recertification

* Use different fault scenarios per iteration
* Evolve CRD schema version mid-lab to force adaptation
* Time limit changes or adding distractions

---

## Lab-to-Module Mapping

| Course   | Lab Focus                                                      |
| -------- | -------------------------------------------------------------- |
| Course 1 | Read-only exploration, CRD inspection, GitOps UI browsing      |
| Course 2 | Create / modify networks and hosts, confirm in UI              |
| Course 3 | Trigger changes, monitor metrics / alerts / dashboards         |
| Course 4 | Fault injection, diagnostics, rollback, escalation, postmortem |

Each module should include a **lab card** with:

* What file(s) to edit
* What CLI/API commands to run
* Which UI screens or dashboards to inspect
* What artifacts to collect
* What validation learners should confirm

---

## Bootcamp / Delivery Guide

* **Core content time:** ~240 minutes (4 hours)
* **Breaks:** 2 × 10 minutes
* **Lab time:** integrated (~5–10 min per module)
* **Capstone / assessment:** final 60 minutes
* **Pre-lab orientation:** 10 mins for environment / UI overview
* **Demo scripts / instructor notes:** one per module
* **Checkpoints / Q&A:** end of each module
* **Modular variant:** asynchronous with embedded labs and quizzes

---

## Implementation / Handoff Guidance

* Replace placeholder CRD, API, CLI names with actual implementation from docs.
* Populate error cases, status field names, drift metrics, diagnostic commands.
* Provide screenshots, YAML samples, UI mockups, and demo scripts.
* Ensure the lab supports fault injection, log / metric capture, rollback.
* Tag background modules as metadata (e.g. `background: true`).
* Embed glossary hyperlinks or tooltips for domain terms.
* Expand narrative sketches into full prose, slides, and visuals.
* Define concrete capstone details (ranges, hosts, faults, grading rubric).
* Establish recertification and versioning plan.

---

**End of Document**
