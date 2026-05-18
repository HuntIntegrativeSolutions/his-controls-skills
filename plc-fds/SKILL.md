---
name: plc-fds
description: Generate a Functional Design Specification (FDS) for an industrial automation project. Produces a contractually-defensible, customer-ready document with Requirements Traceability Matrix, Assumptions Register, conditional regulated-industry compliance overlays (GAMP 5 / 21 CFR Part 11 / NERC CIP / AWIA / OSHA PSM), SIL Verification columns, ISA-88 phase state model, and platform-conditional content for Rockwell / Siemens / Beckhoff / other. Follows ANSI/ISA-5.06.01-2007, ANSI/ISA-88.01-2010, ANSI/ISA-18.2-2016, ANSI/ISA-101.01-2015, IEC 61511-1:2016, IEC 62443 (parts 2-1 / 3-2 / 3-3), ISO 13849-1:2023, NFPA 79, NEC 110.26 / 409, EEMUA 191, NIST SP 800-82.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [fds, plc, industrial, isa-5.06, isa-88, isa-101, isa-18.2, iec-61511, his]
    ported_from: claude-code/fds
---
# /fds — Functional Design Specification Generator

**Invocation:** `/fds [project-name]`

**Example:** `/fds "Wastewater Lift Station No. 4 Control Upgrade"`

---

## Purpose

Generate a complete Functional Design Specification (FDS) — the contractual constitution of an industrial automation project. The FDS defines WHAT the system does before any code is written; every subsequent deliverable (I/O list, PLC program, HMI, FAT checklist, validation package) traces back to it. The skill produces a document that survives a customer's chief-controls-engineering review, an FDA / EU GMP audit, an OSHA PSM inspection, a NERC CIP audit, and a commissioning-dispute deposition.

Standards alignment (specific editions matter — reviewers check first):

- **ANSI/ISA-5.06.01-2007** — Functional Requirements Documentation for Control Software Applications (the master standard for this document type)
- **ANSI/ISA-88.01-2010** — Batch Control: Models and Terminology (procedural state machine)
- **ANSI/ISA-18.2-2016** — Management of Alarm Systems
- **ANSI/ISA-101.01-2015** — Human Machine Interfaces
- **IEC 61511-1:2016 (Edition 2)** — Functional Safety — Safety Instrumented Systems for the Process Industry
- **IEC 62443-2-1:2010** — Establishing an IACS Security Program
- **IEC 62443-3-2:2020** — Security Risk Assessment for System Design
- **IEC 62443-3-3:2013** — System Security Requirements and Security Levels
- **ISO 13849-1:2023 (Edition 4)** — Safety of Machinery — Safety-Related Parts of Control Systems
- **NFPA 79-2024** — Electrical Standard for Industrial Machinery (machine-control projects)
- **NEC 2023 (NFPA 70)** — Articles 110.26 (working clearance) and 409 (industrial control panels)
- **EEMUA 191:2013 (Edition 3)** — Alarm Systems: Design, Management and Procurement (paired with ISA-18.2)
- **NIST SP 800-82 Rev. 3** — Guide to Operational Technology (OT) Security (paired with IEC 62443)
- Conditional regulated-industry standards layered in by Section 18 based on the industry answer in Round 1.

---

## Instructions

### Step 1 — Conduct the Project Interview Using AskUserQuestion

**MANDATORY:** Every question to the user goes through `AskUserQuestion`. Never ask in plain text. This is a formal interview, not a chat. Never assume a value the user did not provide. Defaults are only acceptable when explicitly labeled `(Recommended)` inside an option, never silently in code.

Conduct the interview in **three sequential rounds plus targeted follow-ups**. Wait for answers from each round before proceeding to the next.

---

#### Interview Round 1 — Project Identity, Industry, Schedule (ask all 4 together)

```
AskUserQuestion([
  {
    question: "What is the project name and HIS project number?",
    header: "Project ID",
    multiSelect: false,
    options: [{ label: "I'll type it", description: "Use the Other input to capture project name and HIS project number." }]
  },
  {
    question: "Who is the customer and where is the site located?",
    header: "Customer",
    multiSelect: false,
    options: [{ label: "I'll type it", description: "Use the Other input for customer name + site location." }]
  },
  {
    question: "What type of project is this?",
    header: "Project Type",
    multiSelect: false,
    options: [
      { label: "Greenfield — new system, nothing there before", description: "Full FDS from scratch." },
      { label: "Brownfield Upgrade — existing system being modified or expanded", description: "Reuse some existing equipment/logic." },
      { label: "PLC Migration — replacing old PLC with modern hardware", description: "Tag mapping appendix likely required." },
      { label: "Process Skid — packaged skid with its own controls", description: "Vendor-supplied skid; FDS focuses on integration boundary." },
      { label: "Machine Control — discrete manufacturing machine", description: "NFPA 79 machine; ISO 13849-1 may apply." },
      { label: "SCADA / HMI Only — no PLC work, visualization layer only", description: "FDS scoped to HMI / SCADA; PLC sections marked Not Applicable." }
    ]
  },
  {
    question: "What industry / regulatory environment governs this project?",
    header: "Industry",
    multiSelect: false,
    options: [
      { label: "Pharmaceutical / Biotech (FDA / EU GMP)", description: "GAMP 5 categorization + 21 CFR Part 11 + EU Annex 11 + ALCOA+ overlay required." },
      { label: "Food & Beverage (FSMA / 3-A)", description: "FSMA preventive controls + 3-A sanitary standards." },
      { label: "Oil & Gas / Petrochemical (OSHA PSM / API)", description: "OSHA PSM 29 CFR 1910.119 overlay + API RP 554." },
      { label: "Power Generation (NERC CIP)", description: "NERC CIP-002 / 005 / 007 / 010 / 008 overlay." },
      { label: "Water / Wastewater (AWIA / EPA)", description: "AWIA §2013 risk & resilience for ≥ 3,300-served population utilities." },
      { label: "Chemical Processing (OSHA PSM / CCPS)", description: "PSM + CCPS guidelines." },
      { label: "Discrete Manufacturing (NFPA 79 / OSHA)", description: "NFPA 79 machinery + OSHA general industry." },
      { label: "Other / I'll describe", description: "Use Other to specify; Section 18 will be marked Not Applicable unless additional regulatory overlay is identified." }
    ]
  }
])
```

Then a fifth question (separate `AskUserQuestion` call — `AskUserQuestion` caps at 4 per call):

```
AskUserQuestion([
  {
    question: "Is there an existing PLC indexed in Nexus for this project?",
    header: "Nexus Data",
    multiSelect: false,
    options: [
      { label: "Yes — pull existing plant data from Nexus", description: "Used to pre-populate sections, all flagged [ASSUMED from Nexus — verify with customer]." },
      { label: "No — this is new, no Nexus data exists", description: "Build template from interview only." },
      { label: "Not sure — check Nexus and report what you find", description: "Run plant_summary + list_documented_plcs and surface results." }
    ]
  }
])
```

---

#### Round 1 Follow-Up — Document Revision, Schedule, and Document Owner

Immediately after Round 1 main answers (before Round 2), ask:

```
AskUserQuestion([
  {
    question: "Document revision letter for this issue?",
    header: "Doc Rev",
    multiSelect: false,
    options: [
      { label: "A — first issue (Recommended for new projects)", description: "Use Rev A for the first issue. Increment after every change." },
      { label: "B / C / D / later", description: "Continuing an existing FDS; I'll specify in Other." },
      { label: "Draft / pre-revision", description: "Internal working copy; do not issue. Filename will be -Draft.md." }
    ]
  },
  {
    question: "Project's required-on-site / FAT date?",
    header: "FAT Date",
    multiSelect: false,
    options: [
      { label: "Within 8 weeks — aggressive", description: "Aggressive schedule; flag any TBD-blocking item." },
      { label: "8–16 weeks — typical", description: "Typical schedule." },
      { label: "16+ weeks — long lead", description: "Long lead; standard schedule pressure." },
      { label: "Unknown / not yet set", description: "Skip schedule-relative checks." }
    ]
  },
  {
    question: "Document Owner and Custodian — who will sign the FDS for HIS, and who maintains the master copy?",
    header: "Doc Owner",
    multiSelect: false,
    options: [
      { label: "Timothy Hunt is both Owner and Custodian (Recommended for HIS-internal projects)", description: "Default for HIS engagements." },
      { label: "I'll type both names (Other)", description: "Customer or partner-led ownership; capture both names." }
    ]
  }
])
```

---

#### Interview Round 2 — Process and Equipment (ask all 4 together)

After Round 1 + follow-up are complete:

```
AskUserQuestion([
  {
    question: "Describe what the system does in plain English. Walk through a normal start from dead-stop to full production. Include: what enters the system, what transformations happen, what exits. Aim for 2–4 paragraphs. A new operator should be able to understand the process from this alone.",
    header: "Process Description",
    multiSelect: false,
    options: [{ label: "I'll describe it", description: "Use Other for 2–4 paragraphs of plain-English process narrative." }]
  },
  {
    question: "List every major piece of equipment, ONE PER LINE, in this format: `Tag | Description | Type | Range/Rating | Location | P&ID Ref`. Example: `P-101 | Influent pump | VFD-driven centrifugal | 500 GPM @ 50 ft TDH | Pump Room A | PID-100 SH2`. If you paste free text instead, expect a follow-up to structure it.",
    header: "Equipment List",
    multiSelect: false,
    options: [{ label: "I'll list them in the format", description: "Use Other; one device per line in the pipe-delimited format." }]
  },
  {
    question: "What control platform is being used? (Determines platform-conditional template content for Sections 7.5, 8.1, 11.3, 12.)",
    header: "Control Platform",
    multiSelect: false,
    options: [
      { label: "Rockwell ControlLogix / CompactLogix / GuardLogix (Studio 5000) (Recommended for HIS-default)", description: "Phase Manager + ControlLogix scan model. Most common HIS platform." },
      { label: "Rockwell PLC-5 / SLC 500 (legacy)", description: "Migration project; legacy file/N-file addressing in Section 7.5." },
      { label: "Siemens S7-1200 / S7-1500 (TIA Portal)", description: "S88 state machine in TIA Portal (no Phase Manager); Siemens scan model." },
      { label: "Beckhoff TwinCAT", description: "PackML state machine in TwinCAT; IEC 61131-3 native." },
      { label: "Other / Mixed (I'll describe)", description: "Use Other; Section 7.5/8.1 content will be platform-generic with placeholders." }
    ]
  },
  {
    question: "Which operating modes does this system need? (Select all that apply.)",
    header: "Operating Modes",
    multiSelect: true,
    options: [
      { label: "Auto (fully automatic)", description: "PLC-driven sequence." },
      { label: "Manual (operator-commanded, permissives enforced)", description: "HMI-initiated, interlocks active." },
      { label: "Maintenance (permissives bypassed, key switch)", description: "Keyswitch-protected bypass; bypass logging required." },
      { label: "Step (one-step-at-a-time, used for commissioning)", description: "Operator advances each step." },
      { label: "Hand (hardwired HOA stations)", description: "Local panel control bypassing PLC." },
      { label: "Clean-In-Place (CIP)", description: "Sequenced cleaning cycle; food/pharma." },
      { label: "Emergency Stop (hardwired E-stop circuit)", description: "Always present in some form." },
      { label: "Other (describe in Other)", description: "Custom mode." }
    ]
  }
])
```

After Round 2, if the user selected `Batch / Sequential` operating modes (CIP) or the project type is "Process Skid," ask one targeted follow-up:

```
AskUserQuestion([
  {
    question: "Is recipe management in scope?",
    header: "Recipe Mgmt",
    multiSelect: false,
    options: [
      { label: "Yes — multiple recipes with operator-selectable parameters", description: "Section 8.7 will be populated; recipe structure interview to follow." },
      { label: "Yes — single fixed recipe (no operator switching)", description: "Section 8.7 short-form: parameters fixed in PLC." },
      { label: "No — no recipe concept in this system", description: "Section 8.7 marked Not Applicable." }
    ]
  }
])
```

---

#### Interview Round 3 — Safety, Interfaces, Constraints, Cybersecurity (ask all 4 together)

After Round 2:

```
AskUserQuestion([
  {
    question: "What are the critical safety requirements? Are there SIL or Performance Level requirements?",
    header: "Safety Requirements",
    multiSelect: false,
    options: [
      { label: "No formal SIL/PL requirement — process interlocks only", description: "Section 11 will document process interlocks; SIF tables marked Not Applicable." },
      { label: "SIL 1 required — IEC 61511-1:2016 applies", description: "SIF table populated; SRS reference required." },
      { label: "SIL 2 required — IEC 61511-1:2016 applies", description: "SIF table + PFDavg + Proof Test Interval required." },
      { label: "SIL 3 required — IEC 61511-1:2016 applies", description: "SIF table + redundancy architecture required." },
      { label: "ISO 13849-1:2023 PLc or higher (machinery)", description: "ISO 13849-1 row block in Section 11.1: PLr / Category / MTTFd / DCavg / CCF / SISTEMA Ref required." },
      { label: "Not yet determined — needs risk assessment", description: "Section 11 marked TBD; HAZOP/LOPA reference required before approval." }
    ]
  },
  {
    question: "What external systems does this PLC/SCADA need to talk to? (Select all that apply.)",
    header: "Interfaces",
    multiSelect: true,
    options: [
      { label: "SCADA / DCS (OPC-UA or OPC-DA)", description: "Section 10.2." },
      { label: "Historian (OSIsoft PI / FactoryTalk Historian / Ignition)", description: "Section 10.2 + Section 14.2 retention policy." },
      { label: "MES / ERP (SAP / Plex / other)", description: "Section 10.2 — bidirectional." },
      { label: "Third-party drives or instruments (EtherNet/IP / Modbus / PROFINET)", description: "Section 10.2." },
      { label: "Corporate IT network / cloud", description: "DMZ + Section 6.3 conduit." },
      { label: "Other PLCs via produced/consumed tags", description: "Section 10.3." },
      { label: "No external interfaces", description: "Section 10.2 marked Not Applicable." }
    ]
  },
  {
    question: "Customer standards, constraints, naming, redundancy, enclosure rating? (Select all that apply.)",
    header: "Constraints",
    multiSelect: true,
    options: [
      { label: "Customer has a tagging/naming convention we must match", description: "Section 7.6 populated from convention." },
      { label: "Customer has a specific spare I/O requirement (beyond 20%)", description: "Override the 20% default." },
      { label: "Customer has a cybersecurity policy (IEC 62443 zone requirements)", description: "62443-3-2 risk assessment follow-up will fire." },
      { label: "Customer requires redundant controllers", description: "Section 12 idle-time targets adjusted to 40%+." },
      { label: "Customer requires specific paint, enclosure, or NEMA rating", description: "Captured in Section 6.1." },
      { label: "Existing drawings/P&IDs must be followed", description: "Section 2 reference table." },
      { label: "No special constraints", description: "" }
    ]
  },
  {
    question: "Has a 62443-3-2 cybersecurity risk assessment been performed for this project?",
    header: "Cyber Risk Asmt",
    multiSelect: false,
    options: [
      { label: "Yes — I have the report (I'll provide doc number)", description: "Section 6.3 SL targets pulled from the risk assessment." },
      { label: "Yes — to be performed by HIS as part of this project", description: "Section 6.3 SL targets marked TBD; risk assessment listed as a deliverable." },
      { label: "No — use HIS default tiers as starting placeholders (Recommended for greenfield)", description: "Section 6.3 SL targets pre-filled with [HIS DEFAULT — RECONCILE AGAINST 62443-3-2 RISK ASSESSMENT]." },
      { label: "Out of scope — customer manages cybersecurity separately", description: "Section 6.3 marked Not Applicable; document references customer's cybersecurity program." }
    ]
  }
])
```

---

#### Conditional Follow-Up Questions (trigger based on Round 1–3 answers)

Apply targeted single-question `AskUserQuestion` calls:

- **PLC Migration:** "What is the existing PLC (PLC-5, SLC-500, etc.) and is an L5X / rung export available?"
- **SIL ≥ 1:** "Has a Safety Requirements Specification (SRS) been written? Provide doc number, owner, and revision."
- **Ignition SCADA:** "Standard Ignition Vision or Ignition Perspective (mobile-first)?"
- **Industry = Pharma:** "GAMP 5 software category target (Cat 3 / 4 / 5)? Is 21 CFR Part 11 in scope? Is the system subject to EU Annex 11?"
- **Industry = Power:** "NERC CIP applicability (BES Cyber System categorization — High / Medium / Low Impact)? CIP-002 entity registration?"
- **Industry = Water:** "Population served (drives AWIA §2013 applicability — threshold is 3,300)? Is a Risk & Resilience Assessment current?"
- **Industry = Oil & Gas / Chemical:** "OSHA PSM 29 CFR 1910.119 applicability (threshold quantity of highly hazardous chemicals on site)? PHA report current?"
- **62443 = HIS-default tiers:** Confirm Section 6.3 will carry the `[HIS DEFAULT]` flag and that risk assessment is a project deliverable.
- **Nexus = Yes / Not sure:** Run `mcp__nexus__list_documented_plcs` and `mcp__nexus__plant_summary`; surface PLC list as a follow-up question to confirm which apply.
- **Operating modes include Hand:** "Where are the HOA stations physically located? Are they keyswitch-protected?"

---

#### Nexus Data Integration Rules (CRITICAL)

If Nexus data exists, you MAY use Nexus tools to pre-populate sections of the FDS. **Treat ALL Nexus data as unverified assumptions until confirmed by the user.**

The rules:
- Every Nexus-populated field is flagged: `[ASSUMED from Nexus — verify with customer]`
- Never present Nexus tag names, descriptions, or interlock logic as facts without user confirmation.
- After pre-populating from Nexus, ask the user via `AskUserQuestion` to confirm or correct each assumption.
- Tag descriptions from Nexus are often abbreviated or auto-generated.

**Acceptable Nexus use cases:**
- `mcp__nexus__generate_io_inventory` → Section 4 (Equipment List) and Section 5 (I/O Summary).
- `mcp__nexus__find_interlocks` → Section 7.4 (Interlock Logic).
- `mcp__nexus__generate_sequence_of_operations` → Section 8.
- `mcp__nexus__hmi_project_summary` / `mcp__nexus__screen_inventory` → Section 10.1.
- `mcp__nexus__plant_summary` → Section 1.3 (Project Background).

**Never use Nexus for:**
- Safety function definitions (SIFs) — must come from a customer-approved hazard analysis.
- Security zone assignments — require customer IT/OT input plus 62443-3-2 risk assessment.
- Approval signatures, dates, or document numbers.

---

### Step 2 — Research Applicable Standards (Scoped to Industry Answer)

Based on the Round 1 industry answer, fetch standards-relevant pages — not blog posts. Pull 1–2 sources maximum and cite at least one specific clause in the generated document.

| Industry | Research Targets |
|---|---|
| Pharma / Biotech | GAMP 5 Second Edition (ISPE) — software categorization; 21 CFR Part 11 audit trail § 11.10; EU Annex 11 §4 |
| Food & Beverage | FSMA 21 CFR Part 117 preventive controls; 3-A Sanitary Standards relevant equipment |
| Oil & Gas | OSHA PSM 29 CFR 1910.119 §(d) PSI requirements; API RP 554 process control system |
| Power Generation | NERC CIP-002-5.1a (BES Cyber System categorization); CIP-005 ESP; CIP-007 system management |
| Water / Wastewater | AWIA §2013 (PL 115-270); EPA risk & resilience assessment guidance; AWWA Manual M2 |
| Chemical Processing | OSHA PSM (same as oil & gas); CCPS Guidelines for Safe and Reliable Instrumented Protective Systems |
| Discrete Manufacturing | NFPA 79-2024 §13 wire identification; ISO 13849-1:2023 §4–§6 PL determination |

Use `mcp__scrapling__fetch` with `disable_resources: true` and `main_content_only: true`. If a fetch fails, proceed with built-in standards knowledge.

---

### Step 3 — Generate the FDS Document

Produce the complete FDS as a single Markdown document. Every section is REQUIRED. Use `[TBD — REQUIRES CUSTOMER INPUT]` for missing data; never omit a section — write "Not applicable — [reason]" where genuinely not applicable.

Where data came from Nexus and has not been confirmed, mark it `[ASSUMED from Nexus — verify with customer]`.

Where data carries a HIS default that should be reconciled later, mark it `[HIS DEFAULT — RECONCILE BEFORE APPROVAL]`.

Write as if this document will be reviewed by the customer's engineering team and used as testimony in a commissioning dispute. Be specific, be unambiguous, be traceable.

**Platform-conditional content** — wherever the template uses `{{IF_PLATFORM_ROCKWELL}}` / `{{IF_PLATFORM_SIEMENS}}` / `{{IF_PLATFORM_BECKHOFF}}` / `{{IF_PLATFORM_OTHER}}`, emit only the block that matches the Round 2 platform answer.

**Industry-conditional content** — Section 18 emits one or more sub-sections matched to the Round 1 industry answer.

---

## FDS Document Template

````markdown
# Functional Design Specification
## {{PROJECT_NAME}}

---

| Field | Value |
|---|---|
| Document Number | {{HIS_DOC_NUMBER}} |
| Revision | {{REV}} |
| Status | {{DOC_STATUS}} (DRAFT — internal / FOR REVIEW — customer review / APPROVED — signed) |
| Date | {{TODAY_DATE}} |
| Prepared By | Timothy Hunt, Hunt Integrative Solutions LLC |
| Document Owner | {{DOC_OWNER}} |
| Document Custodian | {{DOC_CUSTODIAN}} |
| Customer | {{CUSTOMER_NAME}} |
| Site | {{SITE_LOCATION}} |
| Project Type | {{PROJECT_TYPE}} |
| Industry | {{INDUSTRY}} |
| Required-on-site / FAT Date | {{FAT_DATE}} |

---

## Table of Contents

1. Purpose and Scope (1.1 Purpose · 1.2 Scope · 1.3 Project Background · 1.4 Requirements Traceability Matrix · 1.5 Assumptions and Exclusions · 1.6 Document Owner, Review Cycle, MOC)
2. Reference Documents
3. System Description (3.1 Process Overview · 3.2 ISA-88 Equipment Hierarchy · 3.3 Operating Modes)
4. Equipment List
5. I/O Summary
6. Hardware and Network Architecture (6.1 Control System Hardware · 6.2 Network Topology · 6.3 IEC 62443 Security Zones and Conduits)
7. Control Philosophy (7.1 Overview · 7.2 Control Loops · 7.3 Permissive Logic · 7.4 Interlock Logic · 7.5 Analog Scaling · 7.6 Tag Naming Convention · 7.7 Power-Up & Recovery · 7.8 Failure-Mode Response by Device Class · 7.9 Manual Override / Force Indication)
8. Sequence of Operations (8.1 Startup · 8.2 Normal Running · 8.3 Normal Shutdown · 8.4 Emergency Shutdown · 8.5 Additional Modes · 8.6 ISA-88 Phase State Model · 8.7 Recipe Management)
9. Alarm Philosophy
10. Interfaces (10.1 HMI / SCADA · 10.2 External Systems · 10.3 Inter-PLC)
11. Safety Architecture (11.1 SIF / Function Summary · 11.2 E-Stop Architecture · 11.3 Safety Logic Rules · 11.4 SRS Reference · 11.5 HAZOP / LOPA Reference · 11.6 FSM Plan · 11.7 Bypass Management)
12. Performance Requirements
13. Backup, Historian, Spare-Parts, Diagnostic Logging
14. FAT / SAT Acceptance Criteria
15. Open Items and TBDs
16. Glossary
17. Revision History
18. Industry-Specific Compliance Overlay (conditional — emitted only when an industry overlay applies)

---

## Revision History

| Rev | Date | Author | Description |
|---|---|---|---|
| {{REV}} | {{TODAY_DATE}} | T. Hunt | Initial draft for customer review |

---

## Approval Signatures

| Role | Name | Signature | Date |
|---|---|---|---|
| Prepared By | Timothy Hunt / HIS | | |
| Reviewed By (HIS) | | | |
| Approved By (Customer) | | | |
| Approved By (Customer Engineering) | | | |

**This document is not valid for design until all approval signatures are obtained.**

---

## 1. Purpose and Scope

### 1.1 Purpose

This Functional Design Specification (FDS) defines the functional requirements for the {{PROJECT_NAME}} control system. It establishes WHAT the system must do — not HOW the software implements it. This document is the contractual basis for all subsequent design, coding, testing, and commissioning activities.

This FDS is the primary reference for:
- PLC program development
- HMI/SCADA configuration
- Factory Acceptance Test (FAT) procedure development
- Site Acceptance Test (SAT) procedure development
- Operator training
- As-built documentation
- Regulatory audit (industry-specific overlay in Section 18)

### 1.2 Scope

**In Scope:**
{{IN_SCOPE_ITEMS — list each system, subsystem, geographic boundary; reference RTM Req IDs.}}

**Out of Scope:**
{{OUT_OF_SCOPE_ITEMS — explicitly state what is NOT covered.}}

**System Boundary:**
{{BOUNDARY_DESCRIPTION — reference P&ID drawing numbers.}}

### 1.3 Project Background

{{2–4 sentences: why this project, what problem it solves, what was there before.}}

### 1.4 Requirements Traceability Matrix (RTM)

Per **ANSI/ISA-5.06.01-2007 §5.6**, every requirement traces from its source (URS / customer spec / regulatory clause / HAZOP / standard) through this FDS to a verification method and a FAT/SAT test case. Status closes when the test case passes at FAT or SAT.

| Req ID | Source | FDS Section | Verification Method (Test / Inspection / Analysis / Demonstration) | Test Case ID | Status (Open / Verified / Waived) |
|---|---|---|---|---|---|
| REQ-001 | {{SOURCE}} | §3.3 Operating Modes | Demonstration | TC-MODE-01 | Open |
| REQ-002 | {{SOURCE}} | §6.3 Security Zones | Inspection | TC-SEC-01 | Open |
| REQ-003 | {{SOURCE}} | §11.1 SIF | Test (proof-test procedure) | TC-SIF-01 | Open |
| {{REQ-XXX}} | {{SOURCE}} | {{SECTION}} | {{METHOD}} | {{TC}} | Open |

**RTM maintenance:** New requirements added during review get a new Req ID; deleted requirements are marked Waived (with rationale) — never removed from the table.

### 1.5 Assumptions and Exclusions

Every assumption and exclusion is registered with an ID, source, validity window, impact-if-invalid, and owner. Commissioning disputes start here.

| ID | Assumption / Exclusion | Source | Validity Window | Impact If Invalid | Owner |
|---|---|---|---|---|---|
| AS-001 | Customer provides 480V 3-phase utility power per electrical one-line | Customer | Project duration | Power-up impossible; schedule impact | Customer |
| AS-002 | All field instruments are calibrated by the customer prior to loop check | HIS scope | Through SAT | Loop check delayed; schedule impact | Customer |
| AS-003 | Customer maintains the historian server hardware and OS patching | HIS scope | Project duration | Historian outage during commissioning | Customer |
| AS-004 | Customer's IT department provides corporate-network IP allocations and DNS | HIS scope | Through commissioning | Cannot configure SCADA-to-corporate interface | Customer IT |
| EX-001 | Field-instrument supply (other than vendor-supplied skid devices) is not in HIS scope | Contract | Project duration | Procurement gap; schedule impact | Customer |
| {{ID}} | {{ASSUMPTION}} | {{SOURCE}} | {{WINDOW}} | {{IMPACT}} | {{OWNER}} |

### 1.6 Document Owner, Review Cycle, Management of Change

**Document Owner:** {{DOC_OWNER}} — accountable for content accuracy and approval signatures.
**Document Custodian:** {{DOC_CUSTODIAN}} — maintains the master copy; processes MOC requests; updates revision history.
**Review Cycle:**
- DRAFT → FOR REVIEW: when all `[TBD]` items are resolved or moved to Open Items (Section 15) with owner and due date.
- FOR REVIEW → APPROVED: when all signatures in the Approval Signatures table above are obtained.
- APPROVED → revised: only via Management of Change (below).

**Management of Change (MOC) Procedure:**
1. Change requestor submits a written change request to the Document Custodian, citing the affected Req ID(s) from Section 1.4.
2. Custodian performs an impact-analysis: verification methods affected, test cases requiring re-execution, downstream documents requiring revision (`/io-list`, `/bom`, `/wire-diagram`, `/panel-layout`, `/prog-arch`, `/hmi-spec`, `/alarm-philosophy`, `/soo`).
3. Document Owner authorizes the change (with customer signature for changes after FOR REVIEW).
4. Custodian increments the Rev letter, updates Section 17 Revision History with the change description, and re-issues the FDS.
5. Affected verification methods are re-executed; RTM Status reset to Open for impacted Req IDs.

---

## 2. Reference Documents

| Document | Number / Revision | Description |
|---|---|---|
| P&ID | {{PID_NUMBER}} | Process and Instrumentation Diagram |
| Electrical One-Line | {{ONELINE_NUMBER}} | Main power distribution |
| Panel Layout | {{PANEL_DRAWING}} | Control panel arrangement (see `/panel-layout` output if available) |
| Network Diagram | {{NETWORK_DRAWING}} | Control network topology |
| BOM | {{BOM_NUMBER}} | Hardware Bill of Materials (see `/bom` output if available) |
| I/O List | {{IOL_NUMBER}} | Field signal list (see `/io-list` output if available) |
| Wire Schedule | {{WIRE_NUMBER}} | Point-to-point wiring (see `/wire-diagram` output if available) |
| ANSI/ISA-5.06.01-2007 | — | Functional Requirements Documentation for Control Software |
| ANSI/ISA-88.01-2010 | — | Batch Control: Models and Terminology |
| ANSI/ISA-18.2-2016 | — | Management of Alarm Systems |
| ANSI/ISA-101.01-2015 | — | Human Machine Interfaces |
| EEMUA 191:2013 (Edition 3) | — | Alarm Systems: Design, Management and Procurement |
| IEC 61511-1:2016 (Edition 2) | — | Functional Safety — Safety Instrumented Systems (if applicable) |
| IEC 62443-2-1:2010 | — | IACS Security Program |
| IEC 62443-3-2:2020 | — | Security Risk Assessment for System Design |
| IEC 62443-3-3:2013 | — | System Security Requirements and Security Levels |
| ISO 13849-1:2023 (Edition 4) | — | Safety of Machinery — Performance Levels (if applicable) |
| NFPA 79-2024 | — | Electrical Standard for Industrial Machinery (if applicable) |
| NEC 2023 (NFPA 70) | — | Articles 110.26 (working clearance) and 409 (industrial control panels) |
| NIST SP 800-82 Rev. 3 | — | Guide to Operational Technology (OT) Security |
| {{INDUSTRY_STANDARD_1}} | {{REV}} | Industry overlay (e.g., GAMP 5, NERC CIP-002, AWIA §2013, OSHA PSM 1910.119) |
| {{CUSTOMER_STD}} | {{REV}} | Customer engineering standard |

---

## 3. System Description

### 3.1 Process Overview

{{PROCESS_NARRATIVE — 2–4 paragraphs of plain English. No acronyms on first use. What enters, what transformations, what exits. Written so a new operator can understand it on day one.}}

### 3.2 ISA-88 Equipment Hierarchy

The physical equipment is mapped to the **ISA-88.01-2010** model:

```
Enterprise: {{COMPANY_NAME}}
  Site: {{SITE_NAME}}
    Area: {{AREA_NAME}}
      Process Cell: {{CELL_NAME}}
        Unit(s):
          {{UNIT_1_NAME}} — {{UNIT_1_DESCRIPTION}}
          {{UNIT_2_NAME}} — {{UNIT_2_DESCRIPTION}}
        Equipment Modules:
          {{EM_1}} — {{EM_1_DESCRIPTION}}
          {{EM_2}} — {{EM_2_DESCRIPTION}}
        Control Modules:
          {{CM_1}} — {{CM_1_DESCRIPTION}}
```

### 3.3 Operating Modes

Each mode is mutually exclusive unless noted. Modes match the Round 2 Q4 answer set; any mode not in this table that was selected in Round 2 is a defect.

| Mode | Description | Activation | Priority |
|---|---|---|---|
| Emergency Stop (E-Stop) | All outputs de-energized, safe state achieved | Hardwired E-stop circuit | 1 — Highest |
| Safety Shutdown | SIFs execute; controlled shutdown sequence | SIS trip | 2 |
| Auto | Fully automatic per Sequence of Operations | HMI "Auto" command | 3 |
| Step | Auto sequence advances one step per operator command | HMI "Step" command | 4 |
| Manual | Individual equipment commanded by operator; permissives enforced | HMI "Manual" command | 5 |
| Maintenance | Permissives bypassed via key switch; bypasses logged | Physical key switch + HMI | 6 |
| {{ADDITIONAL_MODE}} | {{DESCRIPTION}} | {{ACTIVATION}} | {{PRIORITY}} |

**Mode transition rules:**
- E-Stop overrides all other modes at any time.
- Maintenance → Manual requires key switch to Normal position.
- Auto → Manual: {{bumpless or state-transfer behavior}}
- Manual → Auto: {{conditions required, pre-conditions}}

---

## 4. Equipment List

Signal type codes: DI / DO / AI / AO / RTD / TC / HSC / SAFE-DI / SAFE-DO / COMM.

| Tag | Description | Type | Range / Rating | Location | P&ID Ref | Notes |
|---|---|---|---|---|---|---|
| {{TAG_1}} | {{DESCRIPTION}} | {{TYPE}} | {{RANGE}} | {{LOCATION}} | {{PID_REF}} | |
| {{TAG_2}} | {{DESCRIPTION}} | {{TYPE}} | {{RANGE}} | {{LOCATION}} | {{PID_REF}} | |

**Spares:** Minimum 20% spare capacity on all I/O cards per HIS standard (or customer-specified higher per Round 3 Constraints).

**Critical devices:**

| Tag | Criticality | Fail-Safe State | Justification |
|---|---|---|---|
| {{TAG}} | Safety / Process / None | {{STATE}} | {{REASON}} |

---

## 5. I/O Summary

| Signal Type | Used | Spare (≥20% or customer-specified) | Card Count | Module Type |
|---|---|---|---|---|
| Digital Input (24VDC) | {{COUNT}} | {{COUNT}} | {{COUNT}} | {{MODULE}} |
| Digital Output (24VDC) | {{COUNT}} | {{COUNT}} | {{COUNT}} | {{MODULE}} |
| Analog Input (4–20mA) | {{COUNT}} | {{COUNT}} | {{COUNT}} | {{MODULE}} |
| Analog Output (4–20mA) | {{COUNT}} | {{COUNT}} | {{COUNT}} | {{MODULE}} |
| RTD / TC | {{COUNT}} | {{COUNT}} | {{COUNT}} | {{MODULE}} |
| Safety DI | {{COUNT}} | {{COUNT}} | {{COUNT}} | {{MODULE}} |
| Safety DO | {{COUNT}} | {{COUNT}} | {{COUNT}} | {{MODULE}} |
| **TOTAL** | | | | |

---

## 6. Hardware and Network Architecture

### 6.1 Control System Hardware

| Component | Model / Part Number | Quantity | Location | Notes |
|---|---|---|---|---|
| PLC Processor | {{MODEL}} | {{QTY}} | {{PANEL}} | Firmware: {{VER}} |
| I/O Chassis | {{MODEL}} | {{QTY}} | {{PANEL}} | |
| Power Supply | {{MODEL}} | {{QTY}} | {{PANEL}} | Redundant: {{Y/N}} |
| HMI Terminal | {{MODEL}} | {{QTY}} | {{LOCATION}} | Screen size: {{SIZE}} |
| Engineering Workstation | {{SPEC}} | 1 | {{LOCATION}} | {{Studio 5000 / TIA Portal / TwinCAT}} v{{VER}} |
| Managed Switch | {{MODEL}} | {{QTY}} | {{LOCATION}} | DLR / RSTP: {{TOPOLOGY}} |

### 6.2 Network Topology

{{DESCRIPTION — plain English, reference network drawing.}}

Network drawing: {{NETWORK_DRAWING_NUMBER}}

**Layer 2 / Layer 3 layout:**
- Control network (VLAN {{ID}}): {{IP_RANGE}} — PLC, HMI, drives, I/O
- OT/IT DMZ: {{firewall / conduit between control and corporate}}
- Remote access: {{VPN concentrator, jump host, MFA method}}

**Device IP Allocation Table:**

| Device | Hostname | IP Address | Subnet | VLAN | MAC | Default Gateway | Notes |
|---|---|---|---|---|---|---|---|
| {{DEVICE}} | {{HOSTNAME}} | {{IP}} | {{SUBNET}} | {{VLAN}} | {{MAC}} | {{GW}} | |

**CIP Connection Budget** (ControlLogix Ethernet/IP modules):

| Module | Connections Used | Connections Available | Margin |
|---|---|---|---|
| {{1756-EN2TR slot N}} | {{USED}} | {{AVAIL}} | {{MARGIN}} |

**Produced/Consumed RPI Strategy:**
- High-speed inter-PLC interlocks: RPI = 2× consuming task period (typical 20 ms RPI for 50 ms task).
- Continuous process values: RPI = consuming task period × 4.
- Recipe / batch / configuration data: use MSG (not produced/consumed) — lower bandwidth, on-demand.
- Safety inter-PLC: CIP Safety produced/consumed only — never MSG.

**NTP / Time Sync:**
- NTP server(s): {{SERVER}} (stratum {{STRATUM}})
- Sync hierarchy: PLC → managed switch → corporate NTP → public NTP
- Accuracy target: ±{{MS}} ms across all PLCs and HMI clients
- Affects: alarm timestamps, historian, audit trail, security event logs

**DNS / Active Directory / FactoryTalk Security integration:**
- AD domain: {{DOMAIN}}
- FactoryTalk Security mode: {{Windows-Linked / Local}}
- Roles: {{LIST customer-specific roles}}

### 6.3 IEC 62443 Security Zones and Conduits

Per **IEC 62443-3-2:2020**, security zones are defined based on a documented risk assessment. Zone targets below are {{either pulled from the customer's risk-assessment report (REF: {{DOC}}) OR marked `[HIS DEFAULT — RECONCILE AGAINST 62443-3-2 RISK ASSESSMENT]` per Round 3 answer}}.

**System Under Consideration (SuC) boundary** (per 62443-3-2 §5.4):

{{Describe the SuC: what is in / out, the physical and logical boundary. Reference network drawing.}}

**Zones:**

| Zone | Assets | Target Security Level (SL-T) per 62443-3-3 | Notes |
|---|---|---|---|
| Zone 1 — Safety | {{Safety PLC, safety I/O}} | {{SL_T}} | Physically isolated; CIP Safety only |
| Zone 2 — Control | {{Standard PLC, HMI, drives}} | {{SL_T}} | Managed switch with ACLs |
| Zone 3 — Supervisory | {{SCADA server, historian}} | {{SL_T}} | |
| Zone 4 — Corporate | Business network | {{SL_T}} | |

**Conduits (boundaries between zones):**
- **Conduit A** (Zone 2 → Zone 3): {{firewall / DMZ configuration}}
- **Conduit B** (Zone 3 → Zone 4): {{IT/OT gateway, unidirectional if required}}

**System Requirements (SR) per IEC 62443-3-3:2013** — applicability to this SuC:

| SR Family | Topic | Applicability | Implementation Reference |
|---|---|---|---|
| SR 1.x | Identification & Authentication | {{Y/N}} | FactoryTalk Security / AD integration in §6.2 |
| SR 2.x | Use Control | {{Y/N}} | Role-based access matrix in §10.1 |
| SR 3.x | System Integrity | {{Y/N}} | CIP Security / signed firmware |
| SR 4.x | Data Confidentiality | {{Y/N}} | TLS / CIP Security on conduits |
| SR 5.x | Restricted Data Flow | {{Y/N}} | Conduit ACLs |
| SR 6.x | Timely Response to Events | {{Y/N}} | Event logging — see §13.4 |
| SR 7.x | Resource Availability | {{Y/N}} | Backup / restore — see §13.1 |

---

## 7. Control Philosophy

### 7.1 Overview

{{2–4 paragraphs describing the overall strategy. Setpoint-driven PID? Sequence-based? Event-driven? Where does logic live?}}

**Fundamental rule:** All process logic resides in the PLC. The HMI is a display and command interface only. No critical process decisions in HMI scripting.

### 7.2 Control Loops

| Loop Tag | Description | Type | Setpoint Source | Output | Tuning Approach |
|---|---|---|---|---|---|
| {{PID_TAG}} | {{DESCRIPTION}} | PID / Cascade / Ratio / Feedforward | {{HMI / Recipe / Hardwired}} | {{OUTPUT_TAG}} | {{Lambda / IMC / Ziegler-Nichols / Manufacturer Recommended}} |

**Loop tuning philosophy:** Use {{LAMBDA tuning for slow process loops; IMC for first-order plus dead-time; Ziegler-Nichols only as a starting point}}. Final tuning parameters are recorded in the As-Built FDS.

### 7.3 Permissive Logic

Equipment shall not start unless ALL permissives are satisfied. Permissives are enforced in Auto and Manual; Maintenance bypasses with key switch.

**{{EQUIPMENT_TAG}} — Start Permissives:**

| Permissive | Tag | Required State | Bypass in Maintenance? |
|---|---|---|---|
| {{PERMISSIVE_1}} | {{TAG}} | {{STATE}} | {{Y/N}} |

### 7.4 Interlock Logic

Interlocks act on running equipment.

**{{EQUIPMENT_TAG}} — Running Interlocks:**

| Condition | Tag | Trip Value | Action | Reset Method |
|---|---|---|---|---|
| {{CONDITION}} | {{TAG}} | {{VALUE}} | {{ACTION}} | {{MANUAL/AUTO}} |

### 7.5 Analog Scaling and Engineering Units

All analog signals shall be scaled to engineering units in the I/O Mapping routine. Raw I/O addresses shall not be used outside the I/O Mapping routine.

{{IF_PLATFORM_ROCKWELL}}
Raw I/O example: `Local:1:I.Ch0Data` (ControlLogix). Scaling AOI: `P_AIn` (PlantPAx) or custom `IO_Scale_AOI`.
{{IF_PLATFORM_SIEMENS}}
Raw I/O example: `%IW256` (S7-1500 PIW). Scaling: `SCALE_X` instruction or custom FB_AnalogScale.
{{IF_PLATFORM_BECKHOFF}}
Raw I/O example: `%IW0` (TwinCAT). Scaling: custom FB_AnalogScale function block.
{{IF_PLATFORM_OTHER}}
Raw I/O example: {{platform-specific syntax}}. Scaling: {{platform-specific function/AOI}}.

| Tag | Raw Count | Engineering Low | Engineering High | Units | Out-of-Range Action |
|---|---|---|---|---|---|
| {{TAG}} | {{0–32767 / 0–27648 / native}} | {{EU_LOW}} | {{EU_HIGH}} | {{UNITS}} | {{ACTION}} |

### 7.6 Tag Naming Convention

Convention selected per Round 3 Constraints answer:

- **HIS standard:** `[Area][EquipID]_[Suffix]` where Suffix is one of `_Cmd`, `_FB`, `_Sts`, `_Alm`, `_SP`, `_PV`, `_OL`, `_Run`, `_Flt`. Example: `100_P101_RunCmd`.
- **Customer convention:** {{describe — provided by customer per Round 3.}}
- **Example tags:** {{three examples covering DI, DO, and analog from Round 2 equipment.}}

Full tag-database design in `/prog-arch` output (cross-reference).

### 7.7 Power-Up & Recovery Behavior

| Event | System Behavior | Operator Action Required |
|---|---|---|
| Cold power-up after planned outage | Outputs de-energized; mode = Stopped; permissives evaluated; operator must issue Start | Confirm process safe; issue Start |
| Power-blip recovery (UPS-supported) | PLC continues; HMI re-syncs from last known state; running equipment may be dropped depending on duration | None if duration < UPS hold; review alarms |
| PLC fault recovery | Faulted controller switches to safe state; redundant pair switches over (if configured) | Investigate fault; issue Reset |
| Network partition recovery | I/O drops re-establish; produced/consumed tags resume | Confirm all I/O drops green |

**Auto-restart policy:** {{Auto-restart Disabled (Recommended for safety) / Auto-restart Enabled with delay (specify delay) / Mode-dependent.}}

### 7.8 Failure-Mode Response by Device Class

| Device Class | Failure Mode | PLC Response | Process Impact |
|---|---|---|---|
| Drive (VFD) | Comms loss to drive | Output speed = last known for {{N}} seconds, then trip; alarm | Equipment trip after timeout |
| Drive (VFD) | Drive fault | Trip running command; alarm; await operator reset | Immediate trip |
| Field instrument (AI) | Signal out-of-range (high) | Use last good value; alarm; operator alerted | Setpoint hold; manual fallback |
| Field instrument (AI) | Signal out-of-range (low) | Use last good value; alarm | Setpoint hold |
| I/O drop (remote) | Comms loss | All outputs in drop go to fail-safe state per Equipment List Section 4 | Drop offline; affected equipment trips |
| HMI server | Loss of HMI | PLC continues; alarms accumulate; on HMI restore, alarm history backfilled | Operator visibility lost; PLC continues to run |
| Controller (simplex) | Major fault | Outputs de-energize; standby PLC takes over (if redundant) | Bumpless switchover (redundant) / process trip (simplex) |
| Safety I/O | CIP Safety connection loss | Safe state; no recovery without operator reset | Immediate trip |

### 7.9 Manual Override / Force / Bypass Indication

Per **ANSI/ISA-101.01-2015**, every manual override, force, or bypass is visible plant-wide on the HMI:

- A persistent **Override Indicator** appears on every screen header showing count of active overrides / forces / bypasses.
- Equipment with an active override displays a yellow border on its faceplate and on every screen showing that equipment.
- Override list screen accessible from header — shows tag, override value, applied-by user, applied-at timestamp, expected-removal-time (if maintenance bypass).
- Maintenance-mode bypasses are logged to the audit trail (user, equipment, start time, end time, justification).

---

## 8. Sequence of Operations

### 8.1 Startup Sequence

Transition from {{INITIAL_STATE}} to {{RUNNING_STATE}} in Auto mode.

{{IF_PLATFORM_ROCKWELL}}
Implemented using **Rockwell Phase Manager** (ISA-88.01-2010 procedural model). Phase instructions: PSC, PCMD, PFL, PXRQ, PATT, POVL, PCLF, PPD, PRNP.
{{IF_PLATFORM_SIEMENS}}
Implemented using an **S88 state machine in TIA Portal** following the ISA-88.01-2010 procedural model. State machine implemented as a custom FB; equivalent to Phase Manager but coded in SCL.
{{IF_PLATFORM_BECKHOFF}}
Implemented using **PackML state machine in TwinCAT** (ISA-TR88.00.02 OMAC PackML). State machine in IEC 61131-3 Sequential Function Chart or Structured Text.
{{IF_PLATFORM_OTHER}}
Implemented using a {{platform-specific state-machine engine}} following the ISA-88.01-2010 procedural model.

| Step | Description | Entry Condition | Action | Exit Condition | Fault Condition |
|---|---|---|---|---|---|
| S01 | {{STEP_NAME}} | {{ENTRY}} | {{ACTION}} | {{EXIT}} | {{FAULT}} |
| S02 | {{STEP_NAME}} | {{ENTRY}} | {{ACTION}} | {{EXIT}} | {{FAULT}} |

**Timeout handling:** If any step does not complete within {{TIMEOUT}} seconds, the sequence faults, logs the active step number and timestamp, and transitions to Stopping.

### 8.2 Normal Running

{{Describe ongoing auto operation — PID, feed-forward, ratio control, level management.}}

### 8.3 Shutdown Sequence — Normal

| Step | Description | Action |
|---|---|---|
| S01 | {{STEP_NAME}} | {{ACTION}} |

### 8.4 Shutdown Sequence — Emergency

Triggered by hardwired E-stop, safety PLC trip, or {{OTHER_TRIGGER}}.

All outputs de-energize simultaneously (not sequenced). System shall not restart until operator resets from HMI after cause has been cleared.

### 8.5 Additional Operating Sequences

{{Describe CIP, Grade Change, Filling, Purge, etc., as applicable.}}

### 8.6 ISA-88 Phase State Model

{{If Round 2 includes batch / sequential or CIP: emit the full state model below. Otherwise: "Not applicable — no batch or sequential operations in scope."}}

The Phase state model per **ANSI/ISA-88.01-2010 §5.3.6.4**:

```
                  Resetting
                      ↓
                    Idle
                      ↓
                  Running ─────────→ Holding
                  ↓     ↑              ↓
              Complete  ↑             Held
                        ↑              ↓
                        ↑          Restarting
                        ↓
                    Stopping
                        ↓
                    Stopped
                        ↓
                   Aborting
                        ↓
                    Aborted
```

| State | Description | Entry Trigger | Exit Trigger |
|---|---|---|---|
| Idle | Phase ready to accept Start | Reset Complete | Start command |
| Running | Phase executing normal logic | Start command | PSC / Hold / Stop / Abort |
| Holding | Phase transitioning to Held | Hold command | Hold complete |
| Held | Phase paused; outputs in safe-hold state | Holding complete | Restart / Stop / Abort |
| Restarting | Phase resuming from Held | Restart command | Restart complete (→ Running) |
| Stopping | Controlled shutdown sequence | Stop command | Stop complete |
| Stopped | Phase stopped normally | Stopping complete | Reset command |
| Aborting | Emergency / fault shutdown sequence | Abort or PFL | Abort complete |
| Aborted | Phase aborted; requires operator acknowledge | Aborting complete | Reset command |
| Complete | Phase finished normal cycle | PSC instruction | Reset command |
| Resetting | Phase clearing state for next start | Reset command | Reset complete (→ Idle) |

### 8.7 Recipe Management

{{If Round 2 follow-up = "Yes — multiple recipes" or "Yes — single fixed recipe": emit the full structure below. Otherwise: "Not applicable — no recipe management in scope."}}

**Recipe structure** (per **ANSI/ISA-88.01-2010** master/control recipe model):

| Recipe Type | Contents | Storage | Edit Authority | Audit Trail |
|---|---|---|---|---|
| Master Recipe | Product-level recipe (formula, procedure, equipment requirements) | {{HMI / Recipe Manager / SQL}} | Engineer (per role matrix) | Required |
| Control Recipe | Master recipe instantiated for a specific batch | PLC tags | Read-only at run | Required |
| Recipe Parameters | Setpoints, limits, durations | UDT in PLC | Operator (within bounds) / Supervisor (full) | Required |

**Recipe-modify role-based access:**

| Role | Read | Run (load to control recipe) | Edit Parameters | Edit Master Recipe | Approve |
|---|---|---|---|---|---|
| Operator | Y | Y | Y (within bounds) | N | N |
| Supervisor | Y | Y | Y | N | Y |
| Engineer | Y | Y | Y | Y | Y |
| Read-Only | Y | N | N | N | N |

**Recipe versioning:** Every master recipe carries a version number; control recipes record the master recipe version they were instantiated from. Recipe changes require MOC per §1.6.

---

## 9. Alarm Philosophy

Per **ANSI/ISA-18.2-2016** and **EEMUA 191:2013**, alarms are rationalized before configuration. An alarm requires an operator response within the required response time.

### 9.1 Alarm Classification

| Class | Severity | Description | Annunciation | Required Response Time |
|---|---|---|---|---|
| Critical | 1 | Immediate safety or environmental consequence | Audible + Visual (red) | < 5 minutes |
| High | 2 | Process upset, production impact | Visual (red) | < 15 minutes |
| Medium | 3 | Abnormal condition approaching a limit | Visual (yellow) | < 60 minutes |
| Low | 4 | Advisory / informational | Visual (blue/white) | Next shift |
| Event | — | Informational only, no operator action | Event log only | — |

**Alarm flood target:** < 1 alarm per 10 minutes per operator under normal conditions (ISA-18.2 KPI).

### 9.2 Alarm Rationalization Register

Reference the standalone Alarm Philosophy document (see `/alarm-philosophy` output) for the full register. Summary in this FDS:

| Alarm Tag | Description | Class | Setpoint | Deadband | Delay (s) | Response Action | P&ID Ref |
|---|---|---|---|---|---|---|---|
| {{ALARM_TAG}} | {{DESCRIPTION}} | {{CLASS}} | {{SP}} | {{DB}} | {{DELAY}} | {{ACTION}} | {{PID}} |

### 9.3 Alarm Suppression Rules

- Alarms suppressed during startup sequences where the process is intentionally outside normal operating range. Suppression windows: {{LIST}}.
- No alarm permanently suppressed without documented customer approval and an MOC record.

---

## 10. Interfaces

### 10.1 HMI / SCADA Interface

**Platform:** {{FACTORYTALK VIEW SE / VIEW ME / IGNITION PERSPECTIVE / IGNITION VISION / OTHER}}

**Navigation hierarchy** (≤ 3 clicks per **ANSI/ISA-101.01-2015**):
- Level 1 — Plant Overview: status, active alarms count
- Level 2 — Area Overview: equipment group status, trends
- Level 3 — Equipment Faceplate: individual device control, setpoints
- Level 4 — Diagnostic: I/O status, tag values, comms health

**Color philosophy** (ISA-101 / ISA-5.1):
- Gray background (#A0A0A0) — process equipment in normal state
- Red — alarm condition
- Yellow — warning / abnormal / override
- Green — running / open / energized
- White — off / closed / de-energized
- Color is reserved for conveying information, not decoration.

**User Roles & Permission Matrix:**

| Role | View | Acknowledge Alarm | Issue Command | Edit Setpoint | Force I/O | Edit Recipe | Configure |
|---|---|---|---|---|---|---|---|
| Read-Only / Observer | Y | N | N | N | N | N | N |
| Operator | Y | Y | Y | Y (bounded) | N | Y (bounded) | N |
| Supervisor | Y | Y | Y | Y | N | Y | N |
| Engineer | Y | Y | Y | Y | Y (logged) | Y | Y |
| Administrator | Y | Y | Y | Y | Y | Y | Y |

Cross-reference: full HMI screen specification in `/hmi-spec` output.

### 10.2 External System Interfaces

| System | Protocol | Direction | Data Exchanged | Update Rate | Owner |
|---|---|---|---|---|---|
| {{SCADA}} | {{OPC-UA / Modbus / EtherNet/IP}} | PLC → SCADA | {{TAG_LIST}} | {{RATE}} | {{OWNER}} |
| {{HISTORIAN}} | {{OPC-UA / OPC-DA}} | PLC → Historian | {{TAG_LIST}} | {{RATE}} | {{OWNER}} |
| {{MES}} | {{REST API / OPC-UA}} | Bidirectional | {{RECIPE / PRODUCTION}} | {{RATE}} | {{OWNER}} |
| {{VFD}} | EtherNet/IP CIP | PLC ↔ VFD | Speed ref, status, faults | 50 ms | HIS |
| {{INSTRUMENT}} | HART / 4-20 mA | Field → PLC | {{PV}} | {{SCAN}} | HIS |

### 10.3 Produced/Consumed Tag Interfaces (Inter-PLC)

| Produced Tag | Source PLC | Consumer PLC | Data | RPI (ms) | Purpose |
|---|---|---|---|---|---|
| {{TAG}} | {{PLC}} | {{PLC}} | {{DATA}} | {{RPI}} | {{PURPOSE}} |

---

## 11. Safety Architecture

### 11.1 Safety Function Summary

Per **IEC 61511-1:2016 (Edition 2)**, the following Safety Instrumented Functions (SIFs) are identified. SIL verification calculations are provided in the SIL Verification Report referenced in §11.4 (SRS).

| SIF ID | Description | Mode (Demand / Continuous) | SIL Target | RRF Target | Architecture (1oo1 / 1oo2 / 2oo3) | PFDavg or PFH (target) | Proof Test Interval | Initiator(s) | Final Element(s) | SRS Ref | HAZOP/LOPA Ref |
|---|---|---|---|---|---|---|---|---|---|---|---|
| SIF-01 | {{DESCRIPTION}} | Demand | SIL {{1/2/3}} | {{RRF}} | {{ARCH}} | {{PFDavg}} | {{months}} | {{TAG}} | {{TAG}} | {{SRS-DOC}} §{{X}} | {{HAZOP-DOC}} item {{Y}} |

**Machinery safety functions** (per **ISO 13849-1:2023 (Edition 4)**) — emit only when ISO 13849-1 was selected in Round 3:

| Function ID | Description | PLr Target | Achieved Category (B / 1 / 2 / 3 / 4) | MTTFd per channel (years) | DCavg | CCF Score (Annex F) | SISTEMA Report Ref |
|---|---|---|---|---|---|---|---|
| SF-01 | {{DESCRIPTION}} | PL {{c/d/e}} | {{Cat}} | {{years}} | {{Low/Medium/High}} | {{≥65}} | {{SISTEMA-DOC}} |

### 11.2 E-Stop Architecture

E-stop zones and hardwired circuit design are documented in the Electrical Design Package. The safety PLC ({{SAFETY_PLC_MODEL}}) executes safety logic separate from the standard application.

**Safety task parameters:**
- Safety task period: {{PERIOD}} ms
- Safety task watchdog: {{WATCHDOG}} ms
- Safety signature: Generated at commissioning, documented in §11.4 SRS.

### 11.3 Safety Logic Rules

- **Safety tag prefix:** `SZ{{ZONE}}_` (HIS default convention; customer convention overrides if specified in Round 3).
- Standard tags shall NOT appear in safety routines (enforced by Studio 5000 / TIA Portal Safety scope rules).
- Safety logic changes require: safety task in Program mode + download + new safety signature + re-validation per §11.6 FSM.
- No bypass of safety functions without physical key switch + documented procedure per §11.7.

### 11.4 Safety Requirements Specification (SRS)

Per **IEC 61511-1:2016 §10**, the SRS is a separate, customer-approved deliverable.

| Field | Value |
|---|---|
| SRS Document Number | {{SRS_DOC_NUMBER}} |
| SRS Owner | {{SRS_OWNER}} |
| SRS Current Revision | {{SRS_REV}} |
| SRS Status | {{Draft / Approved / TBD}} |

If SRS = TBD, this FDS shall not be approved for safety design until the SRS is issued.

### 11.5 HAZOP / LOPA Reference

Per **IEC 61511-1:2016 §8** (hazard and risk analysis):

| Field | Value |
|---|---|
| HAZOP Document Number | {{HAZOP_DOC_NUMBER}} |
| HAZOP Date | {{HAZOP_DATE}} |
| LOPA Document Number | {{LOPA_DOC_NUMBER}} |
| LOPA Date | {{LOPA_DATE}} |
| Risk Reduction Target Source | {{HAZOP / LOPA / IPL Analysis}} |

### 11.6 Functional Safety Management (FSM) Plan

Per **IEC 61511-1:2016 §5**:

| Field | Value |
|---|---|
| FSM Plan Document | {{FSM_DOC_NUMBER}} |
| Functional Safety Manager (named person) | {{FSM_OWNER}} |
| Safety Review Cadence | {{Each rev / Quarterly / Annual}} |
| Safety Audit Plan | {{Internal / External / TÜV / Exida}} |
| Competence Management | {{HIS-internal training records / customer requirement}} |

### 11.7 Bypass Management

Per **IEC 61511-1:2016 §11.5**, bypass of any SIF is documented, time-limited, and tracked.

**Bypass authorization:**

| Bypass Type | Authorized By | Maximum Duration | Tracking Register |
|---|---|---|---|
| Maintenance bypass (key switch) | Maintenance supervisor + Engineer | Single shift | Maintenance log |
| Engineering bypass (HMI, logged) | Engineer | 8 hours | HMI Audit Trail + Bypass Register |
| Permanent change | MOC required (§1.6) | n/a — issue as configuration | Change Control |

**Bypass register columns:** Bypass ID / Equipment / Initiator / Reason / Start time / End time / Closed-by.

---

## 12. Performance Requirements

{{IF_PLATFORM_ROCKWELL}}
Basis: **Rockwell publication 1756-PM008** (Logix 5000 Controllers Design Considerations).

| Parameter | Requirement |
|---|---|
| PLC scan time — 10 ms task | ≤ 8 ms (80% of period) |
| PLC scan time — 50 ms task | ≤ 40 ms (80% of period) |
| Controller idle time (simplex) | ≥ 20% |
| Controller idle time (redundant) | ≥ 40% |

{{IF_PLATFORM_SIEMENS}}
Basis: Siemens Programming Style Guide (V1.6 or later) and S7-1500 hardware manual.

| Parameter | Requirement |
|---|---|
| OB1 cycle time | ≤ 80% of monitoring time |
| Cyclic interrupt OB load | ≤ 60% of period |
| CPU load (overall) | ≤ 80% |

{{IF_PLATFORM_BECKHOFF}}
Basis: TwinCAT 3 documentation (Beckhoff InfoSys).

| Parameter | Requirement |
|---|---|
| Task cycle time | ≤ 80% of period |
| TC3 RT-Util | ≤ 70% per core |

{{IF_PLATFORM_OTHER}}
Basis: {{platform-specific design guidance}}.

**Universal targets** (regardless of platform):

| Parameter | Requirement | Basis |
|---|---|---|
| HMI screen update rate | ≤ 1 second for process values | ANSI/ISA-101.01-2015 |
| Alarm response to display | ≤ 2 seconds from trip | ANSI/ISA-18.2-2016 |
| System availability target | {{XX}}% ({{HOURS}} downtime/year) | Customer requirement |
| Network latency (control VLAN) | ≤ 5 ms | EtherNet/IP design |
| Time-sync accuracy across all PLCs | ≤ {{MS}} ms | Section 6.2 NTP |

---

## 13. Backup, Historian, Spare-Parts, Diagnostic Logging

### 13.1 Backup & Restore Strategy

| Asset | Backup Frequency | Retention | Custodian | Restore Procedure |
|---|---|---|---|---|
| PLC ACD / project file | Weekly + on every download | 12 months versioned + permanent at FAT/SAT | {{HIS / Customer}} | {{describe — Studio 5000 download from versioned repository}} |
| HMI project (.APA / .gwbk / .MER) | Weekly + on every change | 12 months | {{HIS / Customer}} | {{describe}} |
| Drive parameters | After commissioning + on every parameter change | Permanent | HIS (via FactoryTalk AssetCentre or similar) | {{describe}} |
| Managed switch config | Weekly | 12 months | Customer IT | {{describe}} |
| FactoryTalk Security database | Daily | 90 days | Customer IT | {{describe}} |
| Historian database | Per IT policy | Per Section 13.2 | Customer IT | {{describe}} |

### 13.2 Historian Configuration

| Field | Value |
|---|---|
| Historian Platform | {{OSIsoft PI / FactoryTalk Historian / Ignition / Other}} |
| Tag List Source | {{Auto-discovery / Manual list / OPC-UA browse}} |
| Scan Rate | {{1 s / 5 s / per-tag}} |
| Compression | {{Swinging-door / Boxcar-back-slope / None}} |
| Retention Duration | {{Hot tier 90 days / Warm 1 yr / Cold 7 yr}} |
| Data Custodian | {{Customer IT}} |

### 13.3 Spare-Parts Strategy

Per `/bom` Section 9 (Spare Parts), one of:

- **Basic** — 1 spare per I/O card type; 1 set of fuses; 2 spare E-stops; 4 spare pilot lights. Adequate for OEM machines.
- **Enhanced** — Basic + 1 spare per critical-loop instrument + 4 extra terminal bases. Recommended for process plants and 24/7 operations.
- **SIL-graded** — Enhanced + 1 spare safety relay per redundancy group + 1 spare safety I/O per channel-bank-of-16. Required for SIL-rated systems.

Selected level for this project: {{LEVEL}}.

### 13.4 Diagnostic Logging Strategy

| Event Class | What Is Logged | Where | Retention |
|---|---|---|---|
| HMI login / logout | User, timestamp, station | FactoryTalk Diagnostics / Ignition Audit | 1 year |
| Tag write (operator-initiated) | User, tag, old value, new value, timestamp | HMI Audit Trail | 1 year |
| Alarm acknowledge | User, alarm, timestamp | HMI Audit Log | 1 year |
| PLC program download | User, controller, timestamp, signature | FactoryTalk AssetCentre or equivalent | Permanent |
| Safety signature change | User, controller, old signature, new signature | Safety Validation Record | Permanent |
| Bypass (override) | User, equipment, start, end, justification | Bypass Register §11.7 | Permanent |
| Network event | Switch port up/down, link change | Switch syslog | 90 days |

---

## 14. FAT / SAT Acceptance Criteria

The following criteria must be satisfied before FAT sign-off. The FAT procedure (separate document) shall contain a test case for each Req ID in §1.4 plus the universal items below.

### 14.1 Universal FAT Acceptance Criteria

| Criterion | Test Method | Pass Criteria |
|---|---|---|
| All I/O signals verified | Loop check with field simulation | Each signal matches PLC tag value within ±1% — see §14.5 Loop-Check Matrix |
| All operating modes functional | Exercise each mode per §8 | Mode transitions as specified |
| All alarms functional | Inject each alarm condition | Alarm appears within 2 s, correct class, correct message |
| All interlocks functional | Force each interlock condition | Equipment trips, safe state achieved, reset required |
| All safety functions verified | Per SIF test procedures (§11.4 SRS proof tests) | SIF trips at specified setpoint, safe state achieved |
| HMI navigation ≤ 3 clicks | Navigate to each equipment faceplate | No equipment requires more than 3 transitions |
| Network performance | Monitor CIP connections under load | No packet loss, scan times within §12 limits |
| Backup / restore tested | Restore from backup to spare hardware | System operates identically after restore |

### 14.2 SAT Delta from FAT

SAT re-executes a subset of FAT plus site-specific items: field-instrument calibration, field-wire continuity, end-to-end loops with real field devices, and any site-specific environmental conditions (haz-area, vibration, temperature).

| Criterion | At FAT | At SAT | Notes |
|---|---|---|---|
| I/O loop check (signal level) | ✓ (simulation) | ✓ (real field) | SAT verifies field calibration |
| Mode functional | ✓ | Re-verify | |
| Alarms | ✓ | Re-verify subset (sample-based) | |
| Performance | ✓ | Re-verify under real load | |
| Cybersecurity (SR controls) | Bench-tested | Penetration-tested at site | Per Section 6.3 |

### 14.3 Punch-List Handling

A Punch List of outstanding items at FAT is documented with item / owner / due date / status. Items rated Critical (safety, functional) must close before SAT begins. Items rated Cosmetic may close during SAT or as a post-commissioning action.

### 14.4 Performance Run

For process plants and continuous-operation systems, a Performance Run follows SAT:

| Phase | Duration | Success Criteria |
|---|---|---|
| First-product run | {{HOURS}} | Process produces on-spec product; no Critical alarms; mass-balance closes within {{X}}% |
| Extended run | {{HOURS / DAYS}} | System runs unattended through nominal cycle including overnight; no operator intervention beyond shift handover |
| Acceptance run | {{HOURS}} | Customer-witnessed; all Section 1.4 RTM Verified items closed |

### 14.5 Loop-Check Matrix

One row per analog loop and per critical discrete signal. Loops sign off as field wiring is energized.

| Tag | Description | Pre-Energization (continuity / megger) | Energized (24V / loop power) | Calibrated (LRV / URV) | Signed-off (Tech / Date) |
|---|---|---|---|---|---|
| {{TAG}} | {{DESCRIPTION}} | ☐ | ☐ | ☐ | |

---

## 15. Open Items and TBDs

Items that must be resolved before this FDS is approved for design. Each item must close (or be documented as accepted risk) before FAT.

| Item # | Section | Description | Owner (Customer / HIS) | Due Date | Status |
|---|---|---|---|---|---|
| OI-001 | §{{N}} | {{DESCRIPTION}} | {{OWNER}} | {{DATE}} | Open |

---

## 16. Glossary

| Term | Definition |
|---|---|
| ACD | Rockwell Studio 5000 project file format |
| ALCOA+ | Attributable, Legible, Contemporaneous, Original, Accurate + Complete, Consistent, Enduring, Available (FDA data integrity) |
| AOI | Add-On Instruction — reusable function block in Studio 5000 |
| AWIA | America's Water Infrastructure Act of 2018 |
| CIP | Common Industrial Protocol — EtherNet/IP transport layer |
| CRS | Cybersecurity Requirements Specification |
| DI / DO / AI / AO | Digital Input / Digital Output / Analog Input / Analog Output |
| DCavg | Average Diagnostic Coverage (ISO 13849-1) |
| E-Stop | Emergency Stop — hardwired safety circuit |
| FAT / SAT | Factory Acceptance Test / Site Acceptance Test |
| FDS | Functional Design Specification — this document |
| FSM | Functional Safety Management |
| GAMP 5 | Good Automated Manufacturing Practice, Edition 5 (ISPE) |
| HAZOP | Hazard and Operability Study |
| HMI | Human-Machine Interface |
| ISA-88 | ISA batch control standard (equipment hierarchy, procedural model) |
| ISA-18.2 | ISA alarm management standard |
| ISA-101 | ISA HMI design standard |
| LOPA | Layer of Protection Analysis |
| MOC | Management of Change |
| MTTFd | Mean Time To Dangerous Failure (ISO 13849-1) |
| NERC CIP | North American Electric Reliability Corporation Critical Infrastructure Protection standards |
| OSHA PSM | OSHA Process Safety Management (29 CFR 1910.119) |
| P&ID | Piping and Instrumentation Diagram |
| PFDavg | Average Probability of Failure on Demand (IEC 61511) |
| PFH | Probability of Failure per Hour (IEC 61511 high-demand mode) |
| PL | Performance Level (ISO 13849-1) |
| PLC | Programmable Logic Controller |
| RPI | Requested Packet Interval — EtherNet/IP I/O update rate |
| RRF | Risk Reduction Factor |
| RTM | Requirements Traceability Matrix |
| SCADA | Supervisory Control and Data Acquisition |
| SIF | Safety Instrumented Function |
| SIL | Safety Integrity Level (IEC 61511) |
| SR | System Requirement (IEC 62443-3-3) |
| SRS | Safety Requirements Specification |
| SuC | System Under Consideration (IEC 62443-3-2) |
| TBD | To Be Determined |
| UDT | User-Defined Tag (data type in Studio 5000) |
| VFD | Variable Frequency Drive |
| {{CUSTOMER_ACRONYM}} | {{DEFINITION}} |

(The skill's quality check auto-scans the generated document for capitalized abbreviations and ensures each appears here.)

---

## 17. Revision History

(Sole authoritative revision log. All MOC updates land here.)

| Rev | Date | Author | Description |
|---|---|---|---|
| {{REV}} | {{TODAY_DATE}} | T. Hunt | Initial draft for customer review |

---

## 18. Industry-Specific Compliance Overlay

{{Emit one or more sub-sections matched to the Round 1 industry answer. If industry = Other or no overlay applies, write: "Not applicable — no regulated-industry overlay required for this project type."}}

### 18.1 GAMP 5 & 21 CFR Part 11 Compliance (Pharma / Biotech)

**Software categorization** (per **GAMP 5 Second Edition**):

| Software Component | GAMP Category (1 / 3 / 4 / 5) | Validation Approach |
|---|---|---|
| Operating system | Cat 1 — Infrastructure | IT-managed; record version |
| PLC firmware | Cat 1 | Vendor-validated; record version |
| HMI runtime | Cat 3 — Non-configured | Vendor-validated; configuration only |
| HMI / SCADA configuration | Cat 4 — Configured | Validate configuration; not source code |
| PLC application code | Cat 5 — Custom | Full V-model validation: URS / FDS / DDS / IQ / OQ / PQ |

**21 CFR Part 11 audit trail:** Every electronic record modification carries Attributable / Legible / Contemporaneous / Original / Accurate (ALCOA) data integrity attributes plus Complete / Consistent / Enduring / Available (the "+"). HMI Audit Trail (per §13.4) must be tamper-evident, time-synchronized (per §6.2 NTP), and retained per the customer's Quality Manual.

**21 CFR Part 11 electronic signatures:** Required for any record that, in its paper form, would have required a handwritten signature. Implementation: {{Two-factor user/password challenge / Biometric / Token-based}}.

**EU Annex 11 parity:** {{Discuss how 21 CFR Part 11 implementation also satisfies EU Annex 11 §4 (validation), §7 (data storage), §9 (audit trails), §11 (periodic evaluation), §15 (change and configuration management).}}

**CSV lifecycle position:** This FDS sits between the URS ({{URS-DOC}}) and the Detailed Design Specification ({{DDS-DOC}}); IQ / OQ / PQ test packages reference Section 1.4 RTM Req IDs.

### 18.2 NERC CIP Compliance (Power Generation)

**BES Cyber System (BCS) categorization** per **NERC CIP-002-5.1a**: {{High Impact / Medium Impact / Low Impact}}.

| CIP Standard | Topic | This FDS Section |
|---|---|---|
| CIP-002 | BCS categorization | This subsection |
| CIP-003 | Security management controls | §6.3 + §13 |
| CIP-005 | Electronic Security Perimeter (ESP) | §6.3 Conduits |
| CIP-007 | System security management | §13.1 + §13.4 |
| CIP-008 | Incident response | §13.4 + customer plan |
| CIP-010 | Configuration change management | §1.6 MOC |

### 18.3 AWIA §2013 Risk & Resilience (Water / Wastewater)

**Population served:** {{POPULATION}}. Threshold for AWIA §2013 applicability is **3,300 served population**.

If population ≥ 3,300:
- Risk & Resilience Assessment (RRA) status: {{Current / Due / In progress}}.
- RRA cycle: every 5 years per AWIA §2013.
- Emergency Response Plan (ERP): {{Current / Due}}.
- This FDS contributes to the RRA's "automation, computer, or other systems" inventory and to the cybersecurity risk component.

### 18.4 OSHA PSM Compliance (Oil & Gas / Petrochemical / Chemical)

**Threshold quantity assessment:** {{Above / Below}} the 29 CFR 1910.119 Appendix A threshold for highly hazardous chemicals.

If above threshold:
- Process Hazard Analysis (PHA) document: {{PHA-DOC}}, due / current per 5-year cycle.
- Process Safety Information (PSI) — this FDS is part of PSI per §(d).
- Mechanical Integrity (MI) — referenced in §13 backup / spare-parts.
- Management of Change (MOC) — §1.6 of this FDS aligns with PSM §(l).
- Operator Training — §1.1 references; full training procedures separate.
- Pre-Startup Safety Review (PSSR) — performed before every initial / restart after major change; references this FDS plus FAT / SAT records.

### 18.5 NFPA 79 Machinery Compliance (Discrete Manufacturing)

**Machine description:** {{Brief description}}.

| NFPA 79 Topic | Section in This FDS / Other Doc |
|---|---|
| Wire identification (§13) | `/wire-diagram` output |
| Terminal block marking (§14) | `/wire-diagram` output |
| Disconnect / supply circuit (§5) | Electrical Design Package |
| Operator interface (§10) | §10.1 of this FDS |
| Stop functions (§9) | §11 of this FDS |
| Safeguarding (§9.4) | ISO 13849-1 SF-XX in §11.1 |
| Overcurrent protection (§7) | `/panel-layout` output |

### 18.6 Other Industry — Customer-Specified

{{If Round 1 industry = Other and customer specified standards in Round 3: emit a subsection citing each customer standard, its rev, and the FDS section it constrains.}}

---

*End of Functional Design Specification*

*This document shall not be used for construction or programming until all approval signatures in the Approval Signatures table are obtained. After approval, any change to this document requires a Management of Change (MOC) record per §1.6 with re-approval.*

*Hunt Integrative Solutions LLC — Confidential — Prepared for {{CUSTOMER_NAME}}*
````

---

### Step 3.5 — Mandatory Quality Check Before Saving

Run all rules; report results in the summary block at the end of generation.

1. **RTM coverage.** Every Section 1.2 In-Scope item has at least one corresponding Req ID row in §1.4 with non-blank FDS Section reference.
2. **Equipment / I/O reconciliation.** Section 4 row count vs. Section 5 totals are consistent (within ±10%; if larger, a `[VERIFY]` flag is emitted).
3. **Mode-list coverage.** Every operating mode selected in Round 2 Q4 appears as a row in Section 3.3.
4. **Interlock / alarm reconciliation.** Every interlock in §7.4 has a matching alarm row in §9.2.
5. **SIF column completeness.** Every SIF row in §11.1 has a non-blank Mode, SIL Target, RRF Target, Architecture, PFDavg/PFH, Proof Test Interval, SRS Ref, HAZOP/LOPA Ref. (Or the project has SIL = None per Round 3 and §11.1 reads "Not applicable.")
6. **ISO 13849-1 column completeness** (only if Round 3 selected ISO 13849-1): every machinery-safety row has PLr, Category, MTTFd, DCavg, CCF Score, SISTEMA Ref.
7. **Nexus / TBD migration.** Every `[ASSUMED from Nexus — verify with customer]` and every `[TBD — REQUIRES CUSTOMER INPUT]` placeholder appears as a row in §15 Open Items.
8. **Conditional sections.** §8.6 (Phase State) emitted iff batch / sequential operations selected. §8.7 (Recipe) emitted iff recipe in scope (Round 2 follow-up). §18 emits exactly the sub-sections matching the Round 1 industry answer.
9. **Standards-citation hygiene.** Every standard in §2 cites year/edition (ANSI/ISA-5.06.01-2007, IEC 61511-1:2016, ISO 13849-1:2023, etc.) — no bare "ISA-18.2" or "IEC 61511."
10. **Platform-conditional consistency.** Section 7.5, 8.1, 12 emit only the block matching the Round 2 platform answer — no Rockwell-specific text in a Siemens FDS.
11. **62443 zone targets.** Section 6.3 SL-T values are either populated from a customer risk assessment, marked TBD with risk assessment as deliverable, or carry the `[HIS DEFAULT — RECONCILE]` flag — never silently prescribed.
12. **Glossary completeness.** Every capitalized abbreviation appearing in the document body has a row in §16. Auto-scan and report missing terms.
13. **Document Owner / Custodian / FAT date / Rev.** Header block has no `{{...}}` placeholders left after substitution.

Report:

```
FDS Quality Check
─────────────────────────────────
Total RTM Req IDs:                ###
Open Items:                       ###
Nexus-ASSUMED rows in body:       ###  ← must each appear in §15
TBD placeholders in body:         ###  ← must each appear in §15
SIF rows missing required cols:   0    ← or list rows
ISO 13849-1 rows missing cols:    0    ← or list rows
Modes in Round 2 not in §3.3:     0    ← coverage gap
Interlocks without alarm match:   0    ← §7.4 vs §9.2
Standards missing year/edition:   0    ← list rows
Platform leakage (wrong block):   0    ← Rockwell text in Siemens FDS, etc.
62443 zones silently prescribed:  0    ← must be customer / TBD / HIS-default-flagged
Glossary terms missing:           0    ← list any unhandled abbreviations
Header placeholders unresolved:   0    ← must be 0 before save
```

---

### Step 4 — Save and Report

If outputs from `/io-list`, `/bom`, `/panel-layout`, `/wire-diagram`, `/soo`, `/prog-arch`, `/hmi-spec`, or `/alarm-philosophy` exist in the same save folder, list them in §2 Reference Documents and cross-reference them throughout the body where applicable.

1. **Always ask where to save the document — never silently default to Desktop.** Use `AskUserQuestion`:

   ```
   AskUserQuestion([
     {
       question: "Where should the FDS document be saved?",
       header: "Save Location",
       multiSelect: false,
       options: [
         { label: "Active project folder — I'll type the path (Recommended)", description: "WSL path. Recommended so the FDS lives next to the BOM, I/O list, panel layout, etc." },
         { label: "Same folder as other project documents — I'll type the path", description: "WSL path." },
         { label: "Current working directory", description: "Use the current shell working directory." },
         { label: "Desktop", description: "/mnt/c/Users/HIS Controls/Desktop/ (Windows Desktop via WSL — use only for one-off / scratch documents)." }
       ]
     }
   ])
   ```

2. Save the completed FDS using a status-aware filename:
   - `Status = DRAFT (Rev = Draft)` → `FDS-{{PROJECT_SLUG}}-Draft.md`
   - `Status = DRAFT (Rev = A/B/C/...)` → `FDS-{{PROJECT_SLUG}}-Rev{{REV}}-DRAFT.md`
   - `Status = FOR REVIEW` → `FDS-{{PROJECT_SLUG}}-Rev{{REV}}-FOR-REVIEW.md`
   - `Status = APPROVED` → `FDS-{{PROJECT_SLUG}}-Rev{{REV}}-APPROVED.md`

3. Use `AskUserQuestion` for the completion summary:

   ```
   AskUserQuestion([
     {
       question: "The FDS draft has been saved. What would you like to do next?",
       header: "Next Step",
       multiSelect: false,
       options: [
         { label: "Walk through Open Items (§15) one at a time", description: "I'll resolve each TBD via AskUserQuestion." },
         { label: "Resolve a specific section — I'll tell you which", description: "Targeted section completion." },
         { label: "Customer-send-ready — finalize at the chosen Rev", description: "Switch status to FOR REVIEW; rename the file accordingly." },
         { label: "Add more equipment / scope — continue the interview", description: "Append to existing draft." }
       ]
     }
   ])
   ```

4. After the user responds, either walk Open Items, resolve targeted sections, or rename to FOR REVIEW status. Always remind: **The FDS is not approved for design until all Approval Signatures are obtained. Treat the draft as a working document until then.**

5. Final plain-text report:
   - File path (full WSL path)
   - Quality Check summary block (Step 3.5)
   - Counts: RTM Req IDs / Open Items / Nexus-ASSUMED / TBD
   - Recommended next steps: "Customer review → resolve Open Items → confirm Nexus assumptions → status change to FOR REVIEW → customer signatures → status change to APPROVED → begin I/O Mapping (`/io-list`)"
