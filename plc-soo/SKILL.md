---
name: plc-soo
description: Generate a Sequence of Operations (SOO) document for industrial automation. Multi-audience output (operator at 3 AM / maintenance technician / engineer at FAT / OSHA PSM auditor) with Quick-Reference Card, Cause & Effect Matrix, Cold-Start vs Hot-Restart sequences, Setpoint Authorization Table, Interlock Test Procedures, Partial-Failure Operation, LOTO Points, Operator Skills Checklist, Measurement Points and PM Schedule for maintenance, conditional regulated-industry overlays (OSHA PSM §(f) / 21 CFR Part 11 / AWIA §2013 / FSMA / NERC CIP / NFPA 79), platform-conditional content for Rockwell / Siemens / Beckhoff / other, and RTM linkage to FDS. Follows ANSI/ISA-88.01-2010, ANSI/ISA-5.06.01-2007, ANSI/ISA-18.2-2016, IEC 62061:2021, ISO 13849-1:2023, NFPA 79-2024, OSHA 1910.119, OSHA 1910.147 / ANSI Z244.1, and EEMUA 191:2013.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [soo, sequence-of-operations, plc, industrial, isa-88, isa-5.06, his]
    ported_from: claude-code/soo
---
# /soo — Sequence of Operations Generator

**Invocation:** `/soo [equipment or system name]`

**Examples:**
- `/soo "Cooling Tower Fan VFD — Area 200"`
- `/soo "Wastewater Lift Station No. 4"`
- `/soo "Reactor Feed Pump P-101"`

---

## Purpose

Generate a complete Sequence of Operations (SOO) — the multi-audience reference for an industrial process. The same document serves: the operator at 3 AM trying to restart a tripped pump, the maintenance technician troubleshooting under a fault, the engineer verifying interlocks at FAT, the OSHA PSM auditor reviewing § (f) operating procedures, the customer's process-safety lead at handover. Every step is numbered, every condition is spelled out, every fault has a defined response, every interlock has a test procedure, every safety-energy source has an LOTO point.

Standards alignment (specific editions matter — reviewers check first):

- **ANSI/ISA-88.01-2010** — Procedural control: state machine, phase transitions, hold/abort paths
- **ANSI/ISA-5.06.01-2007** — Functional requirements documentation; Cause & Effect Matrix per §5.7
- **ANSI/ISA-18.2-2016** — Alarm rationalization integrated into fault response
- **IEC 62061:2021 (Edition 2)** — Functional safety of machinery (where applicable)
- **ISO 13849-1:2023 (Edition 4)** — Performance Levels for safety-related parts of control systems (where applicable)
- **IEC 61511-1:2016 (Edition 2)** — SIS for the process industry (where SIL functions are present)
- **NFPA 79-2024** — Electrical standard for industrial machinery (operator interface and E-stop requirements)
- **OSHA 29 CFR 1910.119** — Process Safety Management; § (f) requires written operating procedures (this SOO **is** that deliverable for PSM-covered processes)
- **OSHA 29 CFR 1910.147** + **ANSI/ASSP Z244.1-2016** — Lockout / Tagout (the LOTO Points table in §13.3 is the operational instantiation)
- **EEMUA 191:2013 (Edition 3)** — Alarm management practice (paired with ISA-18.2)
- Industry-specific standards layered into Section 16 by industry answer.

---

## Instructions

### MANDATORY: AskUserQuestion for ALL Questions

Never ask in plain text. Every question goes through `AskUserQuestion`. **This is a formal interview, not a chat.** Never assume a value the user did not provide. Defaults are only acceptable when explicitly labeled `(Recommended)` inside an option, never silently in code.

The interview runs in **three sequential rounds plus targeted follow-ups**. Complete each round before proceeding.

---

### Step 1 — Conduct the Project Interview

#### Interview Round 1 — Identity, Scope, Industry, Nexus (ask all 4 together)

```
AskUserQuestion([
  {
    question: "What is the equipment or system name, and what is the HIS project number?",
    header: "Identity",
    multiSelect: false,
    options: [{ label: "I'll type it", description: "Use Other for equipment name + HIS project number." }]
  },
  {
    question: "What is the scope of this SOO?",
    header: "SOO Scope",
    multiSelect: false,
    options: [
      { label: "Single device — one motor, pump, valve, VFD, blower, conveyor", description: "Single-equipment SOO; partial-failure not applicable." },
      { label: "Equipment group — set of related devices (pump + isolation valves + relief)", description: "Cross-equipment interlocks; partial-failure may apply." },
      { label: "Full system / area — entire process area with multiple units", description: "Lift station / reactor skid / packaging line; partial-failure required." }
    ]
  },
  {
    question: "What industry / regulatory environment governs this equipment?",
    header: "Industry",
    multiSelect: false,
    options: [
      { label: "Water / Wastewater (AWIA / EPA / 10 States)", description: "AWIA §2013 ERP-coordination overlay required if population ≥ 3,300." },
      { label: "Food and Beverage (FSMA / 3-A)", description: "Cleaning records + allergen change-over." },
      { label: "Oil and Gas / Petrochemical (OSHA PSM / API)", description: "OSHA 1910.119 § (f) operating procedures overlay." },
      { label: "Pharmaceutical / Biotech (FDA / EU GMP)", description: "21 CFR Part 11 + ALCOA+ + EU Annex 11 overlay." },
      { label: "Manufacturing / Discrete (NFPA 79 / OSHA)", description: "NFPA 79 operator-interface + stop-function references." },
      { label: "Power / Utilities (NERC CIP)", description: "CIP-008 incident response overlay." },
      { label: "Chemical Processing (OSHA PSM / CCPS)", description: "OSHA PSM + CCPS guidelines." },
      { label: "Other (I'll describe it)", description: "Industry overlay marked Not Applicable unless additional regulatory overlay identified." }
    ]
  },
  {
    question: "Is this equipment indexed in the NEXUS knowledge graph?",
    header: "Nexus Data",
    multiSelect: false,
    options: [
      { label: "Yes — pull existing PLC/HMI data from Nexus", description: "Pre-populate from Nexus, all flagged [ASSUMED from Nexus — verify with customer]." },
      { label: "No — this is new, nothing in Nexus yet", description: "Build template from interview only." },
      { label: "Not sure — check Nexus and tell me what you find", description: "Run plant_summary + list_documented_plcs and surface results." }
    ]
  }
])
```

#### Round 1 Follow-Up — Document Revision and Schedule

Immediately after Round 1, ask:

```
AskUserQuestion([
  {
    question: "Document revision letter for this issue?",
    header: "Doc Rev",
    multiSelect: false,
    options: [
      { label: "A — first issue (Recommended for new projects)", description: "Use Rev A for first issue; increment after every change." },
      { label: "B / C / D / later — continuing an existing SOO", description: "I'll specify in Other." },
      { label: "Draft / pre-revision", description: "Internal working copy; do not issue." }
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

#### Interview Round 2 — Process, Modes, Platform, Audience (ask all 4 together)

After Round 1:

```
AskUserQuestion([
  {
    question: "Describe what this equipment does — walk through a normal cycle from off to running. Include: what enters, what transformations happen, what exits. 2–3 paragraphs of plain English.",
    header: "Process Description",
    multiSelect: false,
    options: [{ label: "I'll describe it", description: "Use Other; 2–3 paragraphs of plain-English process narrative." }]
  },
  {
    question: "What are the operating modes for this equipment?",
    header: "Operating Modes",
    multiSelect: true,
    options: [
      { label: "Auto (PLC sequence-driven)", description: "" },
      { label: "Manual (operator-initiated, HMI / local)", description: "" },
      { label: "Step / Jog (maintenance/commissioning inch mode)", description: "" },
      { label: "Hand (hardwired local control, bypasses PLC)", description: "" },
      { label: "Clean-in-Place / CIP (food/pharma)", description: "Triggers CIP cycle sub-template in §7 / §9." },
      { label: "Simulation / Test (offline)", description: "" },
      { label: "Other mode (I'll describe)", description: "" }
    ]
  },
  {
    question: "What control platform is this system running on?",
    header: "Control Platform",
    multiSelect: false,
    options: [
      { label: "Rockwell ControlLogix / CompactLogix / GuardLogix (Studio 5000) (Recommended for HIS-default)", description: "Phase Manager + Studio 5000 tag examples." },
      { label: "Rockwell PLC-5 or SLC 500 (legacy)", description: "Migration project; legacy file/N-file addressing." },
      { label: "Siemens S7-300 / S7-400 / S7-1500 (TIA Portal)", description: "S88 state machine in TIA Portal; Siemens raw-address syntax." },
      { label: "Beckhoff TwinCAT", description: "PackML state machine in TwinCAT." },
      { label: "Ignition SCADA (soft PLC or OPC-UA client)", description: "" },
      { label: "Allen-Bradley MicroLogix", description: "" },
      { label: "Other / Mixed (I'll describe)", description: "" }
    ]
  },
  {
    question: "Who are the primary users of this SOO document?",
    header: "Target Audience",
    multiSelect: true,
    options: [
      { label: "Operators (running the process day-to-day)", description: "Drives Quick-Reference Card and Steady-State table emphasis." },
      { label: "Maintenance technicians (troubleshooting)", description: "Drives LOTO Points, Measurement Points, Decision Trees, PM Schedule." },
      { label: "Engineers (design review and commissioning)", description: "Drives C&E Matrix, Interlock Test Procedures, timing/state diagrams." },
      { label: "Customer / Owner (approval and handover)", description: "Drives Skills-Verified Checklist." },
      { label: "Regulatory / Auditor (compliance verification)", description: "Drives Industry Overlay (§16) emphasis." }
    ]
  }
])
```

#### Round 2 Follow-Ups (targeted, conditional)

- **If scope = Single device or Equipment group:** ask "What equipment-type pre-start checks apply? (motor / pump / fan / VFD / heat exchanger / reactor / batch vessel / conveyor / other)" — used in §7.1 to emit a tailored Pre-Start Checklist.
- **If scope = Equipment group or Full system:** ask "Are there parallel or redundant equipment groups (e.g., 3 pumps in parallel)? If yes, list each redundancy group" — used in §8.6 Partial-Failure Operation.
- **If modes include CIP:** ask "What CIP cycle steps and durations are required? (rinse / wash / sanitize / final rinse / verification)" — used in §7 / §9 CIP sub-template.
- **For every project:** ask "Are Cold-Start (full-outage recovery) and Hot-Restart (after-trip-during-shift) sequences distinct for this equipment?" — used in §7.2.1 / §7.2.2.

---

#### Interview Round 3 — Faults, Safety, Setpoints, Customer Standards (ask all 4 together)

After Round 2:

```
AskUserQuestion([
  {
    question: "List every fault condition this equipment can experience. For each: cause / symptom / current operator response. If you don't know all yet, list the ones you do.",
    header: "Fault Conditions",
    multiSelect: false,
    options: [{ label: "I'll list them", description: "Use Other; one fault per line." }]
  },
  {
    question: "Are there safety-rated functions on this equipment?",
    header: "Safety Functions",
    multiSelect: false,
    options: [
      { label: "Yes — SIL-rated (IEC 61511) functions present", description: "Triggers proof-test placeholder in §11; SIL columns required." },
      { label: "Yes — PL-rated (ISO 13849-1) machinery functions present", description: "PLr / Category / MTTFd / DCavg required in §11." },
      { label: "Yes — E-stop / light curtain / interlock only (no SIL/PL calc)", description: "Process-safety only; no quantitative verification required." },
      { label: "No — process interlocks only, no certified safety functions", description: "All safety items in §6.2 are process interlocks." },
      { label: "Not determined yet — TBD during hazard analysis", description: "Mark §11 TBD; HAZOP/LOPA reference required before approval." }
    ]
  },
  {
    question: "List EVERY setpoint, timer, and limit. REQUIRED (provide a value or TBD for each): Restart Lockout (s), Minimum Run Time (min), Startup Sequence Timeout (s), Valve Open Timeout (s), Motor Run-Feedback Timeout (s), Pressure Build-Up Timeout (s), HMI Watchdog Timeout (s), Motor Cooling Minimum (min for restart-after-overload), Suction-Valve Idle Hours. PLUS process-specific: max pressure, low flow, level setpoints, etc.",
    header: "Setpoints / Timers / Limits",
    multiSelect: false,
    options: [{ label: "I'll list them", description: "Use Other; one value per line in format `Name | Value | Units | Source`. Quality check enforces no bare {X} remains." }]
  },
  {
    question: "Customer-specific standards, existing SOO documents, or narrative descriptions to align to?",
    header: "Customer Standards",
    multiSelect: false,
    options: [
      { label: "Yes — I'll describe / provide a file", description: "Use Other." },
      { label: "No — use HIS and ISA standards", description: "" },
      { label: "Not sure — use ISA standards as baseline", description: "" }
    ]
  }
])
```

#### Conditional Follow-Up Questions (trigger based on Round 1–3 answers)

- **Scope = Full system / area:** "List every major equipment item with tag and function. One per line."
- **Industry = Pharma:** "Is this system subject to FDA 21 CFR Part 11, GAMP 5, or EU Annex 11 electronic records requirements?"
- **Industry = Oil & Gas / Chemical:** "ATEX/NEC hazardous-area classification (Class I Div 1/2, Zone 0/1/2)? OSHA PSM 1910.119 applicability (threshold quantity)?"
- **Industry = Power:** "NERC CIP applicability (BES Cyber System category — High / Medium / Low Impact)?"
- **Industry = Water:** "Population served (drives AWIA §2013 — threshold 3,300)?"
- **Safety = SIL-rated:** "Target SIL (1 / 2 / 3)? Has LOPA / risk assessment been completed? SRS document number?"
- **Safety = PL-rated:** "Target PLr (c / d / e)? SISTEMA report current?"
- **Nexus = Not sure:** Run `mcp__nexus__plant_summary` and `mcp__nexus__list_documented_plcs`; report what you find.
- **Platform = Rockwell PLC-5 or SLC 500:** "Migration project? Target platform (ControlLogix / CompactLogix)?"

---

### Step 2 — Pull Nexus Data (If Available)

If Nexus data exists for this equipment, run these tools and extract relevant facts. **Do not present raw Nexus output to the user.** Tag everything for inclusion as `[ASSUMED from Nexus — verify with customer]`.

| Tool | Purpose |
|---|---|
| `mcp__nexus__generate_sequence_of_operations` | Pull existing SOO narrative from rung logic |
| `mcp__nexus__find_interlocks` | Extract interlock chains for the equipment |
| `mcp__nexus__find_area_interlocks` | Capture area-level interlocks affecting this equipment |
| `mcp__nexus__tag_search` | Find run commands, feedback, faults, status tags |
| `mcp__nexus__tag_context` | Get description and wiring context for key tags |
| `mcp__nexus__tag_full_chain` | Trace tag from input through logic to output |
| `mcp__nexus__classify_signal_roles` | Identify commands / feedbacks / fault inputs |
| `mcp__nexus__generate_io_inventory` | Pull I/O list for equipment |
| `mcp__nexus__plant_summary` | Plant-level structure |
| `mcp__nexus__list_documented_plcs` | Confirm PLC list |

**Nexus Skepticism Rules — Non-Negotiable:**

- Every Nexus-sourced field is flagged `[ASSUMED from Nexus — verify with customer]`.
- Interlock chains and permissives from Nexus reflect code, which may not reflect current field wiring or customer intent.
- Tag descriptions from Nexus may be auto-generated or inherited from legacy programs.
- Setpoints in Nexus reflect what is currently programmed, not necessarily correct.
- **Nexus data CANNOT be used to define safety function response paths** — those must come from customer or completed LOPA/risk assessment.

After collecting Nexus data, present a summary via `AskUserQuestion` asking which assumptions to accept, reject, or mark TBD.

---

### Step 3 — Research Applicable Standards

Use Scrapling to research standards before generating. Capture the specific clause that supports each requirement.

**Mandatory research (run regardless of equipment type):**

1. ANSI/ISA-88.01-2010 procedural state model — states, transitions, commands, hold/abort paths.
2. ANSI/ISA-18.2-2016 alarm rationalization — priority definitions for fault response table.

**Industry-specific research:**

| Industry | Research Targets |
|---|---|
| Water / Wastewater | AWWA Manual M2; 10 States Standards (pump station design); EPA wet-well management |
| Food & Beverage | 3-A Sanitary Standards; EHEDG cleaning sequence; FSMA 21 CFR 117 preventive controls |
| Oil & Gas / Petrochemical | API RP 554; IEC 61511-1:2016 SIS procedural requirements; OSHA 1910.119 § (f); ISA-84 |
| Pharma / Biotech | GAMP 5 Second Edition; FDA 21 CFR Part 11 § 11.10; EU Annex 11 § 4; USP <1058> |
| Manufacturing / Discrete | NFPA 79-2024 § 13; ISO 13849-1:2023 §§ 4–6 PL determination; IEC 62061:2021 |
| Power / Utilities | NERC CIP-002 / 005 / 007 / 010 / 008; IEEE C37.90 |
| Chemical | OSHA PSM (same as oil & gas); CCPS Guidelines for Safe and Reliable Instrumented Protective Systems |

**Equipment-type-specific research** (centrifugal pump startup, VFD motor sequencing, batch reactor control, belt conveyor sequencing, etc.): manufacturer-recommended startup/shutdown procedures, common failure modes, industry guidelines for minimum run times and restart delays.

---

### Step 4 — Generate the SOO Document

Write the complete SOO using the template below. Every section is mandatory. Use `[TBD — see Open Items]` for missing data; never leave a section blank or invent values.

**Formatting rules:**
- Sequences are numbered lists, never bullets.
- Plain English only. No PLC instruction names (XIC, OTE, TON) in the narrative — those go in the code.
- Conditions spelled out in full: "Wait until discharge pressure rises above 20 PSI as indicated on PIT-101 before proceeding to Step 6" — not "wait for pressure permissive."
- Every setpoint appears once in §6.5 with tag, units, source. Other sections reference §6.5 by tag, not by re-stating the value.
- Every fault in §11 maps to exactly one alarm in §12.
- Platform-conditional content uses `{{IF_PLATFORM_ROCKWELL}}` / `{{IF_PLATFORM_SIEMENS}}` / `{{IF_PLATFORM_BECKHOFF}}` / `{{IF_PLATFORM_OTHER}}` blocks; emit only the matching block.
- Industry-conditional: §16 emits only the sub-section matching Round 1 industry.

---

## SOO Document Template

````markdown
# Sequence of Operations
## {Equipment / System Name}
### {HIS Project Number} | Rev {rev} — {Status: DRAFT / FOR REVIEW / APPROVED}

---

| Field | Value |
|---|---|
| Document Number | SOO-{project-slug}-{equipment-slug} |
| Revision | {rev} |
| Status | {DOC_STATUS} |
| Prepared By | Timothy Hunt, Hunt Integrative Solutions LLC |
| Preparation Date | {date} |
| Customer | {customer name} |
| Site | {site location} |
| Industry | {industry from Round 1} |
| Required-on-site / FAT Date | {fat_date} |
| Customer Approval | __________________ Date: ________ |
| HIS Approval | __________________ Date: ________ |

---

## Table of Contents

1. Purpose and Scope (1.1 Purpose · 1.2 Scope · 1.3 System Boundary · 1.4 Traceability to FDS)
2. Reference Documents
3. Definitions and Abbreviations
4. Equipment Description (4.1 Equipment List · 4.2 System Description · 4.3 Process Visual)
5. Operating Modes
6. Permissives, Interlocks, Safety, C&E Matrix, Setpoints (6.1 Permissives · 6.2 Run Interlocks · 6.3 Safety Functions · 6.4 Cause & Effect Matrix · 6.5 Setpoint Authorization Table)
7. Startup Sequence (7.1 Pre-Start Checks · 7.2.1 Cold-Start Auto · 7.2.2 Hot-Restart Auto · 7.3 Manual Mode · 7.4 Step / Jog Mode · 7.5 Timing Diagram · 7.6 CIP Cycle Sub-Sequence)
8. Normal Operation (8.1 Steady-State Baseline · 8.2 Control Loops · 8.3 Continuous Monitoring · 8.4 Minimum Run Time · 8.5 Restart Lockout · 8.6 Partial-Failure Operation)
9. Normal Shutdown (9.1 Auto · 9.2 Manual · 9.3 CIP Cycle End)
10. Emergency Shutdown / E-Stop / Power Loss (10.1 E-Stop · 10.2 Power Loss · 10.3 HMI Loss vs PLC Loss · 10.4 Network Partition Recovery)
11. Fault Response (11.1 Fault Response Table · 11.2 Interlock Test Procedures · 11.3 Troubleshooting Decision Trees)
12. Alarm Summary
13. Operator and Maintenance Reference (13.1 HMI Faceplate · 13.2 Operator Responsibilities · 13.3 LOTO Points · 13.4 Operator Skills Verification · 13.5 Measurement Points · 13.6 Preventive Maintenance Schedule)
14. Open Items and TBDs
15. Glossary
16. Industry-Specific Operating Procedures Overlay (conditional)
17. Revision History

---

## Operator Quick-Reference Card

> Designed to be printed and laminated. Three numbers operators must remember at a glance.

**Three Numbers:** Max Pressure: {max_pressure} {units} | Min Flow: {min_flow} {units} | Max Temp: {max_temp} {units}

**Cold-Start (4 steps):**
1. Verify pre-start checklist complete (§7.1).
2. Select Auto mode on HMI faceplate.
3. Confirm all permissives green (§6.1).
4. Press Start. Monitor for "Running" status within {startup_timeout} s.

**Normal Stop (4 steps):**
1. Press Stop on HMI faceplate.
2. Confirm Min Run Time {min_run_time} min has elapsed.
3. Verify motor de-energizes within {feedback_timeout} s.
4. Verify "Stopped" status on HMI.

**Emergency Stop (3 steps):**
1. Press nearest E-stop button. All outputs de-energize immediately.
2. Acknowledge E-stop alarm on HMI.
3. Reset only after cause is cleared and area is verified clear.

**Top-5 Faults — Single-Sentence Response:**
| Fault | Immediate Action |
|---|---|
| {top-fault-1} | {one-sentence response} |
| {top-fault-2} | {one-sentence response} |
| {top-fault-3} | {one-sentence response} |
| {top-fault-4} | {one-sentence response} |
| {top-fault-5} | {one-sentence response} |

**Escalation:** If unable to resolve, contact {customer maintenance contact}. Do not bypass safety functions.

---

## Revision History

| Rev | Date | Author | Description |
|---|---|---|---|
| {rev} | {date} | T. Hunt | Initial draft — for customer review |

---

## 1. Purpose and Scope

### 1.1 Purpose

This document describes the Sequence of Operations for {equipment/system name}. It defines how the equipment starts (Cold and Hot), operates normally, shuts down (Normal and Emergency), and responds to fault conditions. The document serves as the reference for operator training, maintenance troubleshooting, FAT/SAT verification, and regulatory audit.

### 1.2 Scope

**In scope:** {what this SOO covers — specific equipment, control boundaries, modes.}

**Out of scope:** {what is NOT covered — adjacent systems, utility supplies, upstream/downstream not under this control system's authority.}

### 1.3 System Boundary

{Where does control authority start and stop? Upstream/downstream interfaces? Physical boundary (process connection points) and control boundary (what the PLC commands vs. only monitors).}

### 1.4 Traceability to Functional Design Specification

If `FDS-*-Rev*.md` exists in the save folder, parse §1.4 RTM and list every Req ID in scope of this SOO with back-reference:

| FDS Req ID | FDS Section | This SOO Section | Status |
|---|---|---|---|
| REQ-FUNCTIONAL-014 | FDS §3.3 Operating Modes | SOO §5 + §7.2.1 | Open / Verified |
| REQ-SAFETY-007 | FDS §11.1 SIF-01 | SOO §6.3 + §11.2 | Open / Verified |
| {{REQ-XXX}} | {{FDS §}} | {{SOO §}} | Open |

If no FDS is found in the folder, mark this section "Not applicable — no parent FDS in this project folder. SOO is freestanding."

---

## 2. Reference Documents

| Document | Number / Revision | Description |
|---|---|---|
| P&ID | {P&ID number and revision} | Process and Instrumentation Diagram |
| Electrical Drawings | {drawing number} | Panel layout, wiring diagrams |
| Functional Design Specification | FDS-{slug} | System-level functional requirements (parent of this SOO) |
| I/O List | IOL-{slug} | Complete field device list (`/io-list` output) |
| BOM | BOM-{slug} | Hardware bill of materials (`/bom` output) |
| Alarm Philosophy | ALARM-PHIL-{slug} | Master alarm rationalization register (`/alarm-philosophy` output) |
| HMI Spec | HMS-{slug} | HMI screen specification (`/hmi-spec` output) |
| ANSI/ISA-88.01-2010 | — | Batch control: procedural model |
| ANSI/ISA-5.06.01-2007 | — | Functional Requirements Documentation; Cause & Effect per §5.7 |
| ANSI/ISA-18.2-2016 | — | Management of alarm systems |
| EEMUA 191:2013 (Edition 3) | — | Alarm Systems: Design, Management and Procurement |
| OSHA 29 CFR 1910.147 | — | Lockout/Tagout (universal — see §13.3) |
| ANSI/ASSP Z244.1-2016 | — | Lockout/Tagout (LOTO complement to OSHA) |
| OSHA 29 CFR 1910.119 | — | Process Safety Management § (f) operating procedures (oil & gas / chemical only) |
| IEC 61511-1:2016 (Edition 2) | — | SIS for the process industry (if SIL-rated functions in scope) |
| IEC 62061:2021 (Edition 2) | — | Functional safety of machinery (if applicable) |
| ISO 13849-1:2023 (Edition 4) | — | PL determination for machinery (if applicable) |
| NFPA 79-2024 | — | Industrial machinery (if applicable) |
| {Industry standard} | {number} | {GAMP 5 / AWIA §2013 / NERC CIP / 3-A / etc., per §16} |
| {Manufacturer manual} | {document number} | {equipment description} |

---

## 3. Definitions and Abbreviations

| Term | Definition |
|---|---|
| Auto Mode | Equipment is controlled automatically by the PLC based on process conditions or supervisory command |
| Manual Mode | Operator initiates start/stop directly from HMI or local panel; PLC interlocks remain active |
| Cold Start | First start of day after a full outage; motor cold, lines unprimed, all valves at fail-state |
| Hot Restart | Restart after a within-shift trip; motor warm, lines may be primed, valves per pre-trip state |
| Permissive | Condition that must be TRUE for equipment to start or continue running |
| Interlock | Condition that, when it occurs, causes equipment to stop (trip) or prevents start |
| Fault | Abnormal condition detected by control system requiring operator attention or automatic response |
| C&E Matrix | Cause and Effect Matrix per ISA-5.06.01 §5.7 |
| LOTO | Lockout / Tagout per OSHA 29 CFR 1910.147 + ANSI/ASSP Z244.1-2016 |
| PV / SP | Process Variable / Setpoint |
| HMI | Human-Machine Interface |
| SIS | Safety Instrumented System (separate from BPCS) |
| E-Stop | Emergency Stop — hardwired safety device |
| RTM | Requirements Traceability Matrix (in parent FDS §1.4) |
| Tag Naming Convention | {Pulled from `/fds` §7.6 if present, or platform-default} |

---

## 4. Equipment Description

### 4.1 Equipment List

For single device:

| Tag | Description | Type | Location | Signal Type | Notes |
|---|---|---|---|---|---|
| {tag} | {description} | {pump/motor/valve/etc.} | {physical location} | {DI/DO/AI/AO/VFD} | |

For full system / area, group equipment by unit. For each unit:

**Unit: {Unit Name}**
| Tag | Description | Type | Location | Signal Type | Notes |
|---|---|---|---|---|---|

### 4.2 System Description

{2–4 paragraph plain-English description: what this equipment does, its role in the process, how it connects to upstream and downstream systems. No PLC jargon.}

### 4.3 Process Visual

Auto-generated from interview equipment list. For pump:

```
[Suction Source] ─→ [XV-100] ─→ [Pump P-101] ─→ [XV-101] ─→ [Discharge / Process]
                       ↑              ↑               ↑
                  open feedback   run feedback    open feedback
                                        ↑
                                    [PIT-101 pressure]
                                    [TT-101 temp]
                                    [VIB-101 vibration]
```

For multi-equipment / parallel: simple block diagram with arrows showing flow.

---

## 5. Operating Modes

| Mode | Description | Who Can Initiate | Required Conditions |
|---|---|---|---|
| Auto | PLC automatically controls based on process conditions; operator sets setpoints only | PLC (automatic) or Operator command to enter Auto | No active faults; all permissives satisfied |
| Manual | Operator starts/stops from HMI; process interlocks remain active | Operator (HMI) | No active faults |
| Step / Jog | Operator inches equipment in short pulses for maintenance/commissioning | Maintenance (key switch) | Equipment in safe state; E-stops active |
| {Add modes from interview Round 2} | | | |

**Mode Priority** (highest to lowest):
1. E-Stop (overrides all)
2. Safety function trip (SIS output)
3. Fault / Trip (automatic equipment protection)
4. Manual
5. Auto
6. Step / Jog

**Mode Transition State Diagram** (ASCII):

```
                       ┌──────────┐
                       │  E-STOP  │ ◀── from any mode
                       └────┬─────┘
                            │ reset
                            ▼
   ┌──────┐  start   ┌──────────┐    stop    ┌──────────┐
   │ AUTO │ ───────▶ │ RUNNING  │ ────────▶ │ STOPPING │
   └──────┘          └──────────┘            └─────┬────┘
       ▲                  ▲                        │
       │ select Manual    │ Manual start           │
       │                  │                        ▼
   ┌────┴─────┐           │                   ┌────────┐
   │ MANUAL   │ ──────────┘                   │ STOPPED│
   └──────────┘                                └────────┘
```

**Mode Transitions:** Mode changes take effect at the next PLC scan after operator selection. The current equipment state (running/stopped) does not change automatically — the operator must issue a Start or Stop command after changing modes.

---

## 6. Permissives, Interlocks, Safety, C&E Matrix, Setpoints

### 6.1 Start Permissives

All conditions TRUE before equipment is permitted to start in Auto or Manual:

| # | Permissive | Tag | Condition Required | Source |
|---|---|---|---|---|
| P-01 | {e.g., Discharge valve open} | {XV-101_OpenFB} | {XV-101 open feedback TRUE} | Customer / FDS-REQ-XXX / [ASSUMED from Nexus] |
| P-02 | {No active faults} | {fault summary tag} | {Fault summary bit FALSE} | |
| P-03 | {Motor thermal overload reset} | {OL_Reset_Sts} | {Thermal overload not tripped} | |

### 6.2 Run Interlocks (Trip Conditions)

Any condition occurring while equipment is running causes an **immediate trip**:

| # | Interlock | Tag | Trip Condition | Response | Reset |
|---|---|---|---|---|---|
| I-01 | {High discharge pressure} | {PSH-101} | {PIT-101 > 90 PSI for > 2 s} | Trip motor; close XV-101; Alarm AL-001 | Operator HMI reset after pressure normalizes |
| I-02 | {Motor overload} | {M-101_OL} | {Overload relay tripped} | Trip motor; Alarm AL-002 | MCC reset, then HMI reset |

### 6.3 Safety Functions

If safety-rated functions exist, they are implemented in a safety-rated control system separate from the BPCS described in this SOO. Safety functions CANNOT be bypassed from the HMI and are not affected by mode selection.

| Safety Function | SIL / PL Target | Initiating Event | Safe State | Reset Method | Proof Test Interval |
|---|---|---|---|---|---|
| {E-Stop Zone 1} | {PLe / SIL 2} | {Any Zone 1 E-stop actuated} | {All Zone 1 outputs de-energized} | {Pull E-stop, HMI safety reset} | {Annual} |
| {SIF-01 High-Pressure Trip} | {SIL 2} | {PIT-101 > 100 PSI for > 1 s} | {Close inlet, vent, alarm} | {HMI safety reset after cause cleared} | {6 months} |

If no safety functions: "No safety-rated (SIL/PL-certified) functions are defined for this equipment at this time. All interlocks listed in §6.2 are process interlocks only."

### 6.4 Cause and Effect Matrix

Per **ANSI/ISA-5.06.01-2007 §5.7**. Initiators (rows) × Actions (columns). `✕` at row × column where the initiator drives the action. Surfaces missing or duplicate actions in O(N²) glance time.

| Initiator → \\ Action ↓ | Trip Motor | Close XV-101 | Open Vent | AL-001 (Critical) | AL-002 (Critical) | AL-003 (High) | Mode → Faulted | Operator Reset Required |
|---|---|---|---|---|---|---|---|---|
| I-01 High Pressure (PIT-101 > 90 PSI 2s) | ✕ | ✕ | | ✕ | | | ✕ | ✕ |
| I-02 Motor Overload | ✕ | | | | ✕ | | ✕ | ✕ (MCC + HMI) |
| F-001 Motor Failed to Start | | | | | | ✕ | ✕ | ✕ |
| Safety SIF-01 | ✕ | ✕ | ✕ | ✕ | | | ✕ | ✕ (Safety reset) |
| E-Stop Zone 1 | ✕ | | | ✕ | | | ✕ | ✕ (E-stop reset) |
| {add row per fault, interlock, permissive failure, safety trip} | | | | | | | | |

**Quality Check rule (E7):** every initiator row must correspond to a fault/interlock/permissive/safety entry; every action column must correspond to a §11/§12 entry.

### 6.5 Setpoint Authorization Table

Single source-of-truth for every setpoint, timer, and limit. Resolves all `{X}` placeholders via the Round 3 Q3 interview answer. Audit-grade: who can change what.

| Tag | Description | Default | Operator Bounds | Supervisor Bounds | Engineer Bounds | Hard-Stop Limit | Source | Last Reviewed |
|---|---|---|---|---|---|---|---|---|
| `RestartLockout_s` | Restart lockout after any stop | 30 s | 15–60 s | 10–120 s | 5–300 s | 5–600 s | Customer / FDS-REQ-XXX | {date} |
| `MinRunTime_min` | Minimum run time before normal stop accepted | 5 min | 2–10 min | 1–15 min | 0–30 min | 0–60 min | Customer | {date} |
| `StartupTimeout_s` | Total startup sequence timeout | 60 s | n/a | 30–180 s | 30–300 s | n/a | HAZOP | {date} |
| `ValveOpenTimeout_s` | Per-valve open feedback timeout | 15 s | n/a | 10–30 s | 5–60 s | n/a | OEM | {date} |
| `MotorRunFB_Timeout_s` | Motor run feedback after run command | 5 s | n/a | 3–10 s | 1–30 s | n/a | OEM | {date} |
| `PressureBuildup_Timeout_s` | Discharge pressure to reach minimum after start | 30 s | n/a | 15–60 s | 5–120 s | n/a | Customer | {date} |
| `HMI_Watchdog_s` | HMI heartbeat timeout | 10 s | n/a | 5–30 s | 1–60 s | n/a | HIS | {date} |
| `MotorCooling_min` | Motor cooling after overload before restart | 15 min | n/a | n/a | 5–60 min | 5 min | OEM | {date} |
| `SuctionValve_IdleHours` | Idle hours after which suction valve auto-closes | 4 h | 2–24 h | 1–48 h | 0–168 h | n/a | Customer | {date} |
| `MaxPressure_PSI` | Maximum allowable discharge pressure | 75 PSI | n/a | n/a | n/a | 90 PSI (HAZOP) | HAZOP | {date} |
| `MinFlow_GPM` | Minimum acceptable flow before low-flow trip | 50 GPM | 40–100 GPM | n/a | n/a | 30 GPM | Customer | {date} |
| {add all setpoints from Round 3 Q3} | | | | | | | | |

---

## 7. Startup Sequence

### 7.1 Pre-Start Checks

**Equipment-type-specific Pre-Start Checklist** (per Round 2 follow-up answer):

{{IF_EQUIPMENT_PUMP}}
1. Suction strainer clear of debris.
2. Suction valve open and locked in position.
3. Discharge valve in starting position (closed for centrifugal-against-closed-valve start; open for positive-displacement).
4. Motor coupling guard in place.
5. All E-stop buttons in released (non-actuated) position.
6. Local disconnect at MCC in ON position.
7. HMI displays no active faults or alarms for this equipment.
8. All permissives in §6.1 satisfied (green on HMI faceplate).
{{IF_EQUIPMENT_FAN}}
1. Inlet damper in starting position.
2. Outlet damper in starting position.
3. Vibration sensor reset; baseline within normal.
4. Bearing oil reservoir at marked level.
5. All E-stop buttons released.
6. Local disconnect ON.
7. HMI no active faults.
8. All §6.1 permissives green.
{{IF_EQUIPMENT_REACTOR}}
1. Agitator clear (no foreign objects).
2. Cooling water flow established to jacket.
3. Jacket pressure relief verified.
4. Charge inventory confirmed against recipe.
5. All E-stop buttons released.
6. Local disconnect ON.
7. HMI no active faults.
8. All §6.1 permissives green.
{{IF_EQUIPMENT_OTHER}}
1. {Custom equipment-type checks from Round 2 follow-up.}
2. All E-stop buttons released.
3. Local disconnect ON.
4. HMI no active faults.
5. All §6.1 permissives green.

**If any pre-start check fails:** Do not proceed. Clear the failing condition before initiating start.

### 7.2.1 Cold-Start Auto Mode (First Start of Day / After Full Outage)

Cold start = motor cold, lines unprimed, all valves at fail-state, full priming sequence required.

**Entry conditions:**
- System in Auto mode
- All §6.1 permissives satisfied
- Supervisory Start command received
- Last shutdown was > {SuctionValve_IdleHours} ago, or system is recovering from a planned outage

**Steps:**

1. PLC verifies all start permissives per §6.1. If any unmet, startup is inhibited; HMI displays the specific unsatisfied permissive.
2. PLC commands suction isolation valve (XV-100) to open. Wait up to `ValveOpenTimeout_s` ({ValveOpenTimeout_s} s) for open feedback.
   - **If no feedback in time:** abort startup, raise AL-003 (Suction Valve Failed to Open), return to Stopped.
3. PLC commands discharge isolation valve (XV-101) to open. Same timeout / abort logic.
4. {{IF_EQUIPMENT_PUMP}}Priming verification: PLC monitors PIT-101. If pressure > 5 PSI after 10 s, suction is primed.{{ELSE}}{Equipment-specific priming step or N/A.}
5. PLC energizes motor starter (M-101_RunCmd). Expect full speed within `MotorRunFB_Timeout_s` ({MotorRunFB_Timeout_s} s).
   - **If no run feedback:** remove run command, raise AL-004 (Motor Failed to Start), return to Stopped.
6. PLC monitors discharge pressure. Wait up to `PressureBuildup_Timeout_s` ({PressureBuildup_Timeout_s} s) for PIT-101 ≥ `MinFlow_GPM` equivalent ({MinFlow_GPM} GPM).
   - **If pressure does not rise:** stop motor, raise AL-005 (Pump Failed to Prime), return to Stopped.
7. Once running and pressure confirmed, transition to Normal Operation (§8). HMI updates "Starting" → "Running."

**Total cold-start timeout:** `StartupTimeout_s` ({StartupTimeout_s} s). If not reached Step 7 within this window, abort, raise AL-006 (Startup Sequence Timeout), return to Stopped.

### 7.2.2 Hot-Restart Auto Mode (Restart After Within-Shift Trip)

Hot restart = motor warm, lines may be primed, valves per pre-trip state, post-trip-cause clearance required first.

**Entry conditions:**
- Trip occurred within current shift (last cold-start < {SuctionValve_IdleHours} ago)
- Trip cause acknowledged and cleared in HMI
- All §6.1 permissives now satisfied
- Restart Lockout `RestartLockout_s` ({RestartLockout_s} s) has elapsed since trip

**Steps:**

1. PLC verifies trip cause is cleared (the previous interlock condition is now FALSE).
2. PLC verifies all §6.1 permissives.
3. {{IF_VALVES_RETAINED_STATE}}If isolation valves are still in pre-trip Open state, skip to Step 5. Otherwise, run §7.2.1 Steps 2–3.{{ELSE}}Run §7.2.1 Steps 2–3.
4. {{IF_LINES_STILL_PRIMED}}Skip priming (lines warm). Verify pressure baseline > 0 PSI. If pressure has fallen to 0 PSI, treat as cold start (run §7.2.1).{{ELSE}}Run §7.2.1 Step 4.
5. Energize motor (run §7.2.1 Step 5 with reduced timeout `MotorRunFB_Timeout_s` × 0.7 — motor is warm, expects faster start).
6. Verify discharge pressure rise within `PressureBuildup_Timeout_s` × 0.5 (faster pressure response on hot restart).
7. Transition to Normal Operation.

### 7.3 Manual Mode Startup

1. Operator confirms §7.1 pre-start checks complete.
2. Operator selects Manual mode on HMI.
3. Operator verifies all permissives green on faceplate.
4. Operator presses Start.
5. PLC executes the same startup sequence (§7.2.1 Cold or §7.2.2 Hot, per system state).
6. Operator monitors faceplate; on abort, HMI displays failed step + alarm.

### 7.4 Step / Jog Mode Startup (Maintenance Only)

Jog mode is enabled by maintenance key switch in JOG position.

1. Maintenance technician confirms area clear, all personnel at safe distance.
2. Technician turns key switch to JOG. HMI confirms Jog mode active.
3. Valve pre-positioning (§7.2.1 Steps 2–3) does NOT execute. Motor energizes directly.
4. Technician presses and holds Jog button. Motor energizes while held.
5. On release, motor de-energizes.
6. E-stop and thermal overload interlocks remain active.
7. Minimum Run Time does NOT apply.

### 7.5 Startup Timing Diagram (ASCII)

For Cold-Start Auto sequence ({{IF_EQUIPMENT_PUMP}}pump example{{ELSE}}equipment-specific{{END}}):

```
Time (s) →   0      15     30     35     65          {StartupTimeout_s}
              │      │      │      │      │           │
Permissives ──█      .      .      .      .           .
XV-100 cmd .──█──────█      .      .      .           .
XV-100 FB  .──.──────█      .      .      .           .   (open)
XV-101 cmd .──.──────█──────█      .      .           .
XV-101 FB  .──.──.───.──────█      .      .           .   (open)
Prime chk  .──.──.───.──────█──────█      .           .
Motor cmd  .──.──.───.──.───.──────█      .           .
Motor FB   .──.──.───.──.───.──.───█      .           .   (running)
Pressure   .──.──.───.──.───.──.───.──────█           .   (≥ MinFlow)
Running    .──.──.───.──.───.──.───.──.───█───────────█

Legend: █ = active, . = inactive
```

### 7.6 CIP Cycle Sub-Sequence (Conditional — only if CIP mode in Round 2)

For each CIP step (rinse / wash / sanitize / final rinse / verification), the SOO emits:

| Step | Action | Setpoint Tag | Duration | Verification |
|---|---|---|---|---|
| CIP-01 Pre-rinse | Open CIP supply valve, run rinse pump | `CIP_PreRinseTime_min` | {value} min | Drain pH within range |
| CIP-02 Wash | Inject detergent at concentration {C} | `CIP_WashTime_min` | {value} min | Conductivity reaches setpoint |
| CIP-03 Sanitize | Inject sanitizer | `CIP_SanitizeTime_min` | {value} min | Temperature ≥ {T}, contact time ≥ {t} |
| CIP-04 Final rinse | Rinse to product-quality water | `CIP_FinalRinseTime_min` | {value} min | Drain conductivity ≤ supply conductivity + Δ |
| CIP-05 Verification | ATP swab / drain test | n/a | n/a | ATP reading ≤ {threshold} |
| CIP-06 Return-to-process | Close CIP valves, open process valves | n/a | n/a | All valves verified per §6.1 |

If CIP mode not in Round 2: this section reads "Not applicable — no CIP cycle in scope."

---

## 8. Normal Operation

### 8.1 Steady-State Baseline Parameters

**What "Running" looks like.** All values pulled from §6.5 and Round 3 Q3 interview.

| Parameter Tag | Normal Range | Warning Range | Alarm Range | Operator Action If Out-of-Range |
|---|---|---|---|---|
| PIT-101 (discharge pressure) | 60–70 PSI | 50–60 or 70–80 | < 50 or > 80 | Check suction; check discharge path; if ≥ MaxPressure-5, prepare for trip |
| FT-101 (flow) | 200–300 GPM | 150–200 | < `MinFlow_GPM` | Check suction; check upstream flow source |
| ACT-101 (motor current) | 8–12 A | 12–14 A | > 14 A | Check for mechanical drag; check process load; check VFD parameters |
| TT-101 (bearing temp) | 50–80 °C | 80–95 °C | > 95 °C | Check lubrication; reduce load; prepare for shutdown |
| VIB-101 (vibration) | 0–2.5 mm/s | 2.5–4.5 mm/s | > 4.5 mm/s | Schedule inspection; if > 7.1 mm/s, immediate shutdown (ISO 10816-3 alarm) |
| {Add per equipment} | | | | |

### 8.2 Control Loops

| Loop | Tag | Controlled Variable | Manipulated Variable | Setpoint | Normal Range | Action |
|---|---|---|---|---|---|---|
| {Discharge Pressure Control} | {PID-101} | {PIT-101} | {VFD speed command} | {65 PSI} | 60–70 PSI | Reverse-acting |

### 8.3 Continuous Monitoring During Normal Operation

While running, PLC monitors at every scan:
- All §6.2 run interlocks remain active.
- All §8.1 baseline parameters within ranges; PLC raises Warning alarm (Priority 3) before reaching interlock limit.
- {Equipment-specific monitoring per interview Round 3 Q1.}

### 8.4 Minimum Run Time

Equipment must run for `MinRunTime_min` ({MinRunTime_min} min) before normal Stop is accepted. Protects against short-cycling.

- Stop command during MRT: HMI displays "Minimum Run Time Active — Stop will execute at {time}."
- BYPASSED for E-stop and safety function trips.
- BYPASSED for interlock trips.

### 8.5 Restart Lockout

After equipment stops for any reason, `RestartLockout_s` ({RestartLockout_s} s) lockout before new Start accepted.

- HMI displays countdown timer.
- Same in Auto and Manual.
- Does NOT apply to initial start after power restoration (§10.2).

### 8.6 Partial-Failure Operation

Conditional — applies only if Round 1 scope = Equipment group or Full system AND Round 2 follow-up identified parallel/redundant equipment groups.

**Redundancy Group: {Group Name}** (e.g., "Three influent pumps in parallel — P-101 / P-102 / P-103"):

| Failed Devices | System Behavior | Reduced Capacity | Alarm | Operator Action |
|---|---|---|---|---|
| 0 (all running) | Normal operation; rotating duty by run-hours | 100% | None | None |
| 1 of 3 (any one failed) | Continue on remaining 2; equalize load | ~67% capacity | AL-PF-001 (High) | Schedule maintenance; inspect failed unit |
| 2 of 3 | Continue on remaining 1; alarm at high priority | ~33% capacity | AL-PF-002 (Critical) | Investigate cause; restore redundancy ASAP |
| 3 of 3 | System trips; safe shutdown | 0% (process trip) | AL-PF-003 (Critical) + interlock cascade | Emergency response per §10 |

If scope = Single device: this section reads "Not applicable — single-device scope."

---

## 9. Normal Shutdown Sequence

### 9.1 Auto Mode Shutdown

PLC initiates shutdown when:
- Supervisory Stop command received, OR
- Process condition no longer requires equipment (setpoint satisfied, batch step complete), OR
- Operator selects Stop from HMI

**Steps:**

1. PLC removes run command (M-101_RunCmd = FALSE). VFD ramps motor to stop over `VFD_DecelTime_s` s.
2. PLC waits up to `MotorRunFB_Timeout_s` ({MotorRunFB_Timeout_s} s) for motor run feedback to clear.
   - If feedback does not clear: raise AL-007 (Motor Failed to Stop). Operator must investigate MCC and motor before allowing restart.
3. Once motor confirmed stopped, PLC commands XV-101 closed. Wait up to `ValveOpenTimeout_s` ({ValveOpenTimeout_s} s) for close feedback.
4. XV-100 remains open after shutdown to maintain prime for next start. If idle > `SuctionValve_IdleHours` ({SuctionValve_IdleHours} h), operator may manually close XV-100 from HMI.
5. System enters Stopped state. HMI updates "Running" → "Stopped." System ready for restart after `RestartLockout_s` ({RestartLockout_s} s).

### 9.2 Manual Mode Shutdown

1. Operator selects Stop on HMI.
2. PLC executes the same shutdown sequence (§9.1).
3. HMI displays shutdown progress on faceplate.

### 9.3 CIP Cycle End (Conditional — CIP mode only)

After CIP-06 Return-to-process, normal §9.1 shutdown applies. CIP records signed by operator; logged per §16.4 (FSMA / 3-A) or §16.2 (21 CFR Part 11) overlay.

---

## 10. Emergency Shutdown / E-Stop / Power Loss

### 10.1 E-Stop Actuation

E-stop is hardwired; bypasses the PLC and de-energizes outputs **immediately upon actuation**, regardless of mode.

**When any E-stop in {zone name} is actuated:**

1. **Immediately (within {hardware_response_ms} ms, hardwired):** Safety relay de-energizes safety output contactor. All controlled outputs in zone de-energize simultaneously. Motor stops without deceleration ramp.
2. **Within 1 PLC scan:** PLC detects E-stop input de-energized; sets EST_Zone1_Fault = TRUE; raises AL-010 (Critical).
3. **On HMI:** E-stop alarm banner appears immediately. Operator display switches to affected area overview. Affected faceplates show "E-Stop Active" in red.
4. **System remains in E-stop state** until: (a) E-stop button physically released, AND (b) operator acknowledges E-stop alarm, AND (c) operator presses E-stop Reset on HMI or local panel.
5. After reset: system enters Stopped state. All permissives re-evaluated. **System will NOT auto-restart after E-stop** — operator must issue new Start command.

**E-Stop Recovery Procedure:**

1. Identify and correct the cause of actuation (or confirm intentional actuation).
2. Confirm work area clear of personnel.
3. Physically release E-stop button(s) — pull or twist as marked.
4. Acknowledge E-stop alarm on HMI.
5. Press E-stop Reset on HMI or local panel. Safety relay re-energizes.
6. Confirm HMI shows no remaining E-stop or safety faults.
7. Issue new Start through normal startup (§7).

### 10.2 Power Loss and Restoration

**On loss of control power:**
1. All outputs de-energize immediately. Valves return to spring-fail positions per P&ID.
2. PLC loses program execution. No alarms during power loss (no power to HMI).

**On restoration of control power:**
1. PLC performs power-up diagnostic ({power_up_diagnostic_s} s). All outputs remain de-energized during this time.
2. PLC enters Run mode and evaluates all permissives.
3. System enters Stopped state. **Equipment does NOT auto-restart** regardless of pre-loss mode.
4. HMI logs "System Power Restored" event.
5. Operator confirms process state safe; issues new Start.

### 10.3 HMI Loss vs PLC Loss (Differential)

| Loss Type | PLC Behavior | Operator Visibility | Action Required |
|---|---|---|---|
| HMI loss only (PLC running) | PLC continues running; alarms accumulate in PLC alarm queue | Lost; control through redundant operator station or local control | Investigate HMI server / network; reconnect; alarm history backfilled on reconnect |
| PLC loss only (HMI running) | All outputs go to fail-safe; communication-loss timeout fires | "Comms Lost" banner; last-known values displayed | Investigate PLC; restore Run mode; manual restart |
| Both loss (full power outage) | See §10.2 | Both lost | See §10.2 |
| UPS-backed PLC, AC-load loss | PLC runs; AC loads (motors, contactors) drop | HMI shows alarms for dropped loads | Restore AC; clear alarms; manual restart of AC loads |

### 10.4 Network Partition Recovery

If control network is partitioned (e.g., switch loss splitting PLC from remote I/O drop):
1. PLC and partitioned drop both detect comms loss; all I/O on the partitioned drop go to fail-safe per §6.2 / Equipment List.
2. PLC raises AL-NET-XXX (network partition) for the affected drop.
3. On network restore: drop re-establishes comms; PLC re-evaluates inputs; alarm clears once stable for {network_stable_s} s.
4. Operator confirms drop status green on HMI before resuming control.

---

## 11. Fault Response

### 11.1 Fault Response Table

Every fault the control system can detect. Each fault: detection method → automatic response → alarm → operator recovery.

| Fault # | Description | Detection Method | Tag | Automatic Response | Alarm | Operator Recovery |
|---|---|---|---|---|---|---|
| F-001 | Motor failed to start | M-101_RunFB not received within `MotorRunFB_Timeout_s` of run cmd | {M-101_RunFB} | Remove run command; abort startup; enter Faulted | AL-004 Critical | Check MCC; reset overload if tripped; ack alarm; new start |
| F-002 | Motor overload trip | M-101_OL contact opens | {M-101_OL} | Remove run cmd immediately; enter Faulted | AL-002 Critical | Allow `MotorCooling_min` cooling; investigate cause; reset OL at MCC; ack; new start |
| F-003 | High discharge pressure | PIT-101 > `MaxPressure_PSI` for 2 s | {PIT-101} | Remove run cmd; enter Faulted | AL-001 Critical | Confirm discharge open; check downstream blockage; investigate; ack; new start after pressure normalizes |
| F-004 | Failed to prime | PIT-101 < min for 30 s after motor start | {PIT-101} | Remove run cmd; abort; enter Faulted | AL-005 Critical | Confirm suction; manually prime; ack; retry |
| F-099 | PLC–HMI comms loss | HMI heartbeat watchdog timeout | {Comms_Watchdog_Sts} | No equipment change; HMI displays "COMMS LOST" | AL-099 High | Investigate network; check switch; contact HIS if persistent |

### 11.2 Interlock Test Procedures

Used at FAT and SAT to verify each interlock. Each row is a procedure step.

| Interlock # | Procedure | Pass Criteria | Reset Path |
|---|---|---|---|
| I-01 High Pressure Trip | (1) Equipment running. (2) Force PIT-101 to 95 PSI for 3 s. (3) Observe motor stop and XV-101 close. | Motor de-energizes within 100 ms of force; XV-101 closes within `ValveOpenTimeout_s`; AL-001 fires; equipment in Faulted | (1) Force PIT-101 back to normal. (2) Ack AL-001. (3) Operator HMI Reset. (4) Verify §7.2.1 Hot-Restart works. |
| I-02 Motor Overload | (1) Equipment running. (2) Open M-101_OL contact in MCC. (3) Observe motor trip. | Motor de-energizes within 1 PLC scan; AL-002 fires | (1) Reset OL at MCC. (2) Ack AL-002. (3) Operator HMI Reset. (4) Verify hot restart. |
| SIF-01 (SIL-rated proof test) | Per IEC 61511-1 §16 proof test procedure (separate doc). | SIF response time within proof-test SLA; all final elements move to safe state | Per SIS reset procedure (separate from BPCS reset). |
| {add per interlock from §6.2 and per safety function from §6.3} | | | |

### 11.3 Troubleshooting Decision Trees

Per top-N faults, common-causes-by-frequency (most-likely → rare):

**F-001 Motor Failed to Start:**
1. **Most likely:** Tripped overload — check OL at MCC; reset; verify motor not seized. → §13.5 Measurement Points (motor current at startup attempt) → §13.3 LOTO before mechanical inspection.
2. **Likely:** MCC breaker open — check breaker; reset if tripped; verify line voltage.
3. **Possible:** Wiring fault between PLC and starter — check field wiring (`/io-list` and `/wire-diagram` outputs) → §13.5 Measurement Points (continuity test).
4. **Rare:** PLC output module failure — swap output module per `/bom` Section 9 Spare Parts; verify channel.

**F-002 Motor Overload Trip:**
1. **Most likely:** Mechanical overload (process load increased, mechanical drag). → Inspect coupling, bearings (§13.6 PM Schedule). § 13.3 LOTO before any mechanical work.
2. **Likely:** Incorrect OL setting (set too low for current load).
3. **Possible:** Phase loss / unbalanced supply.
4. **Rare:** OL relay degraded.

**{Add tree per top-N fault from interview Round 3 Q1.}**

---

## 12. Alarm Summary

Per ANSI/ISA-18.2-2016 priority scheme:
- **Priority 1 — Critical:** Operator response within 5 minutes. Risk to personnel, environment, or major equipment.
- **Priority 2 — High:** Within 15 minutes. Product loss or minor equipment damage.
- **Priority 3 — Medium:** Within 1 hour. Abnormal condition approaching limit.
- **Priority 4 — Low / Journal:** Informational.

| Alarm # | Description | Priority | Setpoint / Condition | Cause | Operator Action | Suppression Rules |
|---|---|---|---|---|---|---|
| AL-001 | High Discharge Pressure | 1 | PIT-101 > `MaxPressure_PSI` for 2 s | Closed discharge, over-speed, blockage | Stop equipment; investigate path; clear blockage | Suppress during startup (first 30 s) |
| AL-002 | Motor Overload Trip | 1 | M-101_OL open | Mechanical overload, electrical fault, OL setting | Allow cooling; investigate; reset OL at MCC | None |
| AL-003 | Suction Valve Failed to Open | 1 | XV-100 no FB within `ValveOpenTimeout_s` | Actuator fault, air supply, mech stuck | Inspect locally; check air; manual operation | Suppress when stopped |
| AL-004 | Motor Failed to Start | 1 | M-101 no FB within `MotorRunFB_Timeout_s` | MCC breaker, wiring, motor fault | Check MCC; check breaker; inspect motor | Suppress when stopped |
| AL-005 | Pump Not Primed | 1 | PIT-101 < min for `PressureBuildup_Timeout_s` | No suction, air-bound, suction valve closed | Confirm suction; prime; retry | Suppress first 30 s of startup |
| AL-099 | PLC–HMI Comms Loss | 2 | Heartbeat watchdog timeout | Network, HMI server | Investigate network; contact HIS | None |
| {add per fault} | | | | | | |

**Alarm design rules (ISA-18.2):**
- Each alarm has setpoint with deadband; clears when condition returns to normal range.
- No re-alarm (chattering) within {chatter_min} min of previous annunciation for same condition.
- Standing alarms (active > 24 h) reviewed at shift handover (§13.2).
- Alarm shelving available to maintenance for confirmed equipment-out-of-service, max 8 h shelving before automatic re-annunciation.

---

## 13. Operator and Maintenance Reference

### 13.1 HMI Faceplate Elements

| Element | Description | Location |
|---|---|---|
| Mode selector | Auto / Manual / {other} toggle | Top left |
| Start button | Issues Start (visible in Manual mode) | Center |
| Stop button | Issues Stop | Center |
| Equipment state | Stopped / Starting / Running / Faulted / E-Stop | Status bar |
| Run indicator | Green=running, red=stopped, yellow=starting | Status indicator |
| Fault indicator | Red when active; opens fault detail popup | Status bar |
| Permissive status | All §6.1 with green/red | Permissives tab |
| Setpoints | Editable per §6.5 authorization | Setpoints tab |
| {Key PV} | Current with trend mini-graph | Center |
| Override / Force indicator | Yellow border if any active force or override | Persistent header |

If `/hmi-spec` output exists in folder, cross-reference for full faceplate spec.

### 13.2 Operator Responsibilities

The operator is responsible for:
- Monitoring faceplate during startup; confirming each step completes without abort.
- Acknowledging alarms within priority time limits (§12).
- Recording abnormal conditions in shift log even if equipment self-recovered.
- Reporting any alarm that activated more than once in a shift to maintenance.
- Reading alarm description before acknowledging — never silent-acknowledge.
- Performing shift handover with review of standing alarms (any > 24 h active).

### 13.3 Lockout / Tagout (LOTO) Points

Per **OSHA 29 CFR 1910.147** + **ANSI/ASSP Z244.1-2016**. One row per energy source per piece of equipment.

| Equipment | Energy Source | Isolation Device | Location | Lock-Out Procedure Step |
|---|---|---|---|---|
| {M-101 motor} | Electrical 480 VAC 3φ | Motor branch breaker | MCC bucket {N} | (1) Lock breaker OFF. (2) Test for 0 V at motor terminals. (3) Apply tag. |
| {M-101 motor} | Stored kinetic / coast-down | Motor coupling | At pump | Wait {coastdown_s} s for shaft to stop. |
| {XV-100 / XV-101 valves} | Pneumatic 80 psi | Air supply ball valve at JB | Field JB-1 | (1) Close ball valve. (2) Vent downstream. (3) Apply tag. |
| {PIT-101 transmitter} | Process pressure | Block-and-bleed manifold | At transmitter | (1) Close primary block. (2) Open bleed. (3) Verify 0 PSI. (4) Apply tag. |
| {PSU 24 VDC} | Electrical 24 VDC | DIN-rail breaker B-X | Panel A | (1) Open breaker. (2) Test for 0 V on PSU output. (3) Apply tag. |
| {add per equipment from interview} | | | | |

Cross-reference to equipment-specific LOTO procedure document (if customer maintains a separate procedure).

### 13.4 Operator Skills Verification

Trained-operator demonstration checklist (signed by operator and supervisor at training completion):

| # | Skill | Demonstrated | Sign-off (Operator / Date) | Sign-off (Supervisor / Date) |
|---|---|---|---|---|
| 1 | Cold start unsupervised (§7.2.1) | ☐ | | |
| 2 | Hot restart unsupervised (§7.2.2) | ☐ | | |
| 3 | Recognize and respond to top-5 faults (per Quick-Reference Card) | ☐ | | |
| 4 | Perform normal shutdown (§9) | ☐ | | |
| 5 | Perform emergency shutdown (§10.1) | ☐ | | |
| 6 | E-stop response and recovery (§10.1) | ☐ | | |
| 7 | Shift handover — review standing alarms | ☐ | | |
| 8 | Escalate alarms beyond authority (per §6.5 authorization bounds) | ☐ | | |
| 9 | {Industry-specific skill — e.g., CIP cycle execution / batch record sign-off / PSM operator-deviation log} | ☐ | | |

### 13.5 Measurement Points (for Maintenance)

Where to put your meter / probe; what's normal; what's action-threshold.

| Tag | Where to Measure | Instrument | Normal Range | Caution Range | Action Range | Reference |
|---|---|---|---|---|---|---|
| M-101 current | Motor terminals (T1/T2/T3) | Clamp ammeter | 8–12 A | 12–14 A | > 14 A | OEM motor nameplate |
| M-101 vibration | Motor bearings (DE / NDE) | Vibration probe | 0–2.5 mm/s | 2.5–4.5 mm/s | > 4.5 mm/s | ISO 10816-3 |
| M-101 winding temp | Motor surface (thermography) | IR thermometer | < 60 °C | 60–85 °C | > 85 °C | OEM Class B/F insulation |
| PIT-101 loop voltage | Transmitter terminals | Multimeter | 12–30 VDC | 10–12 or 30–32 | < 10 or > 32 | Loop power supply |
| PIT-101 loop current | Transmitter loop | Clamp meter (mA) | 4–20 mA | 3.6–4 or 20–22 | < 3.6 or > 22 | Calibration cert |
| Bearing oil level | Sight glass at pump | Visual | Mid-glass | Lower 1/3 | Below 1/3 | OEM manual |
| Coupling alignment | Dial indicator at coupling | Indicator | < 0.05 mm runout | 0.05–0.10 mm | > 0.10 mm | OEM tolerance |
| {add per equipment} | | | | | | |

### 13.6 Preventive Maintenance and Inspection Schedule

| Task | Frequency | Next Due | Who Performs | Reference |
|---|---|---|---|---|
| Motor bearing grease | Every 4,000 run-hours | {date} | Maintenance | OEM manual |
| Motor coupling alignment check | Annually | {date} | Maintenance | OEM tolerance |
| Vibration analysis baseline | Quarterly | {date} | Reliability | ISO 10816-3 |
| Pump packing / mechanical seal inspection | Every 8,000 run-hours | {date} | Maintenance | OEM manual |
| Suction strainer cleanout | Monthly | {date} | Operations | Plant practice |
| Field-instrument calibration (PIT-101, FT-101) | Annually | {date} | Instrumentation | Calibration cert + ISA-RP-105 |
| Safety-function proof test | Per §6.3 interval | {date} | Functional Safety Manager | IEC 61511-1 §16 proof procedure |
| ISO 13849-1 mission time review | Per Cat 3 / 4 mission time | {date} | Functional Safety Manager | ISO 13849-1:2023 §11 |
| LOTO-point function check | Annually | {date} | Maintenance | OSHA 1910.147 |
| {add per equipment} | | | | |

---

## 14. Open Items and TBDs

| # | Section | Description | Owner | Target Date | Status |
|---|---|---|---|---|---|
| TBD-001 | §{N} | {description} | {customer / HIS / engineer} | {date} | Open |

Every `[ASSUMED from Nexus — verify with customer]` flag and every remaining `[TBD]` placeholder appears as a row here.

---

## 15. Glossary

| Term | Definition |
|---|---|
| ALCOA+ | Attributable, Legible, Contemporaneous, Original, Accurate + Complete, Consistent, Enduring, Available |
| AOI | Add-On Instruction (Rockwell) |
| ATP | Adenosine Triphosphate (CIP cleaning verification swab) |
| AWIA | America's Water Infrastructure Act of 2018 |
| C&E | Cause and Effect (Matrix) |
| CIP | Common Industrial Protocol or Clean-in-Place (context-dependent) |
| DI / DO / AI / AO | Digital Input / Digital Output / Analog Input / Analog Output |
| DCavg | Average Diagnostic Coverage (ISO 13849-1) |
| FDS | Functional Design Specification |
| HAZOP / LOPA | Hazard and Operability Study / Layer of Protection Analysis |
| HMI | Human-Machine Interface |
| LOTO | Lockout / Tagout |
| MCC | Motor Control Center |
| MTTFd | Mean Time To Dangerous Failure (ISO 13849-1) |
| NERC CIP | Critical Infrastructure Protection |
| OL | Overload relay |
| OSHA PSM | Process Safety Management (29 CFR 1910.119) |
| P&ID | Piping and Instrumentation Diagram |
| PFDavg / PFH | Average Probability of Failure on Demand / per Hour (IEC 61511) |
| PL | Performance Level (ISO 13849-1) |
| PLC | Programmable Logic Controller |
| QRC | Quick-Reference Card |
| RPI | Requested Packet Interval |
| RTM | Requirements Traceability Matrix (in parent FDS) |
| SIF / SIS / SIL | Safety Instrumented Function / System / Integrity Level |
| SOO | Sequence of Operations |
| SP / PV | Setpoint / Process Variable |
| UDT | User-Defined Data Type |
| VFD | Variable Frequency Drive |
| {Customer Acronym} | {Definition} |

---

## 16. Industry-Specific Operating Procedures Overlay

{{Emit one or more sub-sections matched to the Round 1 industry answer. If industry = Other or no overlay applies, write: "Not applicable — no regulated-industry overlay required for this equipment."}}

### 16.1 OSHA PSM 29 CFR 1910.119 § (f) Operating Procedures (Oil & Gas / Petrochemical / Chemical)

This SOO **is** the written operating procedures deliverable per OSHA PSM § (f)(1). Coverage of required elements:

| PSM § (f)(1) Element | This SOO Section |
|---|---|
| (i) Initial startup | §7.2.1 Cold-Start Auto |
| (ii) Normal operations | §8 |
| (iii) Temporary operations | §7.6 CIP Cycle (or `[Not Applicable]`) |
| (iv) Emergency shutdown — including conditions and assignment of responsibility | §10.1 + §13.2 Operator Responsibilities |
| (v) Emergency operations | §10 |
| (vi) Normal shutdown | §9 |
| (vii) Startup following turnaround / emergency shutdown | §7.2.2 Hot-Restart + §10.1 E-Stop Recovery |
| (viii) Operating limits with consequences of deviation | §6.5 Setpoint Authorization Table + §8.1 Steady-State Baseline |
| (ix) Safety and health considerations | §6.3 Safety Functions + §13.3 LOTO Points |

PSM § (f)(2) annual certification of currency: signed by Document Owner annually; recorded in §17 Revision History.

### 16.2 21 CFR Part 11 Operator Records (Pharmaceutical / Biotech)

Operator-entered values (recipe parameters, setpoint adjustments, alarm acknowledgments, shift-handover notes) carry ALCOA+ data integrity:

- **Attributable** — logged with operator user-ID and timestamp.
- **Legible** — HMI displays values in clear units; printed batch records from same source.
- **Contemporaneous** — records at the time of the event (no after-the-fact entry).
- **Original** — primary record is the HMI Audit Trail (per `/fds` §13.4); printed copies are derived.
- **Accurate** — values within authorized bounds per §6.5; deviations require supervisor approval.
- **+ Complete / Consistent / Enduring / Available** — per the customer's Quality Management System.

Electronic-signature requirement per § 11.50: batch completion sign-off requires two-factor authentication. EU Annex 11 § 4 / § 9 / § 11 / § 15 parity provided by the same controls.

### 16.3 AWIA §2013 Operations (Water / Wastewater)

If population served ≥ 3,300:
- Emergency Response Plan (ERP) reference: {ERP_DOC_NUMBER}, current as of {date}, 5-year cycle.
- ERP coordination with this SOO: §10 (Emergency / E-Stop / Power Loss) operations align with ERP §X.
- Risk & Resilience Assessment (RRA): {RRA_DOC_NUMBER}, current as of {date}, 5-year cycle. This SOO contributes to RRA's automation-systems inventory.

### 16.4 3-A Sanitary Standards / FSMA Cleaning Records (Food & Beverage)

CIP cycle records per §7.6 / §9.3 retained per FSMA 21 CFR 117 Subpart C. Allergen change-over: explicit CIP-04 Final Rinse + CIP-05 Verification before next product run; verification record signed by operator and supervisor.

### 16.5 NERC CIP Operations (Power Generation)

If BES Cyber System: CIP-008 incident response triggered when operator detects:
- Unauthorized access attempt to HMI (per `/fds` §6.3 SR controls).
- Anomalous setpoint change outside §6.5 authorization bounds.
- Comms-loss patterns suggesting active intrusion.

Operator escalates to Cyber Asset Owner per CIP-008 reporting timeline.

### 16.6 NFPA 79-2024 Discrete Manufacturing

Operator-interface and stop-function references per NFPA 79 § 9 (Stop functions, Categories 0/1/2) and § 10 (Operator interface) are captured in §6.3 and §10.1.

### 16.7 Other Industry — Customer-Specified

{If Round 1 industry = Other and customer specified standards in Round 3 Q4: emit a subsection citing each customer standard, its rev, and the SOO section it constrains.}

---

## 17. Revision History

(Sole authoritative revision log.)

| Rev | Date | Author | Description |
|---|---|---|---|
| {rev} | {date} | T. Hunt | Initial draft for customer review |
````

---

### Step 4.5 — Mandatory Quality Check Before Saving

Run all rules; report results in the summary block at the end.

1. **Fault → Alarm reconciliation.** Every fault in §11.1 maps to exactly one alarm in §12.
2. **Interlock → Fault reconciliation.** Every interlock in §6.2 appears in §11.1.
3. **Mode coverage.** Every operating mode selected in Round 2 Q2 appears as a row in §5.
4. **C&E Matrix consistency.** Every initiator row in §6.4 corresponds to a fault, interlock, permissive failure, or safety trip; every action column corresponds to a §11/§12 entry.
5. **No `{X}` placeholders remain.** Every numeric placeholder (`{X}`, `{X seconds}`, `{X minutes}`) is resolved from interview Round 3 Q3 — none remain in the body.
6. **Open Items completeness.** Every `[ASSUMED from Nexus — verify with customer]` and every `[TBD — see Open Items]` flag appears as a row in §14.
7. **Setpoint table completeness.** Every row in §6.5 has Default / Bounds / Hard-Stop / Source columns populated.
8. **Standards-citation hygiene.** Every standard in §2 cites year/edition where applicable.
9. **Industry overlay.** §16 emits exactly the sub-section matching Round 1 industry (or "Not applicable" with rationale).
10. **Cold-Start vs Hot-Restart distinction.** §7.2.1 / §7.2.2 either populated or marked Not Applicable with rationale.
11. **LOTO Points table populated.** §13.3 has at least one LOTO row per energy source per equipment, or marked Not Applicable with rationale.
12. **Skills-Verified Checklist populated.** §13.4 has the eight base rows plus any industry-specific skill from §16.
13. **Glossary completeness.** Every capitalized abbreviation appearing in the body has a §15 row (auto-scan).
14. **Quick-Reference Card complete.** Three Numbers, Cold-Start, Normal Stop, Emergency Stop, Top-5 Faults populated — no `{...}` placeholders.
15. **Platform-conditional consistency.** §§7–10 emit only the block matching Round 2 platform — no Rockwell text in a Siemens SOO.
16. **Partial-Failure Operation.** §8.6 populated when scope = Equipment group / Full system; otherwise marked Not Applicable.
17. **Process Visual.** §4.3 has an ASCII diagram, not a placeholder.
18. **Timing Diagram.** §7.5 has an ASCII timing diagram populated from interview-answered timeouts.
19. **State Diagram.** §5 has the mode state-transition ASCII diagram.

Report:

```
SOO Quality Check
─────────────────────────────────
Total fault rows in §11.1:           ###
Alarms in §12:                       ### (= fault count)
Interlocks in §6.2 not in §11:       0    ← list
Modes selected vs §5 rows:           OK   ← or list gaps
C&E Matrix initiators / actions:     ### / ###  ← consistency check
Unresolved {X} placeholders:         0    ← must be 0
Nexus-ASSUMED rows in body:          ###  ← must each be in §14
TBD placeholders:                    ###  ← must each be in §14
Setpoint table rows missing fields:  0    ← list
Standards missing year/edition:      0    ← list
§16 industry overlay:                {emitted / Not applicable}
Cold-Start §7.2.1 populated:         OK / Not applicable
Hot-Restart §7.2.2 populated:        OK / Not applicable
LOTO Points §13.3 rows:              ###
Skills-Verified §13.4 rows:          ### (≥ 8)
Glossary terms missing:              0    ← list
QRC placeholders unresolved:         0    ← must be 0 before save
Platform leakage:                    0    ← wrong-platform text in body
§8.6 Partial-Failure populated:      OK / Not applicable
Process Visual §4.3:                 OK
Timing Diagram §7.5:                 OK
State Diagram §5:                    OK
```

---

### Step 5 — Finalize and Save the Document

If outputs from `/io-list`, `/bom`, `/panel-layout`, `/wire-diagram`, `/fds`, `/alarm-philosophy`, or `/hmi-spec` exist in the same save folder, parse them and use:

- **`FDS-*-Rev*.md`** — parse §1.4 RTM; populate §1.4 Traceability table here. Parse §7.6 Tag Naming Convention; use throughout §§7–11.
- **`IO-LIST-*.csv`** — cross-check tags in §11/§12 against I/O list; flag mismatches.
- **`BOM-*.csv`** — cross-reference §13.3 LOTO points and §13.5 Measurement Points to part numbers.
- **`ALARM-PHIL-*.md`** — cross-check §12 against the formal philosophy register.
- **`HMS-*-Rev*.md`** — cross-check §13.1 HMI Faceplate Elements against the HMI Spec faceplate template.

Note any cross-reference findings at the top of the saved file.

1. **Always ask where to save — never silently default to Desktop.** Use `AskUserQuestion`:

   ```
   AskUserQuestion([
     {
       question: "Where should the SOO document be saved?",
       header: "Save Location",
       multiSelect: false,
       options: [
         { label: "Active project folder — I'll type the path (Recommended)", description: "WSL path. Recommended so the SOO lives next to FDS, BOM, I/O list." },
         { label: "Same folder as other project documents — I'll type the path", description: "WSL path." },
         { label: "Current working directory", description: "Use the current shell working directory." },
         { label: "Desktop", description: "/mnt/c/Users/HIS Controls/Desktop/ (Windows Desktop via WSL — use only for one-off / scratch documents)." }
       ]
     }
   ])
   ```

2. Save the document using a status-aware filename:
   - `Status = DRAFT (Rev = Draft)` → `SOO-{project-slug}-{equipment-slug}-Draft.md`
   - `Status = DRAFT (Rev = A/B/C/...)` → `SOO-{project-slug}-{equipment-slug}-Rev{rev}-DRAFT.md`
   - `Status = FOR REVIEW` → `SOO-{project-slug}-{equipment-slug}-Rev{rev}-FOR-REVIEW.md`
   - `Status = APPROVED` → `SOO-{project-slug}-{equipment-slug}-Rev{rev}-APPROVED.md`

3. Report the full WSL file path, Quality Check summary block (from Step 4.5), word count, and counts: TBDs / Nexus-ASSUMED / Open Items rows.

---

### Step 6 — Present Next Steps Using AskUserQuestion

```
AskUserQuestion([
  {
    question: "The SOO draft is saved. What would you like to do next?",
    header: "Next Step",
    multiSelect: false,
    options: [
      { label: "Walk through Open Items (§14) one at a time", description: "Resolve each TBD via AskUserQuestion." },
      { label: "Expand fault table — add more fault conditions", description: "Targeted Fault Conditions interview." },
      { label: "Develop a specific operating mode in detail (CIP, Step, Hand)", description: "Targeted mode-by-mode buildout." },
      { label: "Export as Word/PDF for customer-ready format", description: "Convert Markdown via pandoc or similar." },
      { label: "Run /io-list to generate the I/O package from the equipment list", description: "Cross-skill handoff." },
      { label: "Customer-send-ready — finalize at the chosen Rev (FOR REVIEW)", description: "Switch status; rename file accordingly." },
      { label: "Done — this draft is ready for customer review", description: "" }
    ]
  }
])
```
