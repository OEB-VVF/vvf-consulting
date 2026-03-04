# VVF Project Management - Module Specification

## Overview

`vvf_project_management` is a reusable Odoo 19 module that implements VVF Consulting's agile project methodology. It structures ERP implementation projects around MVPs, with a clear pipeline from requirement capture through production deployment.

**Location:** `addons/vvf_project_management/`
**Dependencies:** `project`, `knowledge` (Odoo Enterprise)
**Scope:** Multi-company — each company represents a separate engagement with its own team, organisation, and chart of accounts.

## Deployment Model

| Customer type     | Where it runs                           |
| ----------------- | --------------------------------------- |
| Odoo customer     | Installed on the customer's SH instance |
| Non-Odoo customer | Runs on VVF's own SH instance           |

Currently developed inside the Kebony repository, to be extracted later (same pattern as `vvf_multi_ledger`).

---

## Core Concepts

### Company = Engagement Context

Each Odoo company represents a distinct customer engagement. This provides natural isolation of:
- Teams and resource allocation
- Chart of accounts
- Project configuration
- Reporting

### Agile / MVP-Driven Delivery

Projects are structured in **MVPs** (milestones). Each MVP represents a deliverable increment. Requirements are triaged into MVPs based on criticality:
- **MVP-mandatory** — cannot operate without it
- **Current practice** — reflects existing system behaviour (must be carried over)
- **Post-MVP** — nice-to-have, deferred

### Subject as Task Attribute

A **Subject** is a classification attribute on tasks — not a separate hierarchy level. Subjects represent independent functional domains (e.g., "Inventory Management", "Sales Order Flow").

**Design principle:** A subject must be independent in meaning and intent. If a requirement seems to span multiple subjects, the subjects are not well defined and should be restructured.

This allows full slice-and-dice: group by subject, by MVP, by responsible party, by stage — in any combination using standard Odoo views.

---

## Requirement Pipeline (Stages)

Each task (requirement) flows through the following stages:

### 1. Requirement (Capture)
> General needs capture — raw input from stakeholders

- Free-text description of the need
- Source: workshops, interviews, gap analysis
- No commitment yet

### 2. Requirement Validated
> Customer confirms this is a genuine need

- VVF and customer agree this is a real requirement
- Not yet prioritised or scheduled

### 3. Sorted Requirement
> Triaged, prioritised, and resourced

**Triage questions (in order):**

| # | Question | If YES |
|---|----------|--------|
| 1 | Is this mandatory for the MVP? (Cannot operate without it) | → **MVP scope** |
| 2 | Does it reflect current practice from the existing system? | → **MVP scope** (carry-over) |
| 3 | Neither of the above | → **Post-MVP** (deferred) |

**The triage decision and reasoning MUST be recorded** on the task for transparency — this is surfaced in steer-co reporting so the customer sees why decisions were made.

**Resource allocation (for MVP-scoped items):**

| Assignment | Criteria |
|---|---|
| **VVF resource** | Industry-specific, functional design, custom development |
| **Externalised (to Odoo)** | Generic issue not industry-specific (e.g., approval workflow) OR technical (e.g., API integration with MES) |
| **Customer resource** | Customer-side configuration, data preparation, UAT ownership |

**Fields at this stage:**
- MVP assignment
- Triage result (mandatory / current practice / deferred)
- Triage justification (text — why this decision was made)
- Responsible party (VVF / Externalised / Customer)
- Deadline

### 4. Blueprint
> Detailed functional specification

- Every requirement that reaches this stage MUST have a complete, detailed blueprint
- **Gate rule:** If there is missing information, the task CANNOT move to Blueprint status. Incomplete = stays in Sorted.
- Blueprint is a structured document covering:
  - Current situation (AS-IS)
  - Target situation (TO-BE)
  - Detailed functional description
  - Data model changes (if any)
  - UI/UX mockups or descriptions
  - Edge cases and business rules
  - Acceptance criteria
- Stored as a Knowledge Base article (see [[#Blueprint & Documentation Strategy]])

### 5. Blueprint Validated
> Signed off by both customer and VVF

- Customer confirms the blueprint matches their expectation
- VVF confirms it is technically feasible and complete
- Dual validation required before development starts

### 6. Development
> Implementation in progress

- Code is being written
- Linked to git branch / PR
- Progress tracked

### 7. Customer Test (UAT)
> Customer validates the implementation against the blueprint

- Deployed to staging environment
- Customer executes test scenarios from acceptance criteria
- Issues logged as sub-tasks

### 8. Production
> Deployed and live

- Requirement is fulfilled
- Available in production environment

---

## Risk Management

Risks are tracked directly on tasks using a lightweight model:

### Risk Level (Selection on `project.task`)

| Level | Meaning |
|---|---|
| **None** | No identified risk (default) |
| **Low** | Minor concern, mitigable within current plan |
| **Medium** | Could impact deadline or scope if not addressed |
| **High** | Blocks progress or threatens MVP delivery |

### Risk Fields

| Field | Type | Description |
|---|---|---|
| `risk_level` | Selection | none / low / medium / high |
| `risk_description` | Text | What is the risk? |
| `risk_mitigation` | Text | What is the mitigation plan? |
| `risk_owner` | Many2one → `res.users` | Who owns the mitigation? |

### Risk in Steer-co Reporting

- High/Medium risks are surfaced automatically in the steer-co report
- Grouped by subject, showing risk description + mitigation + owner
- Gives the customer visibility without requiring a separate risk register

---

## Steer-co Reporting

The steer-co report is the primary customer-facing deliverable. It should be a **one-page PDF per MVP** (or a consolidated view across MVPs).

### KPIs per MVP

| KPI | Description |
|---|---|
| **Subject list with sorting status** | Which subjects are sorted / not sorted, with the triage justification visible |
| **% completion** | Tasks completed vs. total (by count, optionally weighted) |
| **Deadline tracking** | MVP target date vs. current projection |
| **Blocked items** | Tasks flagged as blocked, with reason |
| **Risk summary** | High/Medium risks with description, mitigation, and owner |

### Report Structure (Draft)

```
┌─────────────────────────────────────────────────┐
│  VVF Steer-co Report — [Project] — [MVP Name]  │
│  Date: [date]          Deadline: [target_date]  │
├─────────────────────────────────────────────────┤
│  OVERALL PROGRESS          ████████░░  78%      │
├─────────────────────────────────────────────────┤
│  SUBJECTS                                       │
│  ✓ Sales Order Flow        — 5/5 done           │
│  ◐ Inventory Management    — 3/7 done           │
│  ○ Manufacturing           — 0/4 (not started)  │
│  ⊘ Reporting               — deferred (Post-MVP)│
├─────────────────────────────────────────────────┤
│  RISKS                                          │
│  🔴 MES API specs not received — Owner: Customer│
│     Mitigation: Escalate in next workshop       │
│  🟡 Inventory valuation gap — Owner: VVF        │
│     Mitigation: Blueprint in review             │
├─────────────────────────────────────────────────┤
│  BLOCKED                                        │
│  • Task X — waiting on data from customer       │
├─────────────────────────────────────────────────┤
│  DECISIONS LOG (this period)                    │
│  • Subject "Reporting" deferred to MVP2:        │
│    Reason: not mandatory, no current practice   │
└─────────────────────────────────────────────────┘
```

---

## Blueprint & Documentation Strategy

### The Goal: Documentation = Development Context

Blueprints must serve a dual purpose:
1. **Customer-facing** — clear specification the customer validates
2. **Development context** — the document that Claude / developers use alongside the codebase to ensure consistency

### Approach: Odoo Knowledge Base as Primary Store

Use **Odoo Knowledge Base articles** as the primary blueprint storage:
- Native Odoo integration (linked to tasks via `Many2one`)
- Collaborative editing (customer and VVF can both access)
- Structured templates enforced via article templates
- Searchable, taggable, versionable within Odoo

### V1: Git-backed Obsidian as Source of Truth

The development workflow drives the architecture:
1. **VVF writes** blueprints in Obsidian (fast, AI-assisted, markdown)
2. **Git** versions and shares with colleague
3. **Claude reads** markdown from the repo as development context
4. **PDF export** for customer validation (current practice, works fine)

Obsidian vault lives in the project's git repository:

```
project-repo/
  addons/
    vvf_project_management/
  docs/                          ← Obsidian vault
    .obsidian/
    subjects/
      inventory-management/
        blueprint-stock-valuation.md
      sales-order-flow/
        blueprint-order-to-invoice.md
    templates/
      blueprint-template.md
```

The task's `blueprint_article_id` field (V1) stores a **path reference** to the markdown file rather than linking to a Knowledge Base article.

### V2: Git → Odoo Knowledge Base Sync

A sync script pushes markdown blueprints from the git repo to Odoo Knowledge Base, giving customers self-serve access within Odoo.

**Key design requirement:** The sync must be **multi-target**. The same docs repo may need to publish to different Odoo instances (or other systems in the future) depending on the customer.

#### Sync target configuration via front-matter

Each markdown file declares its sync target(s) in YAML front-matter:

```markdown
---
sync_targets:
  - system: odoo
    url: https://kebony.odoo.com
    db: kebony-production
    article_id: 42
  - system: odoo
    url: https://vvf-internal.odoo.com
    db: vvf-main
    article_id: 15
subject: Inventory Management
status: validated
---
# Blueprint: Stock Valuation
...
```

#### Sync target registry (repo-level)

A central config file defines available targets and credentials reference:

```yaml
# docs/.sync-targets.yml
targets:
  kebony:
    system: odoo
    url: https://kebony.odoo.com
    db: kebony-production
    credentials: KEBONY_ODOO_API_KEY   # env var name
  vvf-internal:
    system: odoo
    url: https://vvf-internal.odoo.com
    db: vvf-main
    credentials: VVF_ODOO_API_KEY
  # future: could be notion, confluence, etc.
```

Then front-matter just references target names:

```markdown
---
sync_targets:
  - target: kebony
    article_id: 42
  - target: vvf-internal
    article_id: 15
---
```

#### Sync mechanism

- **Trigger:** GitHub Action on push to `docs/` folder, or manual script
- **Process:** Read changed markdown → convert to HTML → XML-RPC to each target's `knowledge.article.write()`
- **Direction:** One-way (git → Odoo). Git/Obsidian is the source of truth.
- **Extensible:** The `system` field in target config allows future adapters (Notion, Confluence, etc.)

---

## Planning Module Integration

### V1: Lightweight — No Direct Planning Integration

For the MVP of this module, we do NOT integrate with the Odoo Planning module. Resource allocation is handled via the `responsible_party` field (VVF / Externalised / Customer) and optional assignee on tasks.

### Future (V2): Planning Views per MVP

If reporting needs grow:
- Resource allocation view showing VVF consultant load per MVP
- Timeline view of tasks per subject
- Capacity planning across multiple engagements

This is deferred — the priority is getting the requirement pipeline and steer-co reporting right first.

---

## Data Model (Final Draft)

### Subject (`vvf.project.subject`)

A lightweight classification model — not a task hierarchy.

| Field | Type | Description |
|---|---|---|
| `name` | Char | Subject name (e.g., "Inventory Management") |
| `project_id` | Many2one → `project.project` | Parent project |
| `company_id` | Many2one → `res.company` | Company (engagement) |
| `description` | Html | Subject description / scope |
| `task_ids` | One2many → `project.task` | Tasks tagged with this subject |
| `sequence` | Integer | Display order |
| `color` | Integer | Kanban color index |

### Task extensions (`project.task`)

Extends the standard Odoo task with VVF methodology fields:

| Field | Type | Description |
|---|---|---|
| `subject_id` | Many2one → `vvf.project.subject` | Subject classification |
| `mvp_id` | Many2one → `vvf.project.mvp` | MVP assignment |
| `triage_result` | Selection | `mandatory` / `current_practice` / `deferred` |
| `triage_justification` | Text | Why this triage decision was made (shown in steer-co) |
| `responsible_party` | Selection | `vvf` / `externalised` / `customer` |
| `blueprint_article_id` | Many2one → `knowledge.article` | Link to blueprint in Knowledge Base |
| `blueprint_complete` | Boolean | Computed: can this move to Blueprint stage? |
| `risk_level` | Selection | `none` / `low` / `medium` / `high` |
| `risk_description` | Text | Risk description |
| `risk_mitigation` | Text | Mitigation plan |
| `risk_owner` | Many2one → `res.users` | Risk mitigation owner |

### MVP (`vvf.project.mvp`)

| Field | Type | Description |
|---|---|---|
| `name` | Char | MVP name (e.g., "MVP 1 - Go Live") |
| `project_id` | Many2one → `project.project` | Parent project |
| `company_id` | Many2one → `res.company` | Company |
| `sequence` | Integer | MVP order |
| `target_date` | Date | Target delivery date |
| `description` | Html | MVP scope description |
| `task_ids` | One2many → `project.task` | Tasks in this MVP |
| `task_count` | Integer | Computed: total tasks |
| `task_done_count` | Integer | Computed: tasks in Production stage |
| `progress` | Float | Computed: % completion |

---

## Decisions Log

| # | Decision | Rationale |
|---|----------|-----------|
| D1 | Subject is a task attribute, not a hierarchy level | Enables group-by in any combination; slice-and-dice friendly |
| D2 | One subject per task — no multi-subject | If it spans subjects, the subjects are poorly defined |
| D3 | Triage justification is mandatory and shown in steer-co | Transparency: customer sees why decisions were made |
| D4 | Risk tracked on tasks, not separate register | Lightweight; risks surface in steer-co automatically |
| D5 | No Planning module integration in V1 | Keep it light; `responsible_party` + assignee is enough |
| D6 | Git-backed Obsidian is source of truth; sync to Knowledge Base in V2 | Dev loop drives architecture; multi-target sync supports different customers |
| D7 | Externalised task sync with Odoo support — V2 | Not needed for MVP |

---

## Scope Summary

### V1 (MVP of the module)

- [x] Stage pipeline (8 stages)
- [x] Subject model + task attribute
- [x] MVP model + task assignment
- [x] Triage fields with justification
- [x] Resource allocation (selection field)
- [x] Risk fields on tasks
- [x] Blueprint gate (completeness check)
- [x] Knowledge Base article link
- [x] Steer-co QWeb PDF report
- [x] Group-by views (subject, MVP, stage, responsible party)

### V2 (Future)

- [ ] Git → Odoo Knowledge Base sync (multi-target, front-matter driven)
- [ ] Planning module integration
- [ ] Externalised task ↔ Odoo support ticket sync
- [ ] Cross-engagement dashboard (VVF internal)
- [ ] Time tracking / billing integration

---

## Next Steps

1. ~~Validate specification~~ → In progress
2. Define blueprint article template (Knowledge Base)
3. Scaffold the module
4. Implement stage pipeline and data model
5. Build steer-co QWeb report
