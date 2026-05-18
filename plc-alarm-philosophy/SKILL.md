---
name: plc-alarm-philosophy
description: Generate a standalone ISA-18.2 Alarm Philosophy document for industrial automation. Covers full alarm-management lifecycle — classification, priority, rationalization (with extended register columns + RTM linkage to FDS), setpoints / deadbands, suppression / shelving / inhibit (incl. cascade and equipment-fault suppression), annunciation (multi-station / mobile / HMI failover; ISO 7731 audio), flood management (incl. power-restoration alarm shower), bad-actor management (ISA-18.2 §13), KPIs + monthly KPI report template, alarm-system handover checklist, conditional regulated-industry overlays (Pharma 21 CFR Part 11 with tamper-evidence + EU Annex 11; AWWA / AWIA water; OSHA PSM oil & gas; CCPS chemical; NERC CIP-008 power; NFPA 79 discrete; API RP 1167 pipeline), platform-conditional content (FactoryTalk View SE / ME, Ignition Perspective / Vision, Siemens, Other), conditional priority-scheme emission (3 / 4 / 5-level), and SIS pre-alarm vs trip-alarm distinction. Follows ANSI/ISA-18.2-2016 (R2023), EEMUA 191:2013 Edition 3, IEC 62682:2022 Edition 2, ANSI/ISA-101.01-2015 (R2018), IEC 61511-1:2016 Edition 2, ISA-TR18.2.5 / ISA-TR18.2.7, ISO 7731 / ISO 11429, and 21 CFR Part 11.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [alarm-philosophy, plc, isa-18.2, eemua-191, iec-62682, his]
    ported_from: claude-code/alarm-philosophy
---
# /alarm-philosophy — ISA-18.2 Alarm Philosophy Generator

**Invocation:** `/alarm-philosophy [project-name]`

**Examples:**
- `/alarm-philosophy "Wastewater Lift Station No. 4 Control Upgrade"`
- `/alarm-philosophy "Reactor Area 200 — Batch System"`
- `/alarm-philosophy "Packaging Line 3 — Alarm Rationalization"`

---

## Purpose

Generate a complete ISA-18.2 Alarm Philosophy document — the binding agreement between Hunt Integrative Solutions and the customer on how alarms are defined, rationalized, displayed, suppressed, and managed throughout the system's lifecycle. The same document serves: the customer's process engineer leading rationalization workshops; the lead operator who will use the system at 3 AM; the HMI configuration engineer who builds it; the FDA / NERC CIP / OSHA PSM auditor reviewing it; the site team running monthly KPI reports and MOC. For small projects this content may be folded into the FDS; for mid-to-large projects it is a standalone, customer-approved deliverable that precedes any HMI alarm configuration.

Standards alignment (specific editions matter — reviewers check first):

- **ANSI/ISA-18.2-2016 (R2023)** — Management of Alarm Systems for the Process Industries (primary governing standard)
- **EEMUA 191:2013 (Edition 3)** — Alarm Systems: Design, Management and Procurement (industry KPI benchmark, paired with ISA-18.2)
- **IEC 62682:2022 (Edition 2)** — Management of Alarms for the Process Industry (IEC equivalent, harmonized)
- **ANSI/ISA-101.01-2015 (R2018)** — HMI design (color philosophy, annunciation, cognitive load)
- **IEC 61511-1:2016 (Edition 2)** — SIS for the Process Industry (where SIL-rated functions are present)
- **ANSI/ISA-5.1-2024** — Instrumentation Symbols and Identification (alarm-tag naming)
- **ISA-TR18.2.5-2012** — Alarm Rationalization (technical report)
- **ISA-TR18.2.7-2017** — Alarm Management for Batch and Discrete Processes (technical report)
- **ISO 7731:2003** — Audible danger signals
- **ISO 11429:1996** — Auditory and visual signals for safety / status of machinery
- **21 CFR Part 11** — FDA electronic records (pharma overlay only; in §12 + §19.1)
- Conditional regulated-industry standards layered into Section 19 by industry answer (API RP 1167 / AWWA M2 / AWIA §2013 / NERC CIP / OSHA PSM / CCPS / NFPA 79 / NAMUR NA 102 / GAMP 5).

---

## Instructions

### MANDATORY: AskUserQuestion for ALL Questions

Never ask in plain text. Every question goes through `AskUserQuestion`. **This is a formal interview, not a chat.** Never assume a value the user did not provide. Defaults are only acceptable when explicitly labeled `(Recommended)` inside an option.

The interview runs in **three sequential rounds plus targeted follow-ups**.

---

### Step 1 — Conduct the Project Interview

#### Interview Round 1 — Identity, Customer, Scope, Nexus (ask all 4 together)

```
AskUserQuestion([
  {
    question: "What is the project name and HIS project number?",
    header: "Project ID",
    multiSelect: false,
    options: [{ label: "I'll type it", description: "Use Other for project name + HIS project number." }]
  },
  {
    question: "Who is the customer and where is the site located?",
    header: "Customer",
    multiSelect: false,
    options: [{ label: "I'll type it", description: "Use Other for customer name + site location." }]
  },
  {
    question: "What is the scope of this document?",
    header: "Document Scope",
    multiSelect: false,
    options: [
      { label: "Standalone Alarm Philosophy — full document, separate from FDS", description: "Self-contained; FDS may reference this." },
      { label: "Supplement to existing FDS — replaces FDS Section 9", description: "I'll provide FDS document number; this document governs in case of conflict." },
      { label: "New rationalization for brownfield system — existing alarms need ISA-18.2 audit", description: "Rationalization-driven; existing alarm list becomes starting register." },
      { label: "Template only — no specific project; reusable HIS standard template", description: "Generic template for future projects." }
    ]
  },
  {
    question: "Is there existing plant data indexed in Nexus for this project?",
    header: "Nexus Data",
    multiSelect: false,
    options: [
      { label: "Yes — pull alarms, tags, and interlocks", description: "Pre-populate from Nexus, all flagged [ASSUMED from Nexus — verify with customer]." },
      { label: "No — new project; no Nexus data", description: "Build template from interview only." },
      { label: "Not sure — check Nexus and report findings", description: "Run plant_summary + list_documented_plcs and surface results." }
    ]
  }
])
```

#### Round 1 Follow-Up — Document Revision and Schedule

Immediately after Round 1 (before Round 2), ask:

```
AskUserQuestion([
  {
    question: "Document revision letter for this issue?",
    header: "Doc Rev",
    multiSelect: false,
    options: [
      { label: "A — first issue (Recommended for new projects)", description: "Use Rev A for first issue; increment after every change." },
      { label: "B / C / D / later — continuing existing Alarm Philosophy", description: "I'll specify in Other." },
      { label: "Draft / pre-revision", description: "Internal working copy; do not issue. Filename = -Draft.md." }
    ]
  },
  {
    question: "Project's required-on-site / FAT date?",
    header: "FAT Date",
    multiSelect: false,
    options: [
      { label: "Within 8 weeks — aggressive", description: "" },
      { label: "8–16 weeks — typical", description: "" },
      { label: "16+ weeks — long lead", description: "" },
      { label: "Unknown / not yet set", description: "Skip schedule-relative checks." }
    ]
  }
])
```

---

#### Interview Round 2 — Platform, Industry, Alarm Environment (ask all 4 together)

After Round 1:

```
AskUserQuestion([
  {
    question: "What HMI/SCADA platform is this project using? (Determines platform-conditional content for §3, §8, §14.)",
    header: "Platform",
    multiSelect: false,
    options: [
      { label: "FactoryTalk View SE — server-based, multi-client (plant-wide SCADA)", description: "" },
      { label: "FactoryTalk View ME — PanelView terminal (machine-level)", description: "" },
      { label: "Ignition Perspective — Inductive Automation, mobile-first, web-based", description: "" },
      { label: "Ignition Vision — Inductive Automation, desktop client", description: "" },
      { label: "Both FactoryTalk View SE and ME (plant SCADA + machine panels)", description: "" },
      { label: "Siemens WinCC / WinCC OA", description: "" },
      { label: "Third-party / Other (describe in Other)", description: "" },
      { label: "Not yet determined", description: "Skip platform-conditional emission; emit generic placeholders." }
    ]
  },
  {
    question: "What industry / regulatory environment governs this project? (Drives §19 industry overlay.)",
    header: "Industry",
    multiSelect: false,
    options: [
      { label: "Process / Chemical (OSHA PSM / CCPS)", description: "OSHA 1910.119 § (f) + CCPS instrumented-protective-systems alarm overlay." },
      { label: "Pharmaceutical / Biotech (FDA / EU GMP)", description: "GAMP 5 + 21 CFR Part 11 + EU Annex 11 + ALCOA+ overlay required." },
      { label: "Food & Beverage (FSMA / 3-A)", description: "CIP cycle suppression + sanitary alarm requirements." },
      { label: "Water / Wastewater (AWIA / AWWA M2)", description: "AWWA M2 + AWIA §2013 ERP-coordination overlay." },
      { label: "Oil & Gas (OSHA PSM / API)", description: "OSHA PSM + API RP 554 + (pipeline only) API RP 1167 overlay." },
      { label: "Discrete Manufacturing (NFPA 79 / OSHA)", description: "NFPA 79 alarm-light + ISO 13849-1 safety-stop overlay." },
      { label: "Power Generation / Utilities (NERC CIP)", description: "CIP-008 incident response + CIP-007 system monitoring." },
      { label: "Other / Mixed (describe in Other)", description: "§19 marked Not Applicable unless overlay identified." }
    ]
  },
  {
    question: "How many alarms does this system expect to have (or currently has, brownfield)?",
    header: "Alarm Count",
    multiSelect: false,
    options: [
      { label: "Fewer than 100 — small machine or skid", description: "3-level priority scheme typical." },
      { label: "100–500 — mid-size, single area", description: "4-level scheme typical." },
      { label: "500–2,000 — plant-wide / multi-area", description: "4-level (or 5-level)." },
      { label: "More than 2,000 — large plant rationalization project", description: "5-level scheme typical; rationalization is a major project phase." },
      { label: "Not yet known — estimate after equipment list", description: "" }
    ]
  },
  {
    question: "Greenfield, brownfield rationalization, brownfield migration, or hybrid?",
    header: "Project Mode",
    multiSelect: false,
    options: [
      { label: "Greenfield — designing from scratch", description: "" },
      { label: "Brownfield rationalization — existing alarms audit against ISA-18.2", description: "Existing alarm list becomes starting register." },
      { label: "Brownfield migration — existing alarms migrate to new platform with rationalization", description: "Migration + rationalization simultaneously." },
      { label: "Hybrid — some areas new, some carryover", description: "Per-area rationalization decisions." }
    ]
  }
])
```

---

#### Interview Round 3 — Philosophy Choices, Compliance, Thresholds (ask all 4 together)

After Round 2:

```
AskUserQuestion([
  {
    question: "How many alarm priority levels will this system use? (Drives conditional emission of §4.1.)",
    header: "Priority Scheme",
    multiSelect: false,
    options: [
      { label: "4-level ISA-18.2 standard — Critical/High/Medium/Low + Event (Recommended)", description: "Default for most systems; emits §4.1 4-level option only." },
      { label: "3-level simplified — Emergency/Warning/Advisory + Event", description: "Common for small systems < 100 alarms; emits §4.1 3-level only." },
      { label: "5-level extended — adds Diagnostic level below Low", description: "Common for large process plants > 2,000 alarms; emits §4.1 5-level only." },
      { label: "Customer standard — I'll describe in Other", description: "Custom scheme; document the Customer's priority levels in §4.1." }
    ]
  },
  {
    question: "What alarm suppression and shelving features are required? (multi-select)",
    header: "Suppression / Shelving",
    multiSelect: true,
    options: [
      { label: "Process-state suppression — startup / shutdown / CIP", description: "PLC sequence-driven suppression group; programmatic." },
      { label: "Out-of-service shelving — operator temporarily shelves alarm", description: "Operator-initiated; time-limited; audit-trailed." },
      { label: "First-out annunciation — identify first alarm in cascade", description: "PLC latches first; subsequent alarms tagged CASCADE." },
      { label: "Alarm-flood suppression — auto-shelve low-priority during flood", description: "Automatic during flood state; lifts after flood clears." },
      { label: "Cascade suppression — upstream trip suppresses downstream causal alarms", description: "C&E-Matrix-driven; suppresses naturally-related downstream alarms." },
      { label: "Equipment-fault suppression — faulted equipment suppresses its downstream alarms for fault duration", description: "Persists across fault duration; auto-lifts on equipment recovery." },
      { label: "None — all alarms always active", description: "" }
    ]
  },
  {
    question: "Operator Response Guide (ARG) format?",
    header: "ARG Format",
    multiSelect: false,
    options: [
      { label: "Embedded in rationalization register — ARG text is a column", description: "Single-source register; brief ARG content." },
      { label: "Separate ARG document — full diagnostic guide per Priority 1 / 2 alarm", description: "§15 emits full ARG template for each P1/P2." },
      { label: "Both — register short response + separate doc full ARG for criticals", description: "Two-tier; recommended for plants with > 50 P1/P2 alarms." },
      { label: "Not required — operator response in training, not formal document", description: "§15 marked Not Applicable; rely on operator training records." }
    ]
  },
  {
    question: "Special regulatory or compliance requirements? (multi-select)",
    header: "Compliance",
    multiSelect: true,
    options: [
      { label: "21 CFR Part 11 — FDA electronic records / signatures", description: "§12 + §19.1 emit; tamper-evidence required; setpoint-changes are records." },
      { label: "EU Annex 11 — EU pharma electronic records", description: "Parity with 21 CFR Part 11; § 4 / 9 / 11 / 15 references." },
      { label: "NERC CIP — power industry cybersecurity", description: "CIP-008 alarm-driven incident reporting; CIP-007 system monitoring." },
      { label: "SIS / SIF alarms — IEC 61511 SIS trips with separate alarm tier", description: "§11 emits with pre-alarm / trip / diagnostic / proof-test-overdue distinction." },
      { label: "IEC 62443 cybersecurity — alarm-system access control + logging", description: "Cross-reference /fds §6.3 zones." },
      { label: "OSHA PSM 29 CFR 1910.119 — process safety management", description: "§19 PSM overlay emits." },
      { label: "AWIA §2013 — water utility risk & resilience", description: "§19 Water overlay emits if served-population ≥ 3,300." },
      { label: "No special regulatory requirements", description: "" },
      { label: "Other (describe in Other)", description: "" }
    ]
  }
])
```

#### Round 3 Follow-Up — Operational Thresholds (HIS defaults vs customer values)

```
AskUserQuestion([
  {
    question: "Operational alarm-management thresholds — accept HIS defaults or specify customer values?",
    header: "Thresholds",
    multiSelect: false,
    options: [
      { label: "Accept HIS defaults (Recommended for greenfield)", description: "chatter=10 transitions/24h; flood=10 alarms/10min; shelve P3-4=4h max; shelve P2=1h max; P1 never shelved; cascade window=5 min; backup=daily; MOC turnaround=24h or next business day." },
      { label: "Customize — I'll specify each threshold via Other", description: "Provide values: chatter / flood / shelve P3-4 / shelve P2 / cascade window / backup schedule / MOC turnaround. Free text in Other." }
    ]
  }
])
```

#### Conditional Follow-Up Questions (trigger based on Round 1–3 answers)

- **Nexus = Not sure:** Run `mcp__nexus__list_documented_plcs` and `mcp__nexus__plant_summary`. Report and ask which PLC(s) apply.
- **Nexus = Yes (or confirmed from Not sure):** Run Step 2 below; surface alarm/interlock summary and ask which to carry forward.
- **Brownfield / brownfield migration:** Ask for existing alarm list (FT View export, Ignition tag export, spreadsheet) as starting register.
- **21 CFR Part 11 selected:** Ask "Open vs Closed system per § 11.10 / § 11.30? Tamper-evidence method (digital signature per record / hash-chain / write-once storage)?"
- **SIS / SIF selected:** Ask "Are SIS trips handled in safety PLC and mirrored to BPCS, or BPCS-generated? Pre-alarm setpoints (typically 90% of trip threshold) — confirm or specify."
- **Alarm count > 2,000:** "Generate complete rationalization register up front, or emit philosophy and template that rationalization team fills during workshops?"
- **Industry = Pharma:** "GAMP 5 software category target (Cat 3 / 4 / 5)?"
- **Industry = Power:** "BES Cyber System category — High / Medium / Low Impact per CIP-002?"
- **Industry = Water:** "Population served (drives AWIA §2013 — threshold 3,300)?"
- **Industry = Oil & Gas:** "Pipeline operation (drives API RP 1167 applicability) Y/N?"

---

### Step 2 — Pull Nexus Data (If Available)

If Nexus data exists, run these tools and extract relevant alarm data, tag names, and interlock logic. **Do not present raw Nexus output to the user.** Pre-populate the master register and §13 hierarchy.

| Tool | Purpose |
|---|---|
| `mcp__nexus__plant_summary` | Area / unit structure → §13 alarm hierarchy |
| `mcp__nexus__list_documented_plcs` | Confirm PLCs indexed; identify alarm-relevant programs |
| `mcp__nexus__find_interlocks` | Existing interlock chains → candidate alarms (interlocks often correspond to alarms) |
| `mcp__nexus__rung_search` | Search "OTL", "_Alm", "alarm", "ALM" for existing alarm rungs |
| `mcp__nexus__hmi_project_summary` | Overall HMI alarm config summary |
| `mcp__nexus__screen_inventory` | Identify alarm-display screens |
| `mcp__nexus__generate_io_inventory` | Full I/O list — every field signal is alarm candidate |
| `mcp__nexus__tag_search` | Search _Alm / _ALM / _Fault / _FLT suffixes for existing alarm tags |
| `mcp__nexus__tag_context` | Get description and wiring context for key tags |
| `mcp__nexus__tag_where_used` | Cross-reference alarm tag usage |

**Nexus Skepticism Rules — Non-Negotiable:**

Every Nexus-sourced field is flagged `[ASSUMED from Nexus — verify with customer]`.
- Alarm tags from Nexus may not be rationalized — treat as candidates, not approved alarms.
- Interlock logic reflects code, not necessarily customer intent or current field wiring.
- HMI alarm configurations may include nuisance alarms that have never been rationalized.
- I/O counts may not include recently added field devices.

After collecting Nexus data, summarize via `AskUserQuestion`: number of alarm tags, interlock signals, areas with existing alarm configurations. Ask which areas to include.

---

### Step 3 — Research Applicable Standards

Use Scrapling. Capture the specific clause supporting each requirement.

**Mandatory research:**
1. **ISA-18.2-2016 Core Philosophy** — 10 lifecycle stages; alarm definition (requires action); priority-to-consequence mapping; rationalization worksheet fields.
2. **EEMUA 191 Performance Targets** — KPI benchmarks: avg alarm rate ≤ 1/10 min target, ≤ 6/10 min acceptable; standing-alarm threshold; chattering definition.
3. **ISA-18.2 Suppression and Shelving** — definitions of inhibit vs suppress vs shelve; time-limited shelving rules.
4. **ISA-18.2 Alarm Flood Management** — flood definition (rate > 10/10 min); flood-response protocol; cause-of-alarm analysis.
5. **Priority Rationalization Methods** — consequence × likelihood approach; SIL integration.
6. **ISA-TR18.2.5-2012** Alarm Rationalization — workshop-driven rationalization practice.
7. **ISA-TR18.2.7-2017** Alarm Management for Batch and Discrete — recipe-dependent setpoints; CIP suppression detail.

**Platform-specific research:**

| Platform | Search Terms |
|---|---|
| FactoryTalk View SE | "FactoryTalk View SE alarm configuration ISA-18.2 banner shelving" |
| FactoryTalk View ME | "FactoryTalk View ME PanelView alarm display priority" |
| Ignition Perspective | "Ignition Perspective alarm pipeline shelving priority journal mobile" |
| Ignition Vision | "Ignition Vision alarm status table journal priority" |
| Siemens WinCC | "WinCC alarm handling priority ISA-18.2" |

**Industry-specific research** (per Round 2 industry):

| Industry | Search Terms |
|---|---|
| Pharma | "GAMP 5 alarm rationalization 21 CFR Part 11 audit trail acknowledgment FDA EU Annex 11" |
| Water / WW | "AWWA M2 alarm management water treatment SCADA AWIA §2013" |
| Oil & Gas | "API RP 1167 pipeline alarm management ISA-18.2 SIS SIF" |
| Food & Beverage | "ISA-TR18.2.7 batch CIP suppression FSMA alarm" |
| Power | "NERC CIP-008 alarm incident response CIP-007 system monitoring" |
| Chemical | "CCPS alarm management instrumented protective systems OSHA PSM" |

Use `mcp__scrapling__fetch` with `disable_resources: true` and `main_content_only: true`. Pull 2–3 per topic. Capture clause references.

---

### Step 4 — Generate the Alarm Philosophy Document

Use the template below. Every section is mandatory. Use `[TBD — see Open Items]` for missing data; `[ASSUMED from Nexus — verify with customer]` for Nexus-sourced; never invent values.

**Formatting rules:**
- §4.1 emits exactly one priority scheme matching Round 3 Q1 (use `{{IF_SCHEME_*}}` blocks).
- Platform-specific tag-format and alarm-config examples wrapped in `{{IF_PLATFORM_*}}` blocks; emit only the matching block.
- Tag-naming convention pulled from `/fds` §7.6 if present in save folder; otherwise platform-default.
- Industry overlay §19 emits only the sub-sections matching Round 2 industry.

---

## Alarm Philosophy Document Template

````markdown
# Alarm Philosophy
## {Project Name}
### {HIS Project Number} | Rev {rev} — {Status: DRAFT / FOR REVIEW / APPROVED}

---

| Field | Value |
|---|---|
| Document Number | ALARM-PHIL-{project-slug} |
| Revision | {rev} |
| Status | {DOC_STATUS} |
| Prepared By | Timothy Hunt, Hunt Integrative Solutions LLC |
| Preparation Date | {date} |
| Customer | {customer name} |
| Site | {site location} |
| HMI / SCADA Platform | {platform from Round 2} |
| Industry | {industry from Round 2} |
| Required-on-site / FAT Date | {fat_date} |
| Related FDS | {FDS-{slug} Rev {X.X} — or "Standalone — no associated FDS"} |
| Customer Approval | __________________ Date: ________ |
| HIS Approval | __________________ Date: ________ |

---

## Table of Contents

1. Purpose and Scope (1.1 Purpose · 1.2 Scope · 1.3 Standards · 1.4 Traceability to FDS)
2. Alarm Management Lifecycle
3. Alarm Definition (3.1 What Is an Alarm · 3.2 Alarms vs Events vs Status)
4. Alarm Priority System (4.1 Priority Scheme · 4.2 Priority Assignment · 4.3 Distribution Target)
5. Alarm Rationalization Process (5.1 Principle · 5.2 Team · 5.3 Worksheet Fields · 5.4 Sign-Off · 5.5 Workshop Agenda Template)
6. Alarm Setpoint and Deadband Philosophy (6.1 Setpoint Hierarchy · 6.2 Deadband · 6.3 On/Off-Delay · 6.4 Recipe-Dependent Setpoints · 6.5 Bad Quality Alarm Tier)
7. Suppression / Shelving / Inhibit (7.1 Definitions · 7.2 Process-State · 7.3 Operator Shelving · 7.4 Inhibit · 7.5 Cascade Suppression · 7.6 Equipment-Fault Suppression)
8. Alarm Annunciation (8.1 Visual · 8.2 Audio per ISO 7731 · 8.3 Banner · 8.4 History · 8.5 Multi-Station Ack · 8.6 Mobile / Remote Operator · 8.7 HMI Failover)
9. Alarm Flood Management (9.1 Flood Definition · 9.2 Response · 9.3 First-Out · 9.4 Cause-of-Alarm Analysis · 9.5 Power-Restoration Alarm Shower)
10. Alarm Performance KPIs (10.1 KPI Definitions · 10.2 KPI Review Process · 10.3 Monthly KPI Report Template)
11. Safety Instrumented System Alarms (11.1 Policy · 11.2 SIS Alarm Integration · 11.3 SIS Pre-Alarm · 11.4 SIS Diagnostic Alarms · 11.5 Proof-Test-Overdue Alarm · 11.6 SIS Bypass Register)
12. 21 CFR Part 11 Alarm Requirements (12.1 Electronic Records · 12.2 Reason Codes · 12.3 Open vs Closed System · 12.4 Tamper-Evidence · 12.5 Setpoint-Change Records · 12.6 Backup Validation)
13. Alarm Hierarchy by Area (13.1–13.4 Per-Area · 13.5 Bad-Actor Management Process)
14. Master Alarm Rationalization Register
15. Operator Alarm Response Guides (ARGs)
16. Management of Change (16.1 When · 16.2 Standard MOC · 16.3 Emergency MOC · 16.4 Alarm-System Handover Checklist)
17. Open Items and TBDs
18. Glossary
19. Industry-Specific Alarm Management Overlay (conditional)
20. Revision History

---

## Revision History

| Rev | Date | Author | Description |
|---|---|---|---|
| {rev} | {date} | T. Hunt | Initial draft — for customer review |

---

## Approval Signatures

| Role | Name | Signature | Date |
|---|---|---|---|
| Prepared By | Timothy Hunt / HIS | | |
| Reviewed By (HIS) | | | |
| Approved By (Customer Controls Engineering) | | | |
| Approved By (Customer Operations / Lead Operator) | | | |
| Approved By (Customer Functional Safety, if SIS in scope) | | | |
| Approved By (Customer QA, if 21 CFR Part 11 in scope) | | | |

**This document is not valid for HMI alarm configuration until all approval signatures are obtained. Any change to the alarm philosophy after approval requires a Management of Change (MOC) record and a revision to this document.**

---

## 1. Purpose and Scope

### 1.1 Purpose

This Alarm Philosophy establishes the governing rules for design, rationalization, configuration, and management of the alarm system for {project name}, in accordance with **ANSI/ISA-18.2-2016 (R2023)** and **EEMUA 191:2013 (Edition 3)**.

This document defines:
- The formal definition of an alarm and how alarms are distinguished from events and status messages
- The alarm priority scheme and the criteria for assigning each priority level
- The alarm rationalization process — how every alarm is justified, documented, and approved
- Alarm setpoint, deadband, and on/off-delay rules
- Alarm suppression, shelving, and inhibit philosophy (including cascade and equipment-fault suppression)
- Alarm annunciation requirements (visual + audio per ISO 7731; multi-station; mobile; HMI failover)
- Alarm flood management thresholds and response (including power-restoration alarm shower)
- Bad-Actor Management process per ISA-18.2 §13
- Key Performance Indicators (KPIs) and targets
- The master Alarm Rationalization Register
- The process for managing changes to the alarm system after commissioning

**This document is the binding agreement between Hunt Integrative Solutions and the customer on how alarms are defined and managed. HMI alarm configuration must conform exactly to this document. No alarm shall be added, modified, or deleted after commissioning without following the Management of Change process defined in §16.**

### 1.2 Scope

**In scope:** {process areas, equipment, HMI systems covered}.

**Out of scope:** {adjacent systems with own philosophy; safety PLC SIS managed under separate SRS; vendor-packaged unit alarms not integrated into BPCS}.

**Relationship to FDS:**
{If standalone: "This document is standalone. Alarm philosophy content in any FDS shall reference this document and shall not be modified independently."
If supplement: "This document supplements {FDS-{slug}} and replaces Section 9 of that FDS. In conflict, this document governs."}

### 1.3 Standards and References

| Standard / Publication | Title | Version / Year | Relevance |
|---|---|---|---|
| ANSI/ISA-18.2-2016 (R2023) | Management of Alarm Systems for the Process Industries | 2016 (re-aff. 2023) | Primary governing standard |
| EEMUA 191:2013 | Alarm Systems: Design, Management and Procurement | Edition 3 | KPI benchmark; rationalization practice |
| IEC 62682:2022 | Management of Alarms for the Process Industry | Edition 2 | IEC equivalent of ISA-18.2 |
| ANSI/ISA-101.01-2015 (R2018) | Human-Machine Interfaces | 2015 (R2018) | Color philosophy, annunciation, cognitive load |
| ANSI/ISA-5.1-2024 | Instrumentation Symbols and Identification | 2024 | Alarm-tag naming convention |
| ISA-TR18.2.5-2012 | Alarm Rationalization (technical report) | 2012 | Workshop-driven rationalization practice |
| ISA-TR18.2.7-2017 | Alarm Management for Batch and Discrete Processes | 2017 | Recipe-dependent setpoints; CIP suppression |
| IEC 61511-1:2016 (Edition 2) | Functional Safety — Safety Instrumented Systems | 2016 | If SIS in scope |
| ISO 7731:2003 | Audible danger signals | 2003 | Audio annunciation standard |
| ISO 11429:1996 | Auditory and visual signals — safety / status | 1996 | Tactile / accessibility signaling |
| {21 CFR Part 11 — pharma} | Electronic Records; Electronic Signatures | FDA | If pharma overlay applies |
| {EU Annex 11 — pharma} | Computerised Systems | EU GMP | Pharma EU export overlay |
| {API RP 1167 — pipeline} | Pipeline SCADA Alarm Management | 2nd Ed (2016) | If oil & gas pipeline |
| {AWWA Manual M2 — water} | Automation of Water Resource Recovery Facilities | latest | If water/wastewater |
| {NERC CIP-008 — power} | Cyber Security — Incident Reporting and Response Planning | latest | If power-sector |
| {OSHA 29 CFR 1910.119 — process safety} | Process Safety Management | OSHA | If oil & gas / chemical |
| {CCPS — chemical} | Guidelines for Safe and Reliable Instrumented Protective Systems | 2007 | If chemical |
| {NAMUR NA 102 — EU pharma} | Alarm Management | latest | If EU pharma export |
| {GAMP 5 — pharma} | A Risk-Based Approach to Compliant GxP Computerized Systems | 2nd Ed (2022) | If pharma |
| {FDS-{slug}} | Functional Design Specification | Rev {X.X} | Parent project specification |
| {Customer Standard} | {Customer alarm or engineering standard} | {Rev} | Customer-specific requirements |

### 1.4 Traceability to Functional Design Specification

If `FDS-*-Rev*.md` exists in the save folder, parse §1.4 RTM and §11 SIF table. List every Req ID involving alarm requirements:

| FDS Req ID | FDS Section | Alarm Philosophy Section / Register Entry | Status |
|---|---|---|---|
| REQ-ALARM-XXX | FDS §9 | §14 register row {N} | Open / Verified |
| REQ-SAFETY-XXX | FDS §11 SIF-01 | §11.2 SIS-mirror table | Open / Verified |
| {{REQ-XXX}} | {{FDS §}} | {{this doc §}} | Open |

If no FDS exists, mark "Not applicable — no parent FDS in this project folder."

---

## 2. Alarm Management Lifecycle

Per ISA-18.2, alarm management is a continuous lifecycle.

| Stage | ISA-18.2 Description | This Project |
|---|---|---|
| 1. Philosophy | Establish governing rules | **This document** |
| 2. Identification | Identify potential alarm sources | Equipment list (FDS / IO list) + Nexus signal inventory |
| 3. Rationalization | Justify each alarm | §14 Master Register |
| 4. Basic Alarm Design | Configure per rationalized settings | HMI alarm config + PLC alarm rungs |
| 5. Detailed Design | Suppression, shelving, first-out, cascade | §7 + PLC state-based suppression routines |
| 6. Implementation | Build and test | FAT alarm test (FDS §15) |
| 7. Operations | Monitor; respond per ARGs | §15 + Operator training |
| 8. Monitoring & Assessment | Periodic KPI review; bad-actor analysis | §10 monthly + §13.5 monthly bad-actor |
| 9. Audit | Annual audit vs philosophy | Annual rationalization-register-vs-active-alarm comparison |
| 10. Management of Change | Controlled alarm changes | §16 MOC |

---

## 3. Alarm Definition

### 3.1 What Is an Alarm?

Per **ISA-18.2 §3.2.6:**

> *"An audible and/or visible means of indicating to the operator an equipment malfunction, process deviation, or abnormal condition requiring a response."*

For an alert to qualify as an alarm on this project, it must satisfy ALL of the following:

1. **Requires operator action** — operator must do something. If no action required, it's an Event or Status.
2. **Action must be possible** — operator can respond within required response time.
3. **Not a normal operating condition** — deviation from intended operating range.
4. **Not a nuisance** — does not repeat frequently in normal operation. Chattering or fleeting alarms shall be rationalized out.

### 3.2 Alarms vs Events vs Status

{{IF_PLATFORM_FT_VIEW_SE}}
| Type | Definition | Requires ACK? | Recorded? | FT View SE Tag Convention | Example |
|---|---|---|---|---|---|
| **Alarm** | Abnormal; requires response | Yes | Yes — alarm history | `{Area}{EquipID}_Alm` | High-high level trip |
| **Event** | Notable; no action | No | Yes — event log | `{Area}{EquipID}_Evt` | Operator mode change |
| **Status** | Current state | No | No (real-time) | `{Area}{EquipID}_Sts` | Motor running |
| **Diagnostic** | Control-system health | Optional | Optional | `{Area}{EquipID}_Diag` | PLC task overrun |
{{IF_PLATFORM_IGNITION_PERSPECTIVE}}
| Type | Definition | Requires ACK? | Recorded? | Ignition Tag Convention | Example |
|---|---|---|---|---|---|
| **Alarm** | Abnormal; requires response | Yes | Yes — alarm journal | `Area_EquipID_Alarm` | High-high level trip |
| **Event** | Notable; no action | No | Yes — event log | `Area_EquipID_Event` | Operator command |
| **Status** | Current state | No | No (real-time) | `Area_EquipID_Status` | Motor running |
| **Diagnostic** | Control-system health | Optional | Optional | `Area_EquipID_Diag` | OPC quality bad |
{{IF_PLATFORM_SIEMENS_WINCC}}
| Type | Definition | Requires ACK? | Recorded? | WinCC Convention | Example |
|---|---|---|---|---|---|
| **Alarm** | Abnormal | Yes | Yes — Alarm Logging | DB.Alarm bit | High-high trip |
| **Event** | Notable | No | Yes — operating message | DB.Event bit | Mode change |
| **Status** | Current state | No | Real-time | DB.Status bit | Running |
{{IF_PLATFORM_OTHER}}
| Type | Definition | Requires ACK? | Recorded? | Tag Convention | Example |
|---|---|---|---|---|---|
| Alarm | Abnormal; requires response | Yes | Yes | Per project convention | High-high trip |
| Event | Notable; no action | No | Yes | Per project convention | Operator command |
| Status | Current state | No | Real-time | Per project convention | Motor running |

**Rule:** Every alert that a PLC programmer or HMI engineer considers adding must be classified using this table before configuration. If it cannot meet all four §3.1 criteria, it is an Event or Status and shall NOT appear in the alarm system.

---

## 4. Alarm Priority System

### 4.1 Priority Scheme

{{IF_SCHEME_4_LEVEL}}
**4-Level ISA-18.2 Standard** (selected per Round 3 Q1):

| Priority | Name | Severity / Consequence | Required Response Time | Visual | Audio |
|---|---|---|---|---|---|
| 1 | Critical | Immediate safety risk, environmental incident, equipment damage. Potential injury. | ≤ 5 minutes | Red bg — flashing 1 Hz | Continuous audible per ISO 7731 |
| 2 | High | Significant process upset, production loss, equipment damage if not responded. | ≤ 15 minutes | Red text on yellow bg | Intermittent audible |
| 3 | Medium | Process outside normal limits; approaching critical threshold. | ≤ 60 minutes | Yellow | No audible — visual only |
| 4 | Low | Advisory — outside preferred range, no production impact. | Next shift | Cyan / White | None |
| Event | — | Informational. No action. | N/A | Event log only | None |
{{IF_SCHEME_3_LEVEL}}
**3-Level Simplified** (selected per Round 3 Q1, common for systems < 100 alarms):

| Priority | Name | Severity / Consequence | Required Response Time | Visual | Audio |
|---|---|---|---|---|---|
| 1 | Emergency | Immediate safety / equipment damage risk. | ≤ 5 minutes | Red — flashing | Continuous audible per ISO 7731 |
| 2 | Warning | Process upset; action required current operating window. | ≤ 30 minutes | Yellow | Intermittent audible |
| 3 | Advisory | Operator awareness; no immediate action. | Next shift | Cyan | None |
| Event | — | Informational. | N/A | Event log | None |
{{IF_SCHEME_5_LEVEL}}
**5-Level Extended** (selected per Round 3 Q1, common for large process plants > 2,000 alarms):

| Priority | Name | Severity / Consequence | Required Response Time | Visual | Audio |
|---|---|---|---|---|---|
| 1 | Critical | Immediate safety / environmental. | ≤ 5 minutes | Red — flashing | Continuous audible per ISO 7731 |
| 2 | High | Process upset, production impact. | ≤ 15 minutes | Red | Intermittent audible |
| 3 | Medium | Abnormal, approaching limit. | ≤ 60 minutes | Yellow | No audible |
| 4 | Low | Advisory, preferred range exceeded. | Next shift | Cyan | None |
| 5 | Diagnostic | Control system / instrument health. | Maintenance window | White | None |
| Event | — | Informational. | N/A | Event log | None |
{{IF_SCHEME_CUSTOMER}}
**Customer-Defined Priority Scheme** (per Round 3 Q1 Other answer):

{Document the customer's priority levels with names, severity, response time, visual, audio. Cross-reference customer's alarm-management standard.}

### 4.2 Priority Assignment Method

Priority is assigned during rationalization using consequence × urgency matrix:

| Consequence | Urgency: High (< 15 min) | Urgency: Medium (< 1 h) | Urgency: Low (< 8 h) |
|---|---|---|---|
| Safety / Environmental | **Priority 1** | **Priority 1** | **Priority 2** |
| Equipment Damage | **Priority 1** | **Priority 2** | **Priority 3** |
| Production Loss (significant > $100k) | **Priority 2** | **Priority 2** | **Priority 3** |
| Production Loss (minor < $100k) | **Priority 2** | **Priority 3** | **Priority 4** |
| Quality Degradation | **Priority 3** | **Priority 3** | **Priority 4** |
| Informational / Trending | — | — | **Event** |

**Rules:**
- When in doubt, assign the higher priority. May be reduced post-operation.
- Priority 1 alarms shall be reviewed by both HIS engineer and customer's process engineer; two-signature rationalization required.
- SIF trip alarms shall be Priority 1 by default; not suppressed or shelved (see §11).

### 4.3 Alarm Distribution Target

Per ISA-18.2 / EEMUA 191, alarm-priority distribution should be weighted toward lower priorities.

{{IF_SCHEME_4_LEVEL}}
| Priority | Target % of Total |
|---|---|
| Priority 1 (Critical) | < 5% |
| Priority 2 (High) | < 15% |
| Priority 3 (Medium) | < 30% |
| Priority 4 (Low) | > 50% |
{{IF_SCHEME_3_LEVEL}}
| Priority | Target % of Total |
|---|---|
| Priority 1 (Emergency) | < 10% |
| Priority 2 (Warning) | < 30% |
| Priority 3 (Advisory) | > 60% |
{{IF_SCHEME_5_LEVEL}}
| Priority | Target % of Total |
|---|---|
| Priority 1 (Critical) | < 5% |
| Priority 2 (High) | < 15% |
| Priority 3 (Medium) | < 25% |
| Priority 4 (Low) | < 30% |
| Priority 5 (Diagnostic) | > 25% |

If actual distribution deviates significantly at FAT, the register shall be reviewed before commissioning.

---

## 5. Alarm Rationalization Process

### 5.1 Rationalization Principle

Per **ISA-18.2 §6.3** and **ISA-TR18.2.5**: every alarm is rationalized before configuration. Rationalization documents why the alarm exists, what the operator does, and what setpoints / delays apply. An alarm that cannot be rationalized is deleted or re-classified as event.

### 5.2 Rationalization Team

| Role | Responsibility |
|---|---|
| HIS Automation Engineer | Facilitates session; documents results; validates against this philosophy |
| Customer Process Engineer | Defines consequences, response times, setpoints |
| Lead Operator | Validates response actions; confirms response time achievable; identifies nuisance alarms |
| Safety Engineer (if SIS) | Reviews Priority 1 alarms for SIF overlap; confirms SIS trips not duplicated as actionable BPCS alarms |
| QA Representative (if pharma) | Reviews 21 CFR Part 11 audit-trail compliance |

Sessions conducted area by area; one Unit / Equipment Module (ISA-88 level) per session.

### 5.3 Rationalization Worksheet Fields

For each alarm:

| Field | Description | Required? |
|---|---|---|
| Alarm Tag | Full tag with area prefix and _Alm suffix | Mandatory |
| Alarm Description | Plain-English (≤ 40 chars HMI display) | Mandatory |
| Area / Unit | ISA-88 area / unit | Mandatory |
| Priority | Per §4.2 matrix | Mandatory |
| Cause | Physical condition causing alarm | Mandatory |
| Consequence | What happens if no response within required time | Mandatory |
| Operator Response | Step-by-step action | Mandatory |
| Setpoint | Activation value | Mandatory |
| Deadband | Hysteresis (prevents chattering) | Mandatory |
| On-Delay | Persistence before alarm activates | Mandatory (may be 0) |
| Off-Delay | Hold-time after condition clears | Mandatory (may be 0) |
| Suppression Logic | Process states that suppress; reference sequence step | If applicable |
| Cascade Suppression | Upstream alarm that suppresses this one | If applicable |
| Equipment-Fault Suppression | Equipment whose fault suppresses this | If applicable |
| ARG Reference | §15 section number | P1 + P2 mandatory |
| Alarm Source / Origin | PLC interlock / Instrument OOR / Comms loss / Sequencer fault / SIS Mirror / Other | Mandatory |
| HMI Display Path | Comma-separated screen IDs that show this alarm | Mandatory |
| PLC Routine / Logic Reference | Studio 5000 routine + rung; Siemens FB; Ignition tag binding path | Mandatory |
| RTM Req ID | FDS §1.4 cross-reference | If FDS exists |
| First Activation Date | Populated post-commissioning | KPI report |
| Last Activation Date | Populated post-commissioning | KPI report |
| Activation Count (last 12 months) | Populated post-commissioning | KPI report |
| Rationalized By | Process engineer name + role | Mandatory |
| Rationalization Date | Date rationalization completed | Mandatory |

### 5.4 Rationalization Sign-Off

- Every alarm requires sign-off from Customer Process Engineer + Lead Operator before configuration.
- Priority 1 additionally requires HIS Automation Engineer sign-off.
- Blank "Rationalized By" = configuration hold; alarm shall not be built.
- Alarms discovered during commissioning that were not on original list shall be rationalized via emergency MOC (§16.3) before adding.

### 5.5 Rationalization Workshop Agenda Template

Standard 1-hour rationalization workshop:

| Time | Activity |
|---|---|
| 0–5 min | Review prior session's open items; confirm action-item status |
| 5–40 min | Walk through next 10–15 alarms: cause, consequence, response, priority, setpoint, deadband, on/off-delay, suppression, ARG. Record decisions in real-time in the register. |
| 40–50 min | Sign-off (Customer Process Engineer + Lead Operator + HIS Engineer). For Priority 1 alarms, two-signature rationalization confirmed. |
| 50–60 min | Log open items; schedule next session; identify needed information |

Rationalization workshops are scheduled weekly (typical) until register is complete. Pace: ~10–15 alarms per 1-hour session.

---

## 6. Alarm Setpoint and Deadband Philosophy

### 6.1 Setpoint Hierarchy

```
Safety Limit (SIL-rated SIS trip — NEVER exceeded by any alarm)
  │
  ├─ SIS Pre-Alarm (typically 90% of SIS trip setpoint — see §11.3)
  │
  ├─ Critical Alarm (Priority 1) — set BELOW SIS pre-alarm
  │
  ├─ High Alarm (Priority 2) — set below Critical
  │
  ├─ Medium Alarm (Priority 3) — operational limit
  │
  └─ Low / Advisory (Priority 4) — preferred range
```

**Rules:**
- Gap between adjacent priority setpoints ≥ 5–10% of instrument span (smaller gaps require justification).
- Alarm setpoints shall NOT be set at instrument range limits (0% or 100%) — that's a Bad Quality alarm (§6.5).

### 6.2 Deadband Rules

| Rule | Value |
|---|---|
| Minimum deadband for analog alarms | ≥ 1% of span, or ≥ 2× measurement noise (whichever larger) |
| Typical deadband | 2–5% of span |
| Discrete (digital) alarms | Not applicable — use On-Delay instead |
| Maximum deadband | 10% of span (beyond this, move setpoint not deadband) |

**Chattering threshold (per Round 3 follow-up answer):** activation/deactivation > {chatter_threshold} times per 24 h. HIS default: 10/24 h. Chattering alarms reported in monthly KPI report; resolved within 30 days.

### 6.3 On-Delay and Off-Delay Rules

| Delay Type | Purpose | Typical | Maximum |
|---|---|---|---|
| On-Delay | Prevent transient nuisance alarms | 0–10 s | 60 s (process-justified) |
| Off-Delay | Hold alarm briefly to prevent chattering | 0–5 s | 30 s |

On-delay shall NOT mask real process problems. If > 60 s on-delay proposed, re-rationalize the alarm.

### 6.4 Recipe-Dependent Alarm Setpoints (Conditional — Batch Processes Only)

For batch processes with recipe management (per /fds §8.7), some alarm setpoints (reactor-jacket high-temp, agitator high-current) vary by recipe:

| Recipe | Alarm Tag | Setpoint A | Setpoint B | Setpoint C |
|---|---|---|---|---|
| RX-100 (Standard) | TT-101_Hi_Alm | 80 °C | — | — |
| RX-200 (Hot) | TT-101_Hi_Alm | — | 95 °C | — |
| RX-300 (Cold) | TT-101_Hi_Alm | — | — | 65 °C |

Recipe-load downloads new setpoints to PLC alarm logic. Alarm-history records the recipe in effect at activation. Cross-reference /fds §8.7.

If batch / recipe not in scope: "Not applicable — no recipe-dependent alarms in this system."

### 6.5 Bad Quality Alarm Tier

A Bad Quality alarm fires when:
- AI signal is at exact 0% or 100% of instrument span (suggests broken wire / failed transmitter)
- PLC tag quality flag = Bad
- HART communication loss to instrument
- 4-20mA loop current outside 3.6–22 mA range

| Loop Class | Bad Quality Priority | Justification |
|---|---|---|
| Non-critical loop | Priority 3 (Medium) | Maintenance follow-up; no immediate process risk |
| Critical loop (tied to Priority 1 alarm) | Priority 2 (High) | Lost visibility on critical signal; operator must act |
| SIF-tied loop | Priority 1 (Critical) | Lost visibility on safety-critical signal; immediate action required |

On-Delay: 10 s minimum to debounce momentary signal dropouts.

---

## 7. Suppression / Shelving / Inhibit

### 7.1 Definitions

Per ISA-18.2, these terms have specific meanings — NOT interchangeable.

| Term | ISA-18.2 Definition | When to Use | Audit Trail |
|---|---|---|---|
| **Suppress** | Programmatic block based on a condition | Process-state, cascade, equipment-fault | Yes — logged with reason |
| **Shelve** | Operator-applied temporary suppression | Equipment offline for maintenance | Yes — operator, timestamp, reason, duration |
| **Inhibit** | Engineering-level block | Instrument out of service; loop under commissioning | Yes — supervisor approval + work order ref |
| **Out-of-Service (OOS)** | Formal deactivation | Equipment decommissioned or not yet in service | Yes — MOC required |

### 7.2 Process-State Suppression

| Suppression Group | Active During | Alarms Suppressed | Duration Limit |
|---|---|---|---|
| Startup Suppression — {Area} | Steps S01–S{N} of Startup | {alarm tags expected during startup} | Sequence duration; auto-lifts at Running |
| Shutdown Suppression — {Area} | Shutdown Sequence active | {alarms expected during shutdown} | Sequence duration |
| CIP Suppression — {Area} | CIP sequence active | {alarms expected during CIP} | CIP-cycle duration; per CIP step |

**Rules:**
- Suppression is programmatic (PLC sequence steps or mode state machine); operators shall NOT manually suppress by removing setpoints / forcing tags.
- Every suppression logged: alarm tag, start time, reason, end time.
- No alarm shall remain suppressed indefinitely. If always suppressed in normal operation, rationalize it out.

### 7.3 Operator Shelving Rules (per Round 3 follow-up thresholds)

| Parameter | Rule |
|---|---|
| Maximum shelve duration — Priority 3–4 | {shelve_p34_max_hours} h (HIS default: 4 h) |
| Maximum shelve duration — Priority 2 | {shelve_p2_max_hours} h (HIS default: 1 h) |
| Maximum shelve duration — Priority 1 | **Never shelved.** No exceptions. |
| Shelve extension | Allowed once per alarm instance, same duration, new reason code |
| Shelve expired — action | Auto-returns to active state; operator notified |
| Shelving access level | Minimum {Operator / Supervisor} security level |
| Audit trail | Operator name, timestamp, alarm tag, reason code (defined list), duration |

**Priority 1 alarms shall never be shelved.** If a Priority 1 condition exists, it must be resolved or process must shut down.

### 7.4 Inhibit Rules

- Inhibit access requires {Supervisor / Engineer} security level.
- Inhibit must reference active work order number or maintenance record.
- Maximum inhibit duration: {8 h / 1 shift} without re-authorization.
- All inhibits logged + work order reference.
- Engineering inhibits in HMI configuration (rather than operator shelves) require MOC.

### 7.5 Cascade Suppression

When upstream "trip" alarm fires, automatically suppress downstream alarms causally related.

**Example:** M-101 motor trip → automatically suppress (a) FT-101 low-flow alarm immediately downstream, (b) TT-101 high-temp alarm because cooling water no longer required.

Per-equipment cascade map maintained as part of Master Register §14. Cross-references the Cause & Effect Matrix in /soo §6.4 if available.

| Upstream Alarm | Trigger | Suppresses (Downstream) | Lift Condition |
|---|---|---|---|
| AL-MOTOR-TRIP-101 | M-101 trip | AL-FLOW-LOW-101, AL-TEMP-HIGH-101 | M-101 returns to Stopped or Running |
| {add per upstream→downstream} | | | |

### 7.6 Equipment-Fault Suppression

Distinct from §7.5 cascade (which is event-driven on trip): equipment-fault suppression persists across the fault duration.

When equipment is in Faulted state:
- Suppress non-trip alarms downstream of that equipment for the fault duration.
- Lift automatically when equipment returns to Stopped or Running state.

**Example:** M-101 in Faulted state → suppress FT-101 low-flow + TT-101 high-temp until M-101 cleared.

---

## 8. Alarm Annunciation

### 8.1 Visual Annunciation

Per **ANSI/ISA-101.01-2015 (R2018)** and **ISA-18.2**, color in HMI is reserved for conveying information.

| Alarm State | Background | Text | Flashing? |
|---|---|---|---|
| Priority 1 — Unacknowledged | Red | White | Yes — 1 Hz |
| Priority 1 — Acknowledged, still active | Red | White | No |
| Priority 2 — Unacknowledged | Red | Black | Yes — 1 Hz |
| Priority 2 — Acknowledged, still active | Red | Black | No |
| Priority 3 — Unacknowledged | Yellow | Black | No |
| Priority 3 — Acknowledged, still active | Yellow | Black | No |
| Priority 4 — Unacknowledged | Cyan | Black | No |
| Cleared, Unacknowledged (return-to-normal) | White / Gray | Black | No |
| Shelved | Gray | Gray | No |
| Suppressed | Not displayed | — | — |

**HMI baseline per ISA-101:**
- Normal equipment: gray (#A8A8A8)
- Running equipment: muted green (low saturation per ISA-101 reduced-emphasis recommendation)
- Alarm-only color saturation: high

### 8.2 Audio Annunciation (per ISO 7731:2003 + ISO 11429:1996)

| Priority | Audio | Silencing |
|---|---|---|
| Priority 1 | Continuous tone — distinct, ≥ 15 dB above ambient noise per ISO 7731 §3.1; modulation 500 Hz–2.5 kHz; SPL ≥ 65 dB | Operator acknowledgment only. Re-sounds if new Priority 1 activates while previous unacknowledged. |
| Priority 2 | Intermittent tone — different from Priority 1 | Operator acknowledgment |
| Priority 3 | No audio — visual only | N/A |
| Priority 4 | No audio — visual only | N/A |

**Audio system requirements:**
- ≥ 15 dB above ambient at operator workstation (ISO 7731 minimum).
- Horn / speaker wired independently of HMI workstation — HMI failure shall not silence horn.
- Horn-silence / acknowledge button required at operator panel for Priority 1 in facilities where operators not always at HMI.
- For hearing-impaired operators: tactile annunciation (vibration, flashing strobe) per ISO 11429.
- For multilingual sites: alarm text in primary + secondary language.

### 8.3 HMI Alarm Banner

Visible on every HMI screen — operator never navigates away to see active alarms.

**Mandatory alarm banner content:**
- Count of unacknowledged Priority 1
- Count of unacknowledged Priority 2
- Total active alarms count
- Most recent unacknowledged alarm message (tag + description + time)
- Navigation button to Alarm Summary screen
- Override / Force / Bypass count (per /fds §7.9)

**Alarm Summary screen:**
- Sortable: Priority (default), Time, Area, Tag
- Filterable: Priority, Area, Acknowledged/Unacknowledged, Active/Cleared
- Columns: Priority, Tag, Description, Area, Time, Duration, Acknowledged (Y/N), Acknowledged By
- No alarm hidden by default; suppressed alarms not shown; all active alarms shown.

### 8.4 Alarm History

| Requirement | Rule |
|---|---|
| Minimum retention | 90 days on HMI server / historian |
| Fields per event | Tag, Description, Priority, Area, Time of activation, Time of acknowledgment, Acknowledged by (user), Time of return-to-normal, Suppression state at activation |
| Tamper protection | Records non-editable after creation. Deletion requires MOC + supervisor approval. |
| Backup | Included in scheduled HMI backup. Schedule: {backup_frequency} (HIS default: daily) |
| Export | CSV / Excel for monthly KPI reporting |
| {21 CFR Part 11 only — see §12} | Acknowledgment records constitute electronic records; e-signatures linked; audit trail read-only |

### 8.5 Multi-Station Acknowledgment

For systems with multiple operator workstations:

| Behavior | Rule |
|---|---|
| Acknowledgment on Station 1 | Propagates to all stations; "ack'd by {user} at {HH:MM}" displayed on all |
| Multiple stations alarm simultaneously | All stations show identical alarm; first-ack wins; timestamp resolved to single record |
| Per-station-required-ack alarms | Defined explicitly (typically Priority 1 in safety-critical zones); each station must ack independently |
| Station offline at ack time | Sync on reconnect; ack history backfills |

### 8.6 Mobile / Remote Operator (Conditional — Ignition Perspective / mobile platforms only)

| Feature | Rule |
|---|---|
| Push notification | Priority 1 + Priority 2 only; vibration pattern by priority |
| Vibration pattern | Priority 1: 3× short pulses + 1× long; Priority 2: 2× short pulses |
| Time-to-respond on mobile | Logged separately; KPI tracks mobile-only response time |
| Offline behavior | Notifications queue; deliver on reconnect with original timestamp |
| MFA requirement | Required for remote acknowledgment (per /fds §6.3 IEC 62443 SR controls) |
| Privacy / on-call | Operator can disable notifications during off-shift; Priority 1 always-deliver override |

### 8.7 HMI Failover Behavior

For redundant HMI server architectures:

| Scenario | Behavior |
|---|---|
| Primary HMI fails | Backup HMI takes over; pending-alarm queue mirrors to backup; operator continues seamlessly |
| Acknowledged-but-not-yet-cleared alarms | Acknowledged state propagates to backup; alarm remains in active list until cleared |
| Failover detection latency | Alarms during failover (typically 5–30 s) buffered in PLC alarm queue; backfilled to backup on reconnect |
| Audit-trail continuity | Both primary and backup write to common historian; failover not a gap in record |
| Test cadence | Failover drill quarterly; documented in §16.4 Handover. |

If single-HMI architecture: "Not applicable — single HMI server; failover via cold-spare with manual recovery."

---

## 9. Alarm Flood Management

### 9.1 Alarm Flood Definition

Per **ISA-18.2 §9**:

> *"An alarm rate that prevents the operator from effectively managing the alarms."*

**Flood threshold (per Round 3 follow-up):** > {flood_threshold} alarms per {flood_window} period (per operator workstation). HIS default: 10 alarms per 10-minute period.

When threshold exceeded, HMI displays "ALARM FLOOD — {N} alarms in last 10 min" on alarm banner.

### 9.2 Alarm Flood Response Procedure

1. **Operator:** Acknowledge Priority 1 + 2 alarms immediately. Do NOT attempt to acknowledge all; focus on highest priority.
2. **Operator:** Identify root cause via first-out (§9.3) if configured; review first alarm in flood.
3. **Operator:** Implement ARG for root-cause alarm (§15).
4. **Supervisor:** If flood continues > {flood_supervisor_minutes} minutes (HIS default: 5 min), activate plant emergency response.
5. **Post-stabilization:** HIS engineer reviews flood log; classifies cause: (a) single initiating event with cascading alarms, (b) chattering instrument, (c) configuration error.

### 9.3 First-Out Annunciation (Conditional — first-out selected in Round 3 Q2)

Identifies the first alarm in a cascade, distinguishing from subsequent alarms triggered by same root cause.

**Implementation method:**
- PLC captures timestamp of first alarm in each alarm group.
- HMI displays first-out alarm with "FIRST OUT" indicator in description.
- Subsequent alarms in same group displayed with "CASCADE" indicator.
- After root cause cleared and all alarms cleared, first-out latch resets.

**PLC implementation pattern (Rockwell example):**

```
First-Out Logic:
  IF (any alarm in group active) AND (FirstOut_Latched = FALSE) THEN
    FirstOut_AlarmTag = ThisAlarmTag
    FirstOut_Timestamp = Now()
    FirstOut_Latched = TRUE
  IF (no alarm in group active) THEN
    FirstOut_Latched = FALSE
    FirstOut_AlarmTag = (empty)
```

If first-out not in scope: "Not applicable — no first-out annunciation in this system."

### 9.4 Cause-of-Alarm Analysis

For each significant flood or recurring pattern:

1. Export alarm history for incident time window
2. Identify first alarm (first-out or earliest timestamp)
3. Identify alarms within {cascade_window_minutes} min of first (HIS default: 5 min) — likely cascade effects
4. Determine which can be suppressed, masked, or rationalized out
5. Document findings + register changes in MOC record

### 9.5 Power-Restoration Alarm Shower

When control power restored after full outage, hundreds of alarms fire simultaneously. Per ISA-18.2 §9 contemplation:

| Phase | Duration | Behavior |
|---|---|---|
| Phase 1 — Initial | First 60 s after power-up | Suppress all non-Priority-1 alarms via global power-up-suppress flag set by power-up sequence |
| Phase 2 — Triage | Next 5 min | Operator reviews active Priority 1 alarms and resolves; secondary alarms remain suppressed |
| Phase 3 — Restoration | After 5 min stable | Global suppress lifts; remaining alarms annunciate normally |
| Phase 4 — Post-Analysis | Within 24 h | HIS engineer reviews shower; bad-actor analysis includes shower alarms separately from steady-state KPIs |

This procedure is implemented in PLC power-up sequence; HMI displays "POWER-RESTORATION SHOWER ACTIVE — Phase {N}" indicator during Phases 1–3.

---

## 10. Alarm Performance KPIs

### 10.1 KPI Definitions

Per **ANSI/ISA-18.2-2016 §12** + **EEMUA 191:2013 (Edition 3)**:

| KPI | Definition | Target | Acceptable | Action Threshold | Review |
|---|---|---|---|---|---|
| Average Alarm Rate | Alarms per operator per 10 min steady-state | ≤ 1 | ≤ 6 | > 6 → 30-day remediation | Monthly (EEMUA 191) |
| Peak Alarm Rate | Max alarms per 10 min during incident | ≤ 10 | ≤ 20 | > 20 → root cause analysis required | Monthly per incident (ISA-18.2) |
| Operator Action Rate | Operator interventions per shift | Within trained capacity | Per customer policy | Excess → fatigue review | Monthly |
| Alarm Acknowledgment Time | Median time-to-ack from activation | < response-time / 2 | < response-time | Exceeded → operator training review | Monthly |
| Standing Alarms | Alarms continuously active > 24 h | ≤ 5 at any time | ≤ 10 | > 10 → bad-actor analysis (§13.5) | Weekly (EEMUA 191) |
| Chattering Alarms | Alarms with > {chatter_threshold} transitions/24 h | 0 | ≤ 3 (resolve 30 d) | > 3 → bad-actor (§13.5) | Monthly (ISA-18.2) |
| Stale Alarms | Alarms unacknowledged > 24 h | 0 | ≤ 2 | > 2 → operator training | Daily (early ops) |
| Suppressed Alarms | Currently suppressed/shelved/inhibited | ≤ 5% of total | ≤ 10% | > 10% → register review | Monthly (ISA-18.2) |
| Alarm Rate During Floods | % time in flood state | < 1% | < 5% | > 5% → flood-cause review | Monthly (EEMUA 191) |
| Repeat Alarms | Same alarm > N activations / 24 h (longer window than chatter) | ≤ 5 (per alarm) | ≤ 10 | > 10 → bad-actor (§13.5) | Monthly |
| % Rationalized | % of configured alarms with completed register entry | 100% before FAT | — | < 100% before FAT → FAT hold | Pre-FAT and post-MOC |
| Bad-Actor Count | Top-10 bad actors per §13.5 | 0 unresolved > 30 d | ≤ 3 unresolved > 30 d | > 3 → escalation to operations management | Monthly (§13.5) |

### 10.2 KPI Review Process

- Monthly KPI report generated by {HIS / Customer controls engineer}; reviewed by operations supervisor.
- KPI outside Acceptable range → remediation action within 30 days.
- KPI past Action Threshold → escalation per §13.5.
- Remediation actions tracked in §17 Open Items.

### 10.3 Monthly KPI Report Template

```markdown
# Alarm System KPI Report
## Reporting Period: {YYYY-MM} | Issued: {date} | Reviewed by: {operations supervisor}

## KPI Summary

| KPI | Target | Acceptable | Actual | Status |
|---|---|---|---|---|
| Average Alarm Rate | ≤ 1/10 min | ≤ 6/10 min | {value} | 🟢 / 🟡 / 🔴 |
| Peak Alarm Rate (worst incident) | ≤ 10/10 min | ≤ 20/10 min | {value} | 🟢 / 🟡 / 🔴 |
| Standing Alarms (current) | ≤ 5 | ≤ 10 | {value} | |
| Chattering Alarms | 0 | ≤ 3 | {value} | |
| Stale Alarms | 0 | ≤ 2 | {value} | |
| % Suppressed | ≤ 5% | ≤ 10% | {value} | |
| % Time in Flood | < 1% | < 5% | {value} | |
| Bad-Actor Count Open > 30 d | 0 | ≤ 3 | {value} | |

## Top-10 Bad Actors (this period)

| Rank | Tag | Type | Count | Cum Duration | Status |
|---|---|---|---|---|---|
| 1 | {tag} | Chattering / Frequent / Standing / Shelved | {N} | {hours} | Investigating / Remediating / Closed |

## Top-10 Standing Alarms

| Rank | Tag | Active Since | Cause | Owner |
|---|---|---|---|---|
| 1 | {tag} | {date} | {cause} | {name} |

## Open MOC Items Related to Alarms

| MOC # | Description | Owner | Target Close |
|---|---|---|---|

## Remediation Actions Closed Since Last Report

| Action | Alarm(s) Affected | Date Closed | Effect |
|---|---|---|---|

## Remediation Actions Due in Next 30 Days

| Action | Alarm(s) Affected | Owner | Target Date |
|---|---|---|---|

## Reviewer Sign-Off

- HIS Automation Engineer: __________________ Date: ________
- Customer Operations Supervisor: __________________ Date: ________
- Customer Controls Engineering: __________________ Date: ________
```

---

## 11. Safety Instrumented System Alarms

{Include if SIS / SIF selected in Round 3 Q4. Otherwise: "Not applicable — no SIS integration on this project."}

### 11.1 SIS Alarm Policy

Safety Instrumented Function (SIF) trips are generated by the Safety PLC ({GuardLogix catalog from /fds §11.2}). Distinct from BPCS alarms; subject to:

- **SIS trips are Priority 1 by default**; cannot be downgraded.
- **SIS trips are not suppressed or shelved** under any circumstance. Any suppression requires formal SIS bypass procedure with physical key switch + written work permit (§11.6 register).
- **SIS trips are mirrored to BPCS HMI** via {produced/consumed tags / OPC-UA / safety network}. BPCS displays state but does not control or override.
- **SIS trip alarms in BPCS** display with "SIS TRIP" prefix to distinguish from BPCS alarms.
- **SIS trip acknowledgment in BPCS** does not reset SIS or SIS trip condition. SIS reset independently per SIS reset procedure.

### 11.2 SIS Alarm Integration Architecture

Cross-reference /fds §11.1 SIF table.

| SIS Function | SIS Tag | BPCS Mirror Tag | BPCS Alarm Priority | Required Response | SRS Ref | Proof Test Interval |
|---|---|---|---|---|---|---|
| {SIF-01 description} | {SZ1_Trip01} | {SIF01_TripStatus} | Priority 1 | {Immediate shutdown procedure ref} | {SRS doc ref} | {months} |
| {add SIF rows} | | | | | | |

### 11.3 SIS Pre-Alarm Architecture

A **SIS pre-alarm** at typically 90% of trip threshold gives operator time to act before SIS trips.

| Field | Rule |
|---|---|
| Pre-alarm priority | Priority 1 (same as trip) |
| Pre-alarm setpoint | Typically 90% of SIS trip setpoint (specify per project) |
| Suppression / shelving rules | Pre-alarms MAY be shelved with supervisor approval if maintenance is imminent (SIS itself remains active, so shelving the BPCS pre-alarm is OK). Trip alarms — never. |
| HMI distinction | Pre-alarm shows "PRE-TRIP" suffix in description; trip shows "SIS TRIP" suffix |
| Operator action | Identify root cause; act before SIS trip threshold reached |

| SIS Function | Pre-Alarm Tag | Pre-Alarm Setpoint | Trip Setpoint | % of Trip |
|---|---|---|---|---|
| {SIF-01} | {SIF01_PreTrip} | {value} | {value} | 90% |

### 11.4 SIS Diagnostic Alarms

Distinct from SIS trip alarms. Generated by SIS internal diagnostics.

| Diagnostic | Source | BPCS Priority | Operator Action |
|---|---|---|---|
| Degraded I/O channel | SIS I/O module diagnostics | Priority 2 (High) | Notify maintenance; SIS still operational on remaining channels |
| Partial proof-test failure | SIS proof-test routine | Priority 2 (High) | Schedule full proof test; investigate root cause |
| Internal watchdog timeout | SIS controller diagnostics | Priority 1 (Critical) | SIS may fail to safe state; engage Functional Safety Manager |
| Communication degradation (SIS ↔ BPCS) | SIS comms module | Priority 3 (Medium) | Investigate network; SIS itself unaffected |
| {add per project} | | | |

### 11.5 Proof-Test-Overdue Alarm

When SIS proof-test interval is exceeded by > 30 days:

| Field | Rule |
|---|---|
| Alarm priority | Priority 2 (High) |
| Trigger | (Current date − Last proof-test date) > (Proof-test interval + 30 days) |
| Response | Schedule proof test ASAP; notify Functional Safety Manager |
| Reference | IEC 61511-1:2016 §16 |
| Compliance impact | Continued operation past proof-test interval is a regulatory violation in most regulated industries |

### 11.6 SIS Bypass Register

Per **IEC 61511-1:2016 §11.5**:

| Bypass ID | SIF | Authorization | Maximum Duration | Tracking |
|---|---|---|---|---|
| Maintenance bypass (key switch) | Per SIF | Maintenance supervisor + Functional Safety Manager | Single shift | Maintenance log + Bypass Register |
| Engineering bypass | Per SIF | Functional Safety Manager + supervisor | 8 hours | Bypass Register |
| Permanent change | Per SIF | MOC required (§16) + customer approval | n/a | Change Control + this document revision |

Register columns: Bypass ID / SIF / Initiator / Reason / Start time / End time / Closed-by / Risk-mitigation in place during bypass.

---

## 12. 21 CFR Part 11 Alarm Requirements

{Include if 21 CFR Part 11 selected in Round 3 Q4. Otherwise: "Not applicable."}

### 12.1 Electronic Records Requirements

Per **21 CFR Part 11**, alarm acknowledgment records and alarm setpoint changes constitute electronic records.

- Every alarm acknowledgment records: user identity (unique username, not shared), timestamp (synchronized to facility time server), alarm tag, alarm description, priority, acknowledgment reason code.
- Shared operator accounts ("operator1") are **prohibited**.
- Alarm history records read-only after creation. Deletion: supervisor authorization, documented reason, deletion event itself logged.
- Electronic alarm history backups are part of the validated system backup procedure.
- Setpoint changes to alarms are electronic records (§12.5).

### 12.2 Alarm Acknowledgment Reason Codes

Operators select reason code when acknowledging Priority 1 + 2 alarms:

| Code | Reason |
|---|---|
| ACK-01 | Condition understood; corrective action taken |
| ACK-02 | Condition under investigation |
| ACK-03 | Condition expected — process in abnormal state; monitoring |
| ACK-04 | Instrument believed faulty — maintenance notified |
| ACK-05 | Other (requires free-text entry) |

### 12.3 Open vs Closed System Declaration

| Determination | Rule |
|---|---|
| **Closed System** per § 11.10 | System access controlled by persons responsible for record content; records cannot be modified outside controlled environment |
| **Open System** per § 11.30 | System access not controlled by record-content owners; digital signatures required |
| **This project** | {Closed / Open — select per customer's IT architecture} |

If Closed: § 11.10 controls apply (audit trail, validation, system documentation, training).
If Open: § 11.30 controls additionally — digital signatures with cryptographic strength sufficient to deter forgery.

### 12.4 Audit-Trail Tamper Evidence

Audit trail must be cryptographically tamper-evident. Method declared per project:

| Method | Description | Suitable For |
|---|---|---|
| Digital signature per record | Each record signed with PKI key; verifiable independently | Open system; high-stakes trial data |
| Hash-chain | Each record includes hash of previous record; tampering detectable | Closed system; mid-stakes production records |
| Write-once storage (WORM) | Records written to immutable storage; physically tamper-evident | Closed system; archival batch records |

**This project uses:** {select method based on customer IT architecture}.

### 12.5 Alarm Setpoint Change Records

Every setpoint change creates an electronic record. Recorded fields:
- User who made the change (unique ID)
- Timestamp (NTP-synchronized)
- Alarm tag affected
- Old setpoint value
- New setpoint value
- Reason code (linked to MOC if applicable; otherwise documented justification)
- MOC reference number (if §16 MOC was raised)
- Approval signature (if change requires it per role-based access)

### 12.6 Backup Validation

Per **21 CFR § 11.10(c)**: backup procedures shall be validated and tested annually. Validation evidence retained.

| Backup Element | Frequency | Validation | Retention |
|---|---|---|---|
| Alarm history database | Daily | Annual restore-test | 7 years (or per customer Quality Manual) |
| Alarm configuration export | After each MOC | Pre-FAT validation | Permanent (versioned) |
| Operator user database | Weekly | Annual restore-test | 7 years |
| Audit-trail records | Daily | Quarterly read-back | 7 years (or per customer) |

---

## 13. Alarm Hierarchy by Area + Bad-Actor Management

### 13.1 Per-Area Alarm Hierarchy

{Populate from Nexus or FDS / Round 2 interview. One row per area / ISA-88 unit.}

| Area | ISA-88 Unit | Approx Alarm Count | Priority 1 Count | Priority 2 Count | Suppression Groups |
|---|---|---|---|---|---|
| {Area 100 — Feed Prep} | {Feed Unit} | {TBD} | {TBD} | {TBD} | Startup, Shutdown |
| {Area 200 — Reactor} | {Reactor Unit} | {TBD} | {TBD} | {TBD} | Startup, Shutdown, CIP |
| {add areas} | | | | | |
| **TOTAL** | | **{TBD}** | **{TBD}** | **{TBD}** | |

### 13.2 Per-Area Standing-Alarm Watch

Standing alarms (continuously active > 24 h) per area:

| Area | Active Standing Alarms (current) | Threshold for Action | Owner |
|---|---|---|---|
| Area 100 | {N} | ≤ 5 | {name} |

### 13.3 Per-Area Suppression Audit

| Area | Currently Suppressed Alarms | % of Area Total | Trend (vs last month) |
|---|---|---|---|

### 13.4 Per-Area Operator Coverage

For multi-shift / multi-station plants:

| Area | Day-Shift Operator | Night-Shift Operator | Lead | Backup |
|---|---|---|---|---|

### 13.5 Bad-Actor Management Process

Per **ANSI/ISA-18.2-2016 §13**:

#### 13.5.1 Bad-Actor Definitions

| Bad-Actor Type | Criterion |
|---|---|
| Frequent | Top-10 by activation count (12-month window) |
| Standing | Top-10 by cumulative active duration (12-month window) |
| Chattering | Top-10 by transition count per 24 h |
| Frequently Shelved | Top-10 by shelving frequency |

#### 13.5.2 Monthly Bad-Actor Analysis

1. Generate ranked top-10 list per type from alarm history.
2. For each bad actor: investigate root cause (process drift / instrument drift / setpoint inappropriate / nuisance / over-conservative).
3. Propose remediation: rationalization adjustment / setpoint tuning / instrument calibration / hardware repair / alarm deletion.
4. Each remediation tracked in Bad-Actor Remediation Register (§13.5.3).

#### 13.5.3 Bad-Actor Remediation Register

| ID | Alarm Tag | Bad-Actor Type | Root Cause | Proposed Remediation | MOC Ref | Target Close | Status |
|---|---|---|---|---|---|---|---|
| BA-001 | {tag} | Chattering | Sensor drift due to vibration | Recalibrate; tighten on-delay 5 s → 15 s | MOC-2025-014 | 2025-MM-DD | Open / In Progress / Closed |

#### 13.5.4 Escalation

- Bad actor open > 30 days → escalation to operations management.
- Bad actor open > 60 days → escalation to plant manager + recorded as KPI "Open > 60 d" in §10.

---

## 14. Master Alarm Rationalization Register

The complete list of rationalized alarms. **Every alarm configured in HMI or PLC must have a corresponding row with "Rationalized By" and "Date" populated before configuration.**

{Pre-populate from Nexus if available — flag rows as `[ASSUMED from Nexus — verify with customer]`. Otherwise, provide template rows for the rationalization team to complete.}

| # | Alarm Tag | Description (≤ 40 chars) | Area / Unit | Priority | Cause | Consequence | Operator Response | Setpoint | Deadband | On-Delay (s) | Off-Delay (s) | Suppression Logic | Cascade From | Equipment-Fault From | ARG Ref | Source / Origin | HMI Display Path | PLC Routine | RTM Req ID | First Activated | Last Activated | Activation Count | Rationalized By | Date |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | {Tag}_Alm | {Description} | {Area} | {1/2/3/4} | {Cause} | {Consequence} | {Action} | {Value} | {%} | {0} | {0} | {None / Startup / etc.} | {None / AL-XXX} | {None / M-101} | {ARG-N} | {PLC interlock / etc.} | {Screen IDs} | {Routine + rung} | {FDS-REQ-XXX} | {date or empty} | {date or empty} | {N or 0} | {Name, Role} | {Date} |
| {add all alarms} | | | | | | | | | | | | | | | | | | | | | | | | |

**Register statistics:**
- Total alarms: {N}
- Rationalized: {N}
- Pending rationalization: {N}
- Priority 1: {N} ({%})
- Priority 2: {N} ({%})
- Priority 3: {N} ({%})
- Priority 4: {N} ({%})

---

## 15. Operator Alarm Response Guides (ARGs)

{Include per Round 3 Q3 ARG format. For "embedded" format, the Operator Response column in §14 is the ARG. For "separate ARG document" or "both", include the full ARG template below for each Priority 1 + Priority 2 alarm.}

{If not required: "Not applicable — operator response embedded in Master Register §14."}

### ARG-{N}: {Alarm Tag} — {Alarm Description}

| Field | Value |
|---|---|
| Alarm Tag | {Tag}_Alm |
| Alarm Text (HMI) | {Description ≤ 40 chars} |
| Priority | {1 or 2} |
| Area | {Area name} |
| Setpoint | {Value + Units} |
| Last Reviewed | {date} |
| Next Review Due | {date + 1 year} |
| Approver | {name + role} |

**Probable Causes (ranked most-likely → least-likely):**

1. {Most likely} — {how to verify; reference §13.5 Measurement Points if /soo exists}
2. {Second most likely} — {verification}
3. {Less likely} — {verification}

**Visual Aid:** {HMI screenshot of affected faceplate with arrow indicating alarm element. Cross-reference /hmi-spec faceplate spec.}

**Trend Reference:** {Historian tag(s) and time-window operator should examine to confirm cause. Example: "Trend PIT-101 for last 30 minutes; look for step changes or drift."}

**Decision Tree (ASCII):**
```
                    Alarm AL-XXX active
                            │
              ┌─────────────┴─────────────┐
              │                           │
       Most-likely cause             Less-likely cause
              │                           │
       Verify via {measurement}    Verify via {measurement}
              │                           │
       Action: {step}              Action: {step}
```

**Immediate Action (< {response time} minutes):**

1. {Step 1 — specific and actionable}
2. {Step 2}
3. {Step 3}

**Follow-Up Actions:**

- {What to do after immediate response to prevent recurrence}
- {Notify: operations supervisor / maintenance / engineering}
- {Document: shift log / work order / batch record}

**Operator Training Module Reference:** {Link to formal training module (LMS course ID) covering this alarm.}

**Contacts:**

| Contact | Role | When to Call |
|---|---|---|
| {Name} | Operations Supervisor | Priority 1 alarms; escalation after initial response |
| {Name} | Maintenance | Instrument or equipment fault confirmed |
| {HIS contact} | HIS Automation Engineer | Control system fault; alarm configuration issue |

---

## 16. Management of Change

### 16.1 When an MOC Is Required

MOC required for ANY of the following after initial commissioning sign-off:
- Adding new alarm
- Deleting alarm
- Changing priority
- Changing setpoint, deadband, or delay > ±10%
- Changing description
- Adding/modifying/removing suppression logic
- Adding/modifying cascade or equipment-fault suppression
- Changing ARG content
- Adding/removing alarm from HMI display

Changes that do NOT require MOC (but are logged):
- Operator shelving (§7.3 audit trail)
- Engineering inhibit for active maintenance (§7.4 audit trail)

### 16.2 Standard MOC Process

1. **Request:** HIS or customer engineer submits change request with: alarm tag, current config, proposed config, justification.
2. **Review:** Customer process engineer + lead operator review rationalization impact. If priority or setpoint > 10% changes, mini-rationalization session.
3. **Approval:** Customer controls engineering supervisor approves. Priority 1 additions or deletions require operations manager approval.
4. **Implementation:** HIS engineer implements in HMI / PLC.
5. **Testing:** Force the condition; verify alarm at correct setpoint, priority, annunciation.
6. **Documentation:** §14 register updated. MOC record filed. This document revision assessed — if change represents philosophy shift, this document revised.
7. **Trigger §17 Open Items update.** Trigger §13.5 Bad-Actor remediation register entry if applicable.

### 16.3 Emergency MOC Process

For changes during production incident or commissioning emergency:

1. Verbal approval from Customer operations supervisor.
2. Immediate implementation by HIS engineer.
3. Post-implementation: full MOC record completed within {moc_emergency_turnaround} (HIS default: 24 h or next business day).
4. Emergency MOCs flagged in MOC log for review at next monthly KPI meeting.

### 16.4 Alarm-System Handover Checklist

When HIS hands over alarm system to Customer operations:

| # | Item | Signed (HIS) | Signed (Customer) |
|---|---|---|---|
| 1 | §14 Master Register baseline frozen at FAT | | |
| 2 | HMI alarm configuration backed up; backup location documented | | |
| 3 | Bad-actor list (initial state) captured per §13.5 | | |
| 4 | Standing-alarms list (initial state) captured | | |
| 5 | KPI baseline measurement complete (first 30 days post-commissioning) | | |
| 6 | Operator training records linked to ARGs | | |
| 7 | First MOC review meeting scheduled | | |
| 8 | Customer's Alarm Manager (named individual) acknowledged responsibility | | |
| 9 | Failover drill (§8.7) completed and documented | | |
| 10 | (If 21 CFR Part 11) Backup validation per §12.6 complete | | |
| 11 | (If SIS) Proof-test schedule per §11.5 reviewed | | |

---

## 17. Open Items and TBDs

| # | Section | Description | Owner | Target Date | Status |
|---|---|---|---|---|---|
| TBD-001 | §{N} | {description} | {Customer / HIS} | {date} | Open |

Every `[ASSUMED from Nexus — verify with customer]` flag and every remaining `[TBD]` placeholder appears as a row here.

---

## 18. Glossary

| Term | Definition |
|---|---|
| ALCOA+ | Attributable, Legible, Contemporaneous, Original, Accurate + Complete, Consistent, Enduring, Available |
| Alarm | Per ISA-18.2: audible/visible indication of equipment malfunction, process deviation, or abnormal condition requiring operator action |
| ARG | Alarm Response Guide — step-by-step operator procedure |
| Bad Actor | Alarm appearing frequently, chattering, or routinely shelved — rationalization deficiency |
| Bad Quality | Alarm tier for AI signal at range limits or PLC tag quality bad |
| Cascade Suppression | Programmatic suppression of downstream alarms when upstream trip fires |
| Chattering | Alarm activating > {chatter_threshold} times in 24 h |
| Closed System | 21 CFR § 11.10 — access controlled by record-content owners |
| Deadband | Hysteresis on analog setpoint; prevents chattering on recovery |
| EEMUA 191 | Engineering Equipment and Materials Users' Association Publication 191 — alarm-system performance guide |
| Equipment-Fault Suppression | Programmatic suppression of downstream alarms while equipment in Faulted state |
| First-Out | Annunciation method identifying first alarm in cascade |
| Inhibit | Engineering-level alarm block requiring elevated access |
| ISA-18.2 | ANSI/ISA-18.2-2016 (R2023) — primary governing standard |
| KPI | Key Performance Indicator |
| MOC | Management of Change |
| Nuisance Alarm | Alarm activating frequently without useful operator action — rationalization target |
| On-Delay / Off-Delay | Time before alarm activates / deactivates after condition |
| Open System | 21 CFR § 11.30 — digital signatures additionally required |
| Out-of-Service (OOS) | Alarm formally deactivated; equipment decommissioned |
| Pre-Alarm | SIS pre-alarm at typically 90% of trip threshold |
| Priority | Ranking of urgency × consequence (1 = highest) |
| Proof Test | Periodic verification of SIS function per IEC 61511 §16 |
| Rationalization | Documenting cause, consequence, response, setpoint per alarm |
| Rationalization Register | Master table — governing document for alarm configuration |
| RTM | Requirements Traceability Matrix (in parent FDS §1.4) |
| Shelve | Operator-applied temporary alarm suppression |
| SIF / SIS / SIL | Safety Instrumented Function / System / Integrity Level |
| Standing Alarm | Active continuously > 24 h |
| Suppress | Programmatic alarm block based on process state, cascade, or equipment fault |
| Tamper-Evidence | Cryptographic / hash-chain / WORM mechanism preventing audit-trail modification |
| {Customer Acronym} | {Definition} |

---

## 19. Industry-Specific Alarm Management Overlay

{Emit one or more sub-sections matching Round 2 industry. If industry = Other or no overlay applies, write: "Not applicable — no regulated-industry overlay required for this project."}

### 19.1 Pharma / Biotech (FDA / EU GMP)

This project is subject to **GAMP 5 (2nd Edition, 2022)**, **21 CFR Part 11**, and **EU Annex 11**.

| Aspect | Requirement | This Document |
|---|---|---|
| Software Categorization (GAMP 5) | Cat 3 / 4 / 5 per project | {populated from conditional follow-up} |
| Audit Trail | 21 CFR § 11.10(e) + EU Annex 11 § 9 — secure, time-stamped, computer-generated | §12 + §8.4 |
| Electronic Signatures | 21 CFR § 11.50–§ 11.300 | §12.1 + §12.4 |
| Open vs Closed System | 21 CFR § 11.10 vs § 11.30 | §12.3 |
| Tamper-Evidence | § 11.10(c) | §12.4 |
| Setpoint-Change Records | § 11.10(c) | §12.5 |
| Backup Validation | § 11.10(c) | §12.6 |
| ALCOA+ Data Integrity | FDA / EMA practice | §8.4 + operator records |
| EU Annex 11 § 4 (Validation) | Computer system validation | Cross-reference QMS validation plan |
| EU Annex 11 § 11 (Periodic Evaluation) | Annual review | §10.2 + §16 |
| EU Annex 11 § 15 (Change Management) | Controlled MOC | §16 |
| {NAMUR NA 102 — EU export only} | Alarm management | Cross-reference customer's NA 102 mapping |

### 19.2 Water / Wastewater (AWIA / AWWA)

If population served ≥ 3,300 (per Round 3 follow-up):
- **AWIA §2013 ERP coordination**: alarm conditions that trigger ERP activation listed in §15 ARG escalation.
- **AWWA Manual M2** alarm guidance applied to SCADA architecture.
- Risk & Resilience Assessment (RRA) cross-reference: alarm-system contributes to RRA's automation-systems inventory.

| Alarm Class | ERP Activation Trigger |
|---|---|
| Source-water contamination indicators | Yes — within 1 hour |
| Treatment-process bypass | Yes — immediate |
| Distribution low-pressure | Yes — within 4 hours |
| Disinfection residual low | Yes — within 1 hour |

### 19.3 Oil & Gas / Petrochemical (OSHA PSM / API)

OSHA 29 CFR 1910.119 § (f) operating procedures coverage — this Alarm Philosophy is part of the PSM Process Safety Information (PSI) per § (d).

If pipeline operation: **API RP 1167 (2nd Edition, 2016)** Pipeline SCADA Alarm Management applies. Add API 1167 to §1.3 References.

API 1167 key requirements:
- Periodic alarm-management reports to PHMSA
- Operator-action review for missed alarms
- Pipeline-specific bad-actor analysis

### 19.4 Chemical Processing (OSHA PSM / CCPS)

**CCPS Guidelines for Safe and Reliable Instrumented Protective Systems** (2007) applies. SIS-BPCS alarm integration per §11 of this document aligns with CCPS practice.

OSHA PSM § (f) coverage: this document contributes to PSM operating procedures package.

### 19.5 Power Generation (NERC CIP)

If BES Cyber System (per Round 3 follow-up CIP-002 categorization):

| CIP Standard | Topic | Alarm Connection |
|---|---|---|
| CIP-002 | BCS categorization | Determines applicability |
| CIP-005 | Electronic Security Perimeter | Alarms on ESP-boundary anomalies |
| CIP-007 | System Security Management | Alarms on unauthorized access attempts to HMI |
| CIP-008 | Incident Response | Alarm-driven incident reporting timeline |
| CIP-010 | Configuration Change Management | Alarm-config changes are tracked changes |

CIP-008 incident-response triggers from alarms:
- Unauthorized access attempt to HMI → CIP-008 reportable within 1 hour
- Anomalous setpoint change outside §6.5 authorization bounds → CIP-008 reportable within 1 hour
- Comms-loss patterns suggesting active intrusion → CIP-008 reportable within 1 hour

### 19.6 Discrete Manufacturing (NFPA 79 / ISO 13849-1)

NFPA 79-2024 alarm-light requirements for machine operation:

| Light Color | Meaning | NFPA 79 Reference |
|---|---|---|
| Red | Emergency / Fault | §10.4.2 |
| Yellow / Amber | Caution / Approaching limit | §10.4.2 |
| Green | Safety / Normal operation | §10.4.2 |
| White / Clear | Status / Information | §10.4.2 |

ISO 13849-1:2023 safety-stop alarm integration: safety-stop functions cross-referenced to alarms in §11.

### 19.7 Other Industry — Customer-Specified

{If Round 2 industry = Other and customer specified standards in Round 3: emit a subsection citing each customer standard, its rev, and the section it constrains.}

---

## 20. Revision History

(Sole authoritative revision log.)

| Rev | Date | Author | Description |
|---|---|---|---|
| {rev} | {date} | T. Hunt | Initial draft for customer review |
````

---

### Step 4.5 — Mandatory Quality Check Before Saving

Run all rules; report results in the summary block.

1. **§4.1 emits exactly one priority scheme** matching Round 3 Q1; no leakage from unused schemes.
2. **§4.3 priority distribution targets** present and consistent with chosen scheme.
3. **Every priority assigned in §14 register** matches §4.1 scheme (no Priority 5 in a 4-level system).
4. **Every alarm in §14** has all mandatory rationalization fields populated or `[TBD]`.
5. **Every `[ASSUMED from Nexus — verify with customer]` and every `[TBD]` flag** appears as a row in §17 Open Items.
6. **Every Round 2 area** appears in §13.1 Alarm Hierarchy.
7. **Every KPI in §10** cites year/edition source.
8. **Standards in §1.3** cite year/edition.
9. **§19 Industry overlay** emits exactly the sub-section matching Round 2 industry (or "Not applicable" with rationale).
10. **If SIS = Yes:** §11 is present with §11.3 SIS Pre-Alarm + §11.4 Diagnostic + §11.5 Proof-Test-Overdue + §11.6 Bypass Register populated.
11. **If 21 CFR Part 11 = Yes:** §12 is present; §12.3 Open/Closed declared; §12.4 tamper-evidence method declared; §12.5 setpoint-change records described; §12.6 backup validation described.
12. **Every Priority 1 alarm has an ARG** (per Round 3 Q3 ARG format) or non-empty Operator Response in §14.
13. **No `{X}` / `{N}` placeholders remain** — all interview-driven thresholds (chatter / flood / shelve / cascade / backup / MOC turnaround) populated.
14. **§1.4 RTM linkage** populated or marked Not Applicable with rationale.
15. **§13.5 Bad-Actor Management process** present.
16. **§5.5 Workshop Agenda** present.
17. **§7.5 Cascade Suppression and §7.6 Equipment-Fault Suppression** present (or marked Not Applicable per Round 3 Q2).
18. **§8.5–§8.7** Multi-Station / Mobile / HMI Failover present (or marked Not Applicable per platform).
19. **§9.5 Power-Restoration Alarm Shower** procedure present.
20. **§10.3 KPI Report template** present.
21. **§16.4 Handover Checklist** present.
22. **§6.5 Bad Quality alarm tier** present.
23. **Glossary completeness** — every capitalized abbreviation appearing in body has §18 row.
24. **Platform-conditional consistency** — §3.2 emits only the block matching Round 2 platform; no leakage.
25. **ToC** present and accurate.
26. **Cross-skill ingestion** — if FDS / SOO / IO List / HMI Spec / BOM exist in folder, each cited in §1.3.

Report:

```
Alarm Philosophy Quality Check
─────────────────────────────────
Priority schemes leaked:                      0
Priority distribution targets:                OK
Alarm priorities consistent with scheme:      OK
Mandatory register fields populated:          ###/###
Nexus-ASSUMED rows in body:                   ### (must each be in §17)
TBD placeholders remaining:                   ###
Round-2 areas in §13.1:                       OK / list gaps
KPIs missing year/edition:                    0
Standards missing year/edition:               0
§19 Industry overlay:                         {emitted / Not applicable}
§11 SIS sections (if SIS=Yes):                OK / list gaps
§12 Part 11 sections (if 11=Yes):             OK / list gaps
P1 alarms with ARG:                           ###/###
{X} / {N} placeholders unresolved:            0
§1.4 RTM linkage:                             OK / Not applicable
§13.5 Bad-Actor process:                      OK
§5.5 Workshop Agenda:                         OK
§7.5 / §7.6 suppression rules:                OK / list gaps
§8.5–§8.7 (multi-station / mobile / failover): OK / Not applicable
§9.5 Power-Restoration Shower:                OK
§10.3 KPI Report template:                    OK
§16.4 Handover Checklist:                     OK
§6.5 Bad Quality tier:                        OK
Glossary terms missing:                       0
Platform leakage in §3.2:                     0
ToC accurate:                                 OK
Cross-skill ingestions cited:                 ###/{available}
```

---

### Step 5 — Finalize and Save the Document

If outputs from `/io-list`, `/bom`, `/fds`, `/soo`, or `/hmi-spec` exist in the same save folder, parse them and use:

- **`FDS-*-Rev*.md`** — parse §1.4 RTM and §11 SIF table; populate this doc §1.4 + §11.2.
- **`SOO-*-Rev*.md`** — parse §6.4 Cause & Effect Matrix and §11.1 Fault Response; cross-reference into §14 register.
- **`IO-LIST-*.csv`** — every analog AI/AO row is alarm candidate; flag rationalization team to consider hi/hi-hi/lo/lo-lo per signal.
- **`HMS-*-Rev*.md`** — verify §8 banner / Alarm Summary structure matches HMI Spec.
- **`BOM-*.csv`** — confirm safety relays / GuardLogix module appear (sanity check for SIS section).

Note cross-reference findings in §1.3 References table and at top of saved file.

1. **Always ask where to save — never silently default to Desktop.** Use `AskUserQuestion`:

   ```
   AskUserQuestion([
     {
       question: "Where should the Alarm Philosophy be saved?",
       header: "Save Location",
       multiSelect: false,
       options: [
         { label: "Active project folder — I'll type the path (Recommended)", description: "WSL path. Recommended so Alarm Philosophy lives next to FDS, SOO, BOM." },
         { label: "Same folder as FDS or I/O List for this project — I'll type the path", description: "WSL path." },
         { label: "Current working directory", description: "Use the current shell working directory." },
         { label: "Desktop", description: "/mnt/c/Users/HIS Controls/Desktop/ (Windows Desktop via WSL — use only for one-off / scratch documents)." }
       ]
     }
   ])
   ```

2. Save using a status-aware filename:
   - `Status = DRAFT (Rev = Draft)` → `ALARM-PHIL-{slug}-Draft.md`
   - `Status = DRAFT (Rev = A/B/C/...)` → `ALARM-PHIL-{slug}-Rev{rev}-DRAFT.md`
   - `Status = FOR REVIEW` → `ALARM-PHIL-{slug}-Rev{rev}-FOR-REVIEW.md`
   - `Status = APPROVED` → `ALARM-PHIL-{slug}-Rev{rev}-APPROVED.md`

3. Report: full WSL file path, Quality Check summary block (Step 4.5), total alarm count, rationalized vs pending count, TBD count, KPI target source citations, status, RTM linkage status.

---

### Step 6 — Present Next Steps Using AskUserQuestion

```
AskUserQuestion([
  {
    question: "The Alarm Philosophy draft is saved. What would you like to do next?",
    header: "Next Step",
    multiSelect: false,
    options: [
      { label: "Run a rationalization session — work through the register alarm by alarm (per §5.5 agenda)", description: "Targeted workshop walkthrough." },
      { label: "Review priority scheme — confirm the chosen scheme is appropriate", description: "Assess against alarm count and industry." },
      { label: "Review KPI targets — adjust EEMUA 191 benchmarks to customer's internal standards", description: "" },
      { label: "Generate ARGs — produce full §15 Operator Response Guides for all P1 / P2 alarms", description: "" },
      { label: "Resolve open items — work through §17 TBDs", description: "" },
      { label: "Run /fds — generate FDS with this Alarm Philosophy referenced in §9", description: "" },
      { label: "Run /soo — generate SOO with §11 Fault Response cross-referencing this register", description: "" },
      { label: "Run /hmi-spec — generate HMI Spec using this Alarm Philosophy as alarm-display basis", description: "" },
      { label: "Customer-send-ready — finalize at chosen Rev (FOR REVIEW)", description: "Switch status; rename file." },
      { label: "Done — this draft is ready for customer review", description: "" }
    ]
  }
])
```
