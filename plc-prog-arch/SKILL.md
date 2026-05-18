---
name: plc-prog-arch
description: Generate a Program Architecture document for a Rockwell Studio 5000 project. Covers task architecture, program/routine hierarchy, AOI instance catalog, UDT hierarchy, and tag database layout. The programmer's blueprint before a single rung is written. Follows IEC 61131-3, ISA-88, ISA-5.1, and Rockwell publication 1756-PM008.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [prog-arch, plc, studio-5000, rockwell, iec-61131, isa-88, 1756-pm008, his]
    ported_from: claude-code/prog-arch
---
# /prog-arch — Program Architecture Generator

**Invocation:** `/prog-arch [project-name]`

**Examples:**
- `/prog-arch "Wastewater Lift Station No. 4 Control Upgrade"`
- `/prog-arch "Reactor Area 200 — ControlLogix L85E"`
- `/prog-arch "Packaging Line 3 — CompactLogix Migration"`

---

## Purpose

Generate a complete Program Architecture document — the engineering blueprint that governs every task, every program, every routine, every AOI instance, and the entire tag naming convention before a single rung of ladder logic is written. PLC programmers, commissioning engineers, and customers all review this document before Studio 5000 coding begins.

Standards alignment:
- **IEC 61131-3:2013** — PLC programming languages: software model (Configuration → Resource → Task → Program → Function Block)
- **ISA-88.01-2010** — Batch control: equipment hierarchy (Site → Area → Process Cell → Unit → Equipment Module → Control Module) → maps to Studio 5000 program structure
- **ISA-5.1-2009** — Instrumentation symbols and identification: tag naming conventions applied to PLC tag database
- **Rockwell Automation 1756-PM008** — Logix 5000 Controllers Design Considerations: task architecture, priority assignment, CPU utilization targets
- **PlantPAx Process Library** — Standard AOI library for motors, valves, PID loops, analog inputs, phase management
- **IEC 62443-3-3** — Industrial cybersecurity: program-level access controls referenced in context of tag scoping and online edit discipline

---

## Instructions

### MANDATORY: AskUserQuestion for ALL Questions

Never ask questions in plain text. Every question to the user goes through `AskUserQuestion`. This is a structured interview, not a chat.

---

### Step 1 — Conduct the Project Interview

Conduct the interview in **three sequential rounds**. Complete each round fully before proceeding. Answers from early rounds determine which questions in later rounds are relevant.

---

#### Interview Round 1 — Identity and Platform (ask all 4 together)

1. **"What is the project name and HIS project number?"**
   - Header: `Project ID`
   - Options: "I'll type it" (use Other)

2. **"What controller family and catalog number is this project using?"**
   - Header: `Controller`
   - Options (single-select):
     - ControlLogix L8x — 1756-L81E, L82E, L83E, L84E, L85E (large system, up to 31 expansion chassis)
     - ControlLogix L7x — 1756-L71, L72, L73, L74, L75 (current generation, no built-in EtherNet/IP)
     - CompactLogix L3x — 1769-L30ER, L33ER, L36ERM (small to mid-size, local chassis only)
     - CompactLogix L2x — 1769-L24ER, L27ERM (small machine, limited I/O)
     - GuardLogix L8xS — 1756-L81ES, L82ES, L83ES, L84ES (safety PLC, separate safety task)
     - CompactGuardLogix L3xS — 1769-L30ERMS, L33ERMS (compact safety PLC)
     - Not yet determined / I'll describe it

3. **"What Studio 5000 firmware version is this project targeting?"**
   - Header: `Firmware`
   - Options (single-select):
     - V35.x or later (current — recommended for new projects)
     - V34.x
     - V33.x
     - V32.x or earlier (legacy — specify in Other)
     - Match existing plant standard (I'll specify in Other)
     - Not yet determined

4. **"Is there an existing PLC indexed in Nexus for this project?"**
   - Header: `Nexus Data`
   - Options (single-select):
     - Yes — existing PLC program is in Nexus; pull structure and AOI instances
     - No — this is a new project; no Nexus data
     - Not sure — check Nexus and report what you find

---

#### Interview Round 2 — Application Structure (ask all 4 together)

After Round 1 answers are received:

1. **"What type of application is this? (This drives the program decomposition model.)"**
   - Header: `Application Type`
   - Options (single-select):
     - Batch / Sequential — ISA-88 units, phases, recipes, CIP sequences (Phase Manager required)
     - Continuous Process — PID-heavy, running steady-state (reactors, utilities, water/wastewater)
     - Machine / Discrete — cycling machine, conveyor, packaging line, press (state machine or sequential)
     - Hybrid — combination of continuous loops and batch sequences
     - Infrastructure / Utility — compressor, boiler, chiller, power distribution (support systems)

2. **"List every process area or functional group in this project. For each area, list the major equipment: motors, valves, PID loops, instruments, phases. One area per line. Example: 'Area 100 — Feed Prep: 2 feed pumps (VFD), 1 strainer valve, 1 flow meter PID loop'"**
   - Header: `Equipment List`
   - Options: "I'll list them" (use Other)

3. **"What are the safety and motion requirements?"**
   - Header: `Safety & Motion`
   - Options (multi-select):
     - Standard ControlLogix only — no safety task, no motion
     - GuardLogix safety task required (SIS, E-stop zones, light curtains)
     - Servo / CNC motion — motion axes in synchronized task (Kinetix drives)
     - Safety + Motion (GuardLogix with motion)
     - No special safety beyond standard process interlocks
     - Not yet determined

4. **"How does this PLC communicate with other PLCs or systems? (Determines produced/consumed tag architecture.)"**
   - Header: `PLC Comms`
   - Options (multi-select):
     - Produced / Consumed tags — cyclic inter-PLC interlocks (same Ethernet subnet)
     - MSG blocks — infrequent data: recipes, reports, configuration
     - Both produced/consumed and MSG
     - No inter-PLC communication — standalone controller
     - OPC-UA or FactoryTalk Linx to HMI/SCADA only
     - Not yet determined

---

#### Interview Round 3 — Standards and Constraints (ask all 4 together)

After Round 2 answers are received:

1. **"Which AOI / instruction library is this project using?"**
   - Header: `AOI Library`
   - Options (single-select):
     - PlantPAx 5.x — current (requires V32+ firmware, Logix Designer 32+)
     - PlantPAx 4.x
     - PlantPAx 3.x (older — check AOI compatibility before committing)
     - Custom HIS library only — no PlantPAx
     - Mix of PlantPAx and custom AOIs
     - Not using a standard library — building from scratch
     - Not yet determined

2. **"Tag naming convention — will this project use the HIS standard or a customer standard?"**
   - Header: `Naming Convention`
   - Options (single-select):
     - HIS standard — ISA-5.1-based: [Area][EquipID]_[Function] with defined suffixes
     - Customer has an existing naming standard I'll describe
     - Match existing PLC tag database in Nexus
     - No preference — use whatever is most maintainable

3. **"Does this controller require redundancy?"**
   - Header: `Redundancy`
   - Options (single-select):
     - Simplex — single controller, no redundancy module (standard)
     - ControlLogix Redundancy — 1756-RM2 module, dual chassis (40% CPU idle required; no motion; no safety)
     - SIL-rated simplex with external safety PLC
     - Not yet determined

4. **"What is the primary deliverable intent for this document?"**
   - Header: `Intent`
   - Options (single-select):
     - New greenfield project — blank slate, build the architecture from scratch
     - Brownfield re-architecture — existing program needs restructuring; pull current structure from Nexus
     - PLC-5 / SLC-500 migration — translating legacy file-based addressing to tag-based architecture
     - Template for a repeat machine / skid — will be replicated multiple times

---

#### Round 1 Follow-Up — Document Revision and Schedule

After receiving the four Round 1 answers, ask one additional `AskUserQuestion` call to capture the document revision letter and project schedule.

1. **"Document revision letter for this issue?"**
   - Header: `Doc Rev`
   - Options (single-select):
     - A — first issue (Recommended for new projects)
     - B / C / D / later — continuing an existing Program Architecture, I'll specify in Other
     - Draft / pre-revision — internal working copy, do not issue

2. **"Project's required-on-site / FAT date?"**
   - Header: `FAT Date`
   - Options (single-select):
     - Within 8 weeks — aggressive
     - 8–16 weeks — typical
     - 16+ weeks — long lead
     - Unknown / not yet set

---

#### Conditional Follow-Up Questions (trigger based on Round 1–3 answers)

Apply these as targeted AskUserQuestion calls immediately after the relevant round:

- **If controller = GuardLogix:** Ask: "What is the safety task period (ms), how many safety E-stop zones are there, and what SIL level or Performance Level (PLr) is targeted per ISO 13849-1?"
- **If Safety & Motion includes motion:** Ask: "How many servo axes? What is the motion group synchronized task period (typically 2–8ms)? List axis names and drive catalog numbers if known."
- **If redundancy = ControlLogix Redundancy:** Immediately note in the document that CPU idle target is 40%+ (vs. 20% simplex), event tasks are not permitted, motion is not permitted, and safety is not permitted. Ask: "Is there a defined switchover time requirement (ms)?"
- **If Nexus = "Not sure":** Run `mcp__nexus__list_documented_plcs` and `mcp__nexus__plant_summary`. Report to the user what is found before continuing.
- **If Nexus = "Yes" or "Not sure" (and data found):** Run `mcp__nexus__routine_call_tree`, `mcp__nexus__aoi_instances_find`, `mcp__nexus__aoi_library_search`, `mcp__nexus__aoi_version_check`, `mcp__nexus__generate_io_inventory`, and `mcp__nexus__tag_search`. Use this data to pre-populate the program hierarchy, AOI catalog, and tag naming analysis. Flag all data `[ASSUMED from Nexus — verify with customer]`.
- **If intent = "PLC-5 / SLC migration":** Ask: "What is the legacy PLC type and program size (estimated rung count or I/O count)? Is the N-file / B-file tag mapping already documented, or does it need to be derived from the Nexus index?"
- **If application type = "Batch / Sequential" or "Hybrid":** Ask: "How many ISA-88 Units are there? Does each Unit have a Phase Manager instance, or are some Equipment Modules handled with custom state machines?"

---

### Step 2 — Pull Nexus Data (If Available)

If Nexus data exists for this project, run the following tools. **Do not present raw Nexus output to the user.** Extract relevant structure for the program hierarchy, AOI catalog, and tag naming sections.

| Tool | Purpose |
|---|---|
| `mcp__nexus__plant_summary` | Top-level plant structure — areas, PLCs, equipment counts |
| `mcp__nexus__list_documented_plcs` | Confirm which PLCs are indexed and their index completeness |
| `mcp__nexus__routine_call_tree` | Existing program → routine hierarchy (or lack thereof) — basis for re-architecture |
| `mcp__nexus__aoi_instances_find` | Every AOI instance in the existing program: AOI type, instance name, calling routine |
| `mcp__nexus__aoi_library_search` | What library AOIs are available in the indexed plant |
| `mcp__nexus__aoi_version_check` | Identify outdated AOI versions that need upgrading |
| `mcp__nexus__generate_io_inventory` | I/O list — drives equipment count for AOI Instance Catalog |
| `mcp__nexus__tag_search` | Sample tag names to analyze existing naming patterns |
| `mcp__nexus__tag_context` | Deep context on specific tags to verify naming convention compliance |

**Nexus Skepticism Rules — Non-Negotiable:**

Every piece of data sourced from Nexus MUST be flagged `[ASSUMED from Nexus — verify with customer]`. Specifically:

- Routine call trees from Nexus reflect the existing (possibly unstructured) program — they are the baseline to improve, not the target architecture
- AOI instances from Nexus may include test or template instances not in production
- Tag names from Nexus may not follow any convention — document what they ARE, not what they should be
- I/O counts from Nexus may not include recently added field devices

After collecting Nexus data, present a summary to the user with `AskUserQuestion` asking which elements of the existing structure to carry forward vs. redesign.

---

### Step 3 — Research Applicable Standards

Use Scrapling to research current program architecture standards before generating the document. Every design decision in the output must be traceable to a standard or a documented rationale.

**Mandatory research (run regardless of project type):**

1. **Rockwell 1756-PM008 Task Architecture** — search "1756-PM008 Logix 5000 task architecture periodic continuous priority recommendations" and "Rockwell Logix periodic task priority CPU utilization". Capture: recommended task period breakpoints, priority gap convention, CPU idle targets, watchdog calculation method.

2. **IEC 61131-3 Software Model** — search "IEC 61131-3 software model configuration resource task program function block". Capture: the formal hierarchy and how it maps to Studio 5000 constructs (Controller = Configuration, Controller = Resource, Task = Task, Program = Program, AOI = Function Block).

3. **ISA-88 → Studio 5000 Mapping** — search "ISA-88 equipment module control module Studio 5000 UDT program structure". Capture: how ISA-88 hierarchy levels map to Programs, AOIs, and UDTs.

4. **ISA-5.1 Tag Naming** — search "ISA-5.1 tag naming convention PLC automation instrument identification". Capture: the instrument identification system, loop tag format, and how it extends to PLC tag naming.

**Application-type-specific research (select based on Round 2 application type answer):**

| Application Type | Research Topics |
|---|---|
| Batch / Sequential | "Rockwell Phase Manager ISA-88 procedural control Studio 5000 PSC PCMD" and "PlantPAx phase faceplate parameter binding" |
| Continuous Process | "PlantPAx PID AOI PIDE instruction Studio 5000 best practices" and "Rockwell analog input scaling AOI P_AIn" |
| Machine / Discrete | "Studio 5000 state machine equipment module AOI motor starter valve" and "Rockwell safety task GuardLogix machine safety PLr" |
| Hybrid | Both batch and continuous research above |

**Industry-specific research (based on equipment descriptions from Round 2):**

| Industry Signal | Research Topics |
|---|---|
| Water / Wastewater references | "Rockwell ControlLogix water wastewater program structure best practices WWTP" |
| Pharma / Biotech references | "ISA-88 ISA-106 pharmaceutical batch control Studio 5000 21 CFR Part 11 audit trail PLC" |
| Oil and Gas references | "Rockwell ControlLogix oil gas safety PLC SIS SIL GuardLogix" |
| Food and Beverage | "PlantPAx food beverage CIP sequence Phase Manager ISA-88" |

Use `mcp__scrapling__fetch` with `disable_resources: true` and `main_content_only: true`. Use `mcp__scrapling__stealthy_fetch` only if `fetch` fails. Pull 2–3 sources maximum per topic. Capture specific standard numbers, section references, and publication numbers to cite in the document.

---

### Step 4 — Generate the Program Architecture Document

Write the complete document using the template below. Every section is mandatory. Use `[TBD — see Open Items]` for anything that cannot be determined from the interview and research. Never omit a section — write "Not applicable — [reason]" if a section genuinely does not apply to this project.

**Formatting rules:**
- Task names follow Rockwell convention: `Periodic_10ms`, `Periodic_50ms`, etc. — period in the name
- Program names are area-based, one program per unit operation: `Area_100_FeedPrep`, `Area_200_Reactor`
- Routine names are function-based: `IO_Mapping`, `Alarms`, `Interlocks`, `Permissives`, `Modes`, `Sequencing`, `PID_Loops`, `Outputs`
- AOI instance names match equipment tag: `M_101`, `XV_102`, `PID_FIC101` (underscores replace hyphens for Logix compatibility)
- UDT names use PascalCase: `MotorDriveUDT`, `ControlValveUDT`, `AnalogInputUDT`
- Tag names follow the [Area][EquipID]_[Suffix] convention (or customer standard as declared in Round 3)

---

## Program Architecture Document Template

```markdown
# Program Architecture
## {Project Name}
### {HIS Project Number} | Rev {X.X} — {Status: DRAFT / FOR REVIEW / APPROVED}

---

| Field | Value |
|---|---|
| Document Number | PROG-ARCH-{project-slug} |
| Revision | {X.X} |
| Status | DRAFT |
| Prepared By | Timothy Hunt, Hunt Integrative Solutions LLC |
| Preparation Date | {date} |
| Customer | {customer name} |
| Site | {site location} |
| Controller | {catalog number, e.g., 1756-L85E} |
| Chassis | {chassis catalog, e.g., 1756-A17, 17-slot} |
| Firmware | Studio 5000 Logix Designer V{version} |
| AOI Library | {PlantPAx version / Custom HIS Library / None} |
| Customer Approval | __________________ Date: ________ |
| HIS Approval | __________________ Date: ________ |

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
| Approved By (Customer Operations) | | | |

**This document is not valid for PLC coding until all approval signatures are obtained. Any change after approval requires a Management of Change (MOC) record and a revision to this document.**

---

## 1. Purpose and Scope

### 1.1 Purpose

This Program Architecture document defines the complete software structure for the {project name} PLC program before coding begins. It establishes:

- The task structure and CPU budget allocation
- The program and routine decomposition with execution order
- The I/O mapping strategy (centralized buffering)
- The AOI library selection and every instance to be created
- The UDT hierarchy aligned to ISA-88
- The tag naming convention and database layout
- The produced/consumed tag architecture for inter-PLC communication (if applicable)
- The Phase Manager architecture for ISA-88 procedural control (if applicable)

This document is the binding agreement between Hunt Integrative Solutions and the customer on how the PLC program is structured. PLC programmers implement exactly this structure. Changes to task periods, program decomposition, or naming conventions after approval require a formal revision.

### 1.2 Scope

**In scope:**
{List all process areas and functional groups covered by this PLC program}

**Out of scope:**
{Explicitly state what is NOT covered — adjacent PLCs, vendor-supplied packaged controls, safety PLC (if separate document), HMI tag database (see HMS document)}

### 1.3 Standards and References

| Standard / Publication | Title | Relevance to This Document |
|---|---|---|
| IEC 61131-3:2013 | Programmable Controllers — Programming Languages | Software model hierarchy: Configuration → Resource → Task → Program → Function Block |
| ISA-88.01-2010 | Batch Control Part 1: Models and Terminology | Equipment hierarchy mapped to program decomposition |
| ISA-5.1-2009 | Instrumentation Symbols and Identification | Tag naming convention — instrument identification system |
| Rockwell 1756-PM008 | Logix 5000 Controllers Design Considerations | Task architecture, priority assignments, CPU utilization targets |
| PlantPAx {version} Process Library | Rockwell Process Library AOI Reference | AOI catalog and version requirements |
| FDS-{project-slug} | Functional Design Specification | Source of truth for what the system does — this document defines how it is structured |
| IOL-{project-slug} | I/O List | Source of truth for every field signal — drives AOI instance count |
| HMS-{project-slug} | HMI Screen Specification | Tag bindings on HMI must match tag names defined here exactly |

---

## 2. Controller Hardware Overview

### 2.1 Controller Configuration

| Parameter | Value |
|---|---|
| Controller Catalog | {e.g., 1756-L85E/B} |
| Chassis | {e.g., 1756-A17/B — 17-slot} |
| Power Supply | {e.g., 1756-PA75/B} |
| Slot 0 | Controller |
| Slot 1–{N} | {I/O modules — reference IOL document for full card assignment} |
| EtherNet/IP Module | {e.g., 1756-EN2T in Slot {N}} |
| Safety Module (if GuardLogix) | {e.g., 1756-L8SP safety partner in Slot {N}} |
| Redundancy Module (if applicable) | {e.g., 1756-RM2/A in Slot {N}} |

### 2.2 Network Position

{1–2 sentences: Where this controller sits in the plant network — which VLAN, which managed switch, whether it produces/consumes to other PLCs, and the HMI server that connects to it.}

**Control network peers:**

| Peer | PLC Name | IP Address | Communication Method | Data Exchanged |
|---|---|---|---|---|
| {PLC name} | {controller tag} | {IP} | Produced/Consumed | {description} |
| {HMI Server} | N/A | {IP} | FactoryTalk Linx / OPC-UA | All controller-scoped tags |
| {add peers} | | | | |

---

## 3. Task Architecture

### 3.1 Design Basis

Per Rockwell publication 1756-PM008, all application logic uses **periodic tasks exclusively**. The Continuous task is not used. Periodic tasks provide deterministic execution at a defined rate, which is required for PID control, interlock response, and HMI data consistency.

**Priority convention:** Lower number = higher priority (Rockwell standard). Priority gaps of 2 between tasks allow future insertion without renumbering.

**CPU utilization target:**
{If simplex: 20–30% idle time minimum at peak load. If redundant: 40% idle time minimum at all times (cross-loading overhead).}

**Watchdog rule:** Each task watchdog is set to 1.5× the nominal scan time (measured at FAT under full I/O load).

### 3.2 Task Architecture Table

| Task Name | Type | Period | Priority | Watchdog | Programs Assigned | Max CPU Budget |
|---|---|---|---|---|---|---|
| Periodic_10ms | Periodic | 10 ms | 1 | 15 ms | {Motion_Group_Sync (if motion)} | {~10–15%} |
| Periodic_50ms | Periodic | 50 ms | 3 | 75 ms | IO_Mapping, {Area_100_}, {Area_200_}, {Safety (non-safety logic only)} | {~30–40%} |
| Periodic_250ms | Periodic | 250 ms | 5 | 375 ms | System, Totalizers | {~5–10%} |
| Periodic_1000ms | Periodic | 1000 ms | 7 | 1500 ms | HMI_Interface | {~5%} |
| Power_Up_Handler | Event | Power-up trigger | — | 500 ms | Power_Up_Init | One-shot |
| Module_Fault_Handler | Event | Module fault trigger | — | 500 ms | Module_Fault | One-shot |
{Add or remove tasks based on project requirements. Adjust periods if motion or safety tasks are present.}

**Notes on task periods:**
- The 10ms periodic task is only present if servo motion is included. The motion group synchronized task replaces or co-exists with it per the motion group configuration.
- PIDs are in the 50ms task. PID update time must match the task period — never put a PID in a 250ms or 1000ms task unless the loop dynamics permit it (verify with process engineer).
- Event tasks fire once per trigger and do not re-trigger. Never put continuous monitoring logic in an event task.

{If GuardLogix:}

| Safety Task | Type | Period | Watchdog | Safety Signature |
|---|---|---|---|---|
| Safety_Task | Safety Periodic | {10–50 ms} | {1.5× period} | Generated at commissioning; re-verification required after any change |

**Safety task rules (GuardLogix only):**
- Safety task is separate from all standard tasks. No cross-references between safety and standard routines.
- Safety task period and watchdog are locked by the safety signature. They cannot be changed without invalidating the signature.
- Safety tags are prefixed {SZ1_, SZ2_, etc.} per safety zone.
- Never online-edit safety logic. Safety changes require download with safety task in Program mode and re-verification of safety signature.

---

## 4. Program / Routine Hierarchy

### 4.1 Design Basis

Per ISA-88, each Unit Operation becomes one Program. Equipment Modules within a Unit become AOI instances within that program. Control Modules become AOI instances called from Equipment Module routines.

**Folder structure in Studio 5000 Controller Organizer:**

```
Tasks/
  Periodic_10ms/                     ← Only if motion present
    Motion_Sync_Group/
      Motion_Axes             ← One routine per axis group
      Motion_Coordinated      ← Coordinated motion, if applicable

  Periodic_50ms/
    IO_Mapping/                ← CENTRALIZED I/O BUFFER — all raw I/O mapped here
      IO_Map_Inputs            ← All :I.Data reads → buffer tags
      IO_Map_Outputs           ← All buffer tags → :O.Data writes

    {Area_100_FeedPrep}/       ← One program per Unit
      Alarms                   ← Alarm rungs for all equipment in this area
      Interlocks               ← Interlock logic and permissive evaluation
      Permissives              ← Permissive ladder: one rung per permissive per device
      Modes                    ← Area mode state machine (Auto/Manual/Step/Hold)
      Sequencing               ← Step logic, Phase Manager calls (if batch)
      PID_Loops                ← All PID blocks for this area
      Outputs                  ← Final output coils — driven by AOI outputs + interlocks

    {Area_200_Reactor}/        ← Repeat pattern for each area
      Alarms
      Interlocks
      Permissives
      Modes
      Sequencing
      PID_Loops
      Outputs

    {Add one folder/program per process area or functional group}

  Periodic_250ms/
    System/
      PLC_Health               ← GSV: task scan times, memory, controller status
      Comms_Watchdog           ← Heartbeat monitoring for produced/consumed peers
      Module_Status            ← GSV: I/O module fault and inhibit status
    Totalizers/
      Flow_Totals              ← Flow integration, batch quantity totals
      Energy_Totals            ← kWh, BTU, etc.
      Runtime_Totals           ← Equipment run hours

  Periodic_1000ms/
    HMI_Interface/
      HMI_Handshake            ← Heartbeat bit, HMI watchdog
      Navigation_Tags          ← Screen navigation request/confirm bits
      Recipe_Interface         ← Recipe parameter staging (if applicable)

  Power_Up_Handler/
    Power_Up_Init              ← One-shot: clear latched faults, initialize mode tags

  Module_Fault_Handler/
    Module_Fault               ← Log module fault code, set area fault summary bit

  Safety_Task/ (GuardLogix only)
    {SZ1_EStop_Zone_1}/        ← One safety program per E-stop zone
      SZ1_Inputs               ← Safety input evaluation (dual-channel)
      SZ1_Logic                ← Safety function logic (ESTOP, DCS, CROUT instructions)
      SZ1_Outputs              ← Safety output commands
    {Add safety programs per zone}
```

### 4.2 Program Detail — {Area_100_FeedPrep} (Repeat for Each Area)

{Fill in one table per area program. This is the level of detail PLC programmers work from.}

**Program scope:** {1–2 sentences describing what this program controls.}
**Task assignment:** Periodic_50ms
**ISA-88 level:** Unit — {unit name}

| Routine | Execution Order | Description | Key Instructions |
|---|---|---|---|
| Alarms | 1 | Alarm detection rungs for all devices in this area. One rung per alarm tag. Drives _Alm suffix tags. | XIC, OTE, TON (delay on alarms) |
| Interlocks | 2 | Process interlocks that shut down or hold equipment. Evaluated every scan. References permissive bits from Interlocks routine. | XIC, XIO, OTE, OTL/OTU |
| Permissives | 3 | One rung per device per permissive condition. Output is a permissive bit (_Perm tag) consumed by the AOI InOut parameter. | XIC, XIO, OTE |
| Modes | 4 | Area mode state machine. Sets area Auto/Manual/Step/Hold mode based on operator command and equipment state. | JSR to sub-routines, or inline state machine with MOV/CMP |
| Sequencing | 5 | Step sequencer or Phase Manager calls. Only present if application type is batch or has startup/shutdown sequences. | PSC, PCMD, PCLF (Phase Manager) or custom step timer logic |
| PID_Loops | 6 | All PIDE instructions for this area. One rung per PID loop. Setpoint, PV, and output tag connections explicit. | PIDE |
| Outputs | 7 | Final output coils. AOI .Cmd outputs ANDed with interlock bits → final output tags. Buffer tags → IO_Mapping writes actual hardware outputs. | XIC, OTE, OTL/OTU |

**Program-scoped tags (Public parameter access):**

| Tag Name | Data Type | Direction | Description |
|---|---|---|---|
| {Area}_AutoCmd | BOOL | Input (from HMI_Interface) | HMI command to enter Auto mode |
| {Area}_HoldCmd | BOOL | Input (from HMI_Interface) | HMI command to hold sequence |
| {Area}_FaultSummary | BOOL | Output (to HMI_Interface) | Any fault active in this area |
| {Area}_RunSts | BOOL | Output (to HMI_Interface) | Area is in Auto and running |
| {add program parameters per area} | | | |

---

## 5. I/O Mapping Strategy

### 5.1 Design Basis

Per Rockwell 1756-PM008 and HIS standard, raw I/O addresses (`Local:1:I.Data.0`, `Local:3:O.Data`) are referenced in **exactly one location** — the IO_Mapping program in the Periodic_50ms task. Every other program reads from and writes to **buffer tags** with process-meaningful names.

This ensures:
- Every I/O wiring change is a one-rung edit in one known location
- Commissioning I/O checks are done in one program
- Area programs are controller-independent (portable between projects)
- No `Local:X:I` references scattered across 20 programs

### 5.2 I/O Mapping Routine Structure

**IO_Map_Inputs routine** — runs first in Periodic_50ms:

```
Rung format:
[NOP or XIC always-true bit]   [MOV Local:{slot}:I.Data.{word}  →  {Buffer_Tag}]

Example:
[XIC Always_True]  [MOV Local:3:I.Data   →  AI_A100_RawInputs]
[XIC Always_True]  [MOV Local:2:I.Data.0 →  DI_A100_Inputs_W0]
```

**IO_Map_Outputs routine** — runs last in Periodic_50ms (after all area programs):

```
Rung format:
[XIC Always_True]  [MOV  {Buffer_Tag}  →  Local:{slot}:O.Data]
```

### 5.3 Buffer Tag Naming Convention

| Buffer Tag Format | Example | Description |
|---|---|---|
| `AI_{Area}_{ModuleDescriptor}` | `AI_A100_RawPressure` | Analog input buffer for one module or group |
| `AO_{Area}_{ModuleDescriptor}` | `AO_A100_ValveOutputs` | Analog output buffer |
| `DI_{Area}_{ModuleDescriptor}_W{N}` | `DI_A100_Inputs_W0` | Digital input word buffer (W0, W1, ... per 16-bit word) |
| `DO_{Area}_{ModuleDescriptor}_W{N}` | `DO_A100_Outputs_W0` | Digital output word buffer |

**Individual bit extraction** (after the word is buffered):

```
After MOV DI_A100_Inputs_W0, extract individual bits:
[XIC DI_A100_Inputs_W0.0]  [OTE M_101_RunFB]
[XIC DI_A100_Inputs_W0.1]  [OTE XV_101_OpenFB]
```

**Rule:** Every bit in every buffer word must have a named extraction rung. No `DI_A100_Inputs_W0.5` references in area logic — always reference the named tag.

---

## 6. AOI Instance Catalog

### 6.1 AOI Library Summary

| Library | Version | Source | Usage in This Project |
|---|---|---|---|
| PlantPAx Process Library | {version} | Rockwell Automation | Motors, valves, PID loops, analog inputs, phase management |
| HIS Custom AOI Library | {version} | Hunt Integrative Solutions | {custom patterns not covered by PlantPAx, e.g., specialized pump control, custom totalizer} |

**PlantPAx AOIs used in this project:**

| AOI Name | PlantPAx Description | Version | Instances in This Project |
|---|---|---|---|
| P_Motor | Induction motor — DOL/VFD, OL protection, run/stop, permissives | {version} | {N} |
| P_Valve | On/Off or modulating control valve, travel time monitoring | {version} | {N} |
| P_PIDE | PID control with bumpless auto/manual transfer, anti-windup, cascade | {version} | {N} |
| P_AIn | Analog input scaling, EU conversion, alarm limit monitoring | {version} | {N} |
| P_Phase | ISA-88 Phase Manager interface faceplate binding | {version} | {N} |
| P_Alarm | Alarm management: latching, shelving, ISA-18.2 priority | {version} | {N} |
| {add AOIs as needed} | | | |

### 6.2 AOI Instance Table

One row per equipment instance. Every piece of equipment from the I/O List that requires control logic has an entry here.

| Instance Name | AOI Type | Description | Calling Program | Calling Routine | UDT Instance Tag | PLC Tag Prefix |
|---|---|---|---|---|---|---|
| M_101 | P_Motor | Feed pump No. 1 — 15HP VFD | Area_100_FeedPrep | Outputs | Area100.M101 | PM100_M101 |
| M_102 | P_Motor | Feed pump No. 2 — 15HP VFD | Area_100_FeedPrep | Outputs | Area100.M102 | PM100_M102 |
| XV_101 | P_Valve | Feed isolation valve — on/off | Area_100_FeedPrep | Outputs | Area100.XV101 | PM100_XV101 |
| FIC_101 | P_PIDE | Feed flow control loop — FCV-101 | Area_100_FeedPrep | PID_Loops | Area100.FIC101 | PM100_FIC101 |
| FIT_101 | P_AIn | Feed flow transmitter | Area_100_FeedPrep | IO_Map_Inputs | Area100.FIT101 | AI_A100_FIT101 |
| {add all equipment instances} | | | | | | |

**AOI instance count summary:**

| AOI Type | Total Instances | CIP Connection Impact |
|---|---|---|
| P_Motor | {N} | None (no additional CIP connections) |
| P_Valve | {N} | None |
| P_PIDE | {N} | None |
| P_AIn | {N} | None |
| {Custom AOI with comms} | {N} | {Note if MSG inside AOI consumes a CIP connection} |

---

## 7. UDT Hierarchy

### 7.1 Design Basis

UDTs model the ISA-88 equipment hierarchy in the PLC tag database. Nesting provides natural scoping: the Area UDT contains all Equipment Module UDTs; each Equipment Module UDT contains its Control Module UDTs (motor, valve, instrument).

**BOOL packing rule:** BOOLs in UDTs are packed into DINT arrays (32 bits each). Individual BOOLs scattered among REALs waste memory due to 32-bit alignment. Group all status bits into one or more DINT, then use `.0`, `.1`, `.2` notation.

**InOut vs Input rule:** UDTs larger than 32 bytes are passed to AOIs as InOut parameters (by reference). Passing by value copies the entire structure every scan.

### 7.2 UDT Definitions

#### Area-Level UDT (one instance per area, controller-scoped)

```
AreaStatusUDT
  .RunSts          BOOL    — Area is running in Auto
  .FaultSummary    BOOL    — Any device in area is faulted
  .AlarmSummary    BOOL    — Any alarm active in area
  .ModeState       INT     — 0=Idle, 1=Auto, 2=Manual, 3=Hold, 4=Abort
  .ActiveFaults    DINT    — Bit-packed fault summary (32 fault bits)
  .ActiveAlarms    DINT[2] — Bit-packed alarm summary (64 alarm bits)
```

#### Equipment Module UDT (one instance per major equipment group)

```
MotorDriveUDT
  .RunFB           BOOL    — Run feedback from field (hardwired DI)
  .FaultSts        BOOL    — Fault condition (OL trip, drive fault, etc.)
  .ReadySts        BOOL    — Drive ready for start command
  .RunCmd          BOOL    — Run command output to drive/contactor
  .StopCmd         BOOL    — Stop command
  .FaultReset      BOOL    — Fault reset momentary
  .ModeCmd         INT     — 0=Auto, 1=Manual
  .SpeedSP         REAL    — Speed setpoint (Hz or RPM, VFD only)
  .SpeedFB         REAL    — Speed feedback (VFD only)
  .RunHours        REAL    — Accumulated run hours
  .StartCount      DINT    — Total start count
  .PermBits        DINT    — Bit-packed permissives (up to 32)
  .FaultMsg        STRING  — Active fault description for HMI display
```

```
ControlValveUDT
  .OpenFB          BOOL    — Open limit switch feedback
  .CloseFB         BOOL    — Close limit switch feedback
  .FaultSts        BOOL    — Fault (travel timeout, discrepancy)
  .OpenCmd         BOOL    — Open output
  .CloseCmd        BOOL    — Close output
  .PositionFB      REAL    — Position feedback 0–100% (modulating only)
  .PositionSP      REAL    — Position setpoint (modulating only)
  .ModeCmd         INT     — 0=Auto, 1=Manual
  .TravelTimer     REAL    — Elapsed travel time (seconds)
  .PermBits        DINT    — Bit-packed permissives
  .FaultMsg        STRING  — Active fault description
```

```
AnalogInputUDT
  .RawValue        INT     — Raw counts from A/D (0–32767)
  .PV              REAL    — Scaled engineering unit value
  .HHAlm           BOOL    — High-High alarm active
  .HAlm            BOOL    — High alarm active
  .LAlm            BOOL    — Low alarm active
  .LLAlm           BOOL    — Low-Low alarm active
  .BadQuality      BOOL    — Signal quality failure (out-of-range, broken wire)
  .HHLimit         REAL    — High-High alarm setpoint
  .HLimit          REAL    — High alarm setpoint
  .LLimit          REAL    — Low alarm setpoint
  .LLLimit         REAL    — Low-Low alarm setpoint
  .EU              STRING  — Engineering units label (PSI, GPM, °F, %)
```

{Add UDT definitions for all equipment types in this project: VFD, Actuator, PhaseUDT, etc.}

### 7.3 Controller-Scoped UDT Instance Tags

| Tag Name | UDT Type | Description | Scope |
|---|---|---|---|
| Area100 | {Area100UDT} | All equipment data for Area 100 | Controller |
| Area200 | {Area200UDT} | All equipment data for Area 200 | Controller |
| SysStatus | {SystemStatusUDT} | PLC health, comms, task scan times | Controller |
| {add controller-scoped tags} | | | |

**Note:** Area UDTs are controller-scoped because the HMI and other programs need cross-area visibility. Program-scoped tags are used only for tags that are strictly internal to one program and have no HMI binding.

---

## 8. Tag Naming Convention

### 8.1 Convention Format

```
[Area][EquipID]_[Suffix]
```

| Component | Rule | Examples |
|---|---|---|
| Area | 2–4 character area code | PM (Prep/Mix), RX (Reactor), UT (Utilities), CW (Cooling Water) |
| EquipID | ISA-5.1 instrument ID: function letters + loop number | M101, XV102, FIC103, PIT104 |
| Suffix | Functional suffix (see table below) | _RunFB, _FaultSts, _RunCmd |

**Full tag example:** `PM100_M101_RunFB` — Area PM100, Motor 101, Run Feedback

### 8.2 Suffix Table (Applies to All Equipment Types)

| Suffix | Meaning | Data Type | Typical Source |
|---|---|---|---|
| `_Cmd` | Command output from PLC to field device | BOOL | PLC output (via buffer tag) |
| `_FB` | Field feedback from device to PLC | BOOL | PLC input (via buffer tag) |
| `_Sts` | Computed status (logic-derived, not raw input) | BOOL | Logic in area program |
| `_Alm` | Alarm bit (active = alarm condition present) | BOOL | Alarm routine |
| `_AlmAck` | Alarm acknowledged | BOOL | HMI acknowledge command |
| `_SP` | Setpoint (operator-entered or recipe) | REAL | HMI write or recipe |
| `_PV` | Process Variable (measured value) | REAL | Scaled analog input |
| `_Out` | Controller output (PID output, drive speed reference) | REAL | PID or sequencer |
| `_Perm` | Permissive bit (TRUE = permissive satisfied) | BOOL | Permissives routine |
| `_Inh` | Inhibit — suppress alarm without acknowledging | BOOL | Supervisor command |
| `_Ovr` | Override — force output regardless of auto logic | BOOL | Manual override command |
| `_Flt` | Fault latch | BOOL | Interlock or hardware fault |
| `_Mode` | Mode selector (0=Auto, 1=Manual, 2=Step) | INT | Operator command |
| `_Step` | Current step number in sequence | INT | Sequencer |
| `_Timer` | Timer accumulator or preset | REAL / TIMER | Sequencer or interlock |

### 8.3 Scope Decision Matrix

| Tag Category | Scope | Rationale |
|---|---|---|
| Field I/O buffer tags (DI_*, DO_*, AI_*, AO_*) | Controller | IO_Mapping program reads and writes these; area programs consume them |
| Equipment UDT instances (Area100.M101) | Controller | HMI and multiple programs need visibility |
| PID tags | Controller | Historian needs direct access; faceplate uses controller-scoped binding |
| Area status summary tags | Controller | HMI Plant Overview reads all area statuses simultaneously |
| Internal sequencer step tags | Program (if no HMI binding needed) | Reduces tag browser clutter |
| Alarm detail strings | Controller | HMI alarm screen needs them |
| Totalizer tags | Controller | Historian needs direct access |
| HMI handshake bits | Program (HMI_Interface) | Only HMI_Interface program exchanges these with HMI |

### 8.4 Tag Database Sample — Area 100 (First Two Levels)

```
Controller Tags/
  DI_A100_Inputs_W0        DINT    — Raw DI word: Motor run FBs, valve limit switches
  DI_A100_Inputs_W1        DINT    — Raw DI word: pressure switches, float switches
  DO_A100_Outputs_W0       DINT    — Raw DO word: motor run commands, valve commands
  AI_A100_FIT101           REAL    — Scaled: feed flow, GPM
  AI_A100_PIT102           REAL    — Scaled: feed pressure, PSI
  AO_A100_FCV101           REAL    — Analog output: feed control valve position, %
  Area100/
    M101/                  MotorDriveUDT
      .RunFB
      .FaultSts
      .RunCmd
      .SpeedFB
      ... (all UDT members)
    XV101/                 ControlValveUDT
      .OpenFB
      .CloseFB
      .OpenCmd
      ...
    FIC101/                {PID_UDT or PIDE tag structure}
      .PV
      .SP
      .Output
      .Kp, .Ki, .Kd
      ...
    FIT101/                AnalogInputUDT
      .PV
      .HAlm
      .LAlm
      ...
  SysStatus/               SystemStatusUDT
    .Task50ms_ScanTime     REAL
    .Task50ms_MaxScan      REAL
    .Task50ms_Overlap      DINT
    .ControllerFaultCode   DINT
    .PLCCommsOK            BOOL
```

---

## 9. Produced / Consumed Tag Architecture

{Include this section only if inter-PLC communication via produced/consumed tags was declared in Round 2. Otherwise write "Not applicable — this controller has no produced/consumed tag communication with peer PLCs."}

### 9.1 Design Basis

Produced/consumed tags provide deterministic, cyclic inter-PLC data exchange over EtherNet/IP. They are used for data that must be current on every scan: interlocks, safety-adjacent signals, and continuous process values needed by the consuming controller for its own control logic.

MSG blocks are used for infrequent transfers: recipes, reports, batch records. See FDS document for the MSG communication matrix.

### 9.2 Produced Tag Table (This Controller → Peers)

| Produced Tag Name | Data Type | RPI (ms) | Consumers | Description |
|---|---|---|---|---|
| {TagName}_Produced | {UDT / DINT / REAL} | {20–100 ms} | {PLC_Name} | {what this data represents} |
| {add produced tags} | | | | |

**RPI selection rule:** RPI = 2–4× the consuming task's period. Example: if the consuming PLC has a 50ms task, set RPI to 100ms. Lower RPI = more network bandwidth consumed on every scan regardless of value change.

### 9.3 Consumed Tag Table (Peer PLCs → This Controller)

| Consumed Tag Name | Producing PLC | Data Type | RPI (ms) | Description | Comms Watchdog Tag |
|---|---|---|---|---|---|
| {TagName}_From_{PLC} | {PLC_Name} | {type} | {RPI} | {description} | {WatchdogBit} |
| {add consumed tags} | | | | | |

### 9.4 CIP Connection Budget

| Module | CIP Connection Limit | Connections Used | Connections Remaining |
|---|---|---|---|
| {1756-EN2T in Slot N} | 256 (typical) | {count produced + consumed + MSG} | {remaining} |

**Rule:** Never exceed 80% of module CIP connection limit. Reserve the remainder for online programming connections and MSG blocks. Check actual limit for firmware version — consult module release notes.

---

## 10. Phase Manager Architecture

{Include this section only if application type is Batch, Sequential, or Hybrid. Otherwise write "Not applicable — this project does not use ISA-88 procedural control or Phase Manager."}

### 10.1 Design Basis

Rockwell Phase Manager implements ISA-88 procedural control natively in Studio 5000. Each Equipment Module or Unit Operation that has sequential steps gets one Phase Manager instance. The Phase Manager engine handles state transitions; application logic in each state is written by the programmer in the corresponding state routine called by PSC.

### 10.2 Phase Instances

| Phase Instance Name | ISA-88 Level | Associated Unit / EM | Calling Program | States Used | Phase Parameter Tag |
|---|---|---|---|---|---|
| {Phase_Name} | Unit / Equipment Module | {unit or EM name} | {Program} | Idle, Running, Complete, Holding, Held, Stopping, Stopped, Aborting, Aborted, Failure | {Phase_Tag_Name} |
| {add one row per Phase Manager instance} | | | | | |

### 10.3 Standard Phase State Sequence

```
Idle
  │ PSC (Start command)
  ▼
Running ──────────────────────────────────────► Complete
  │                                               │
  │ PATT (Hold command)                          │ (next phase or done)
  ▼                                               │
Holding ──► Held ──► (Restart) ──► Running       │
                                                  │
  │ PCMD (Stop)                                   │
  ▼                                               │
Stopping ──► Stopped                             │
                                                  │
  │ PFL (Failure / E-stop)
  ▼
Aborting ──► Aborted
               │ PCLF (Clear Failure)
               ▼
             Idle
```

### 10.4 Phase Parameter Structure

Each Phase Manager instance has a phase parameter tag of type PHASE (built-in Logix data type). The following application-specific parameters are added via the phase definition:

| Parameter | Data Type | Direction | Description |
|---|---|---|---|
| {PhaseParam_1} | REAL | Input | {e.g., target fill volume, liters} |
| {PhaseParam_2} | REAL | Input | {e.g., target temperature setpoint, °F} |
| {PhaseParam_Result} | REAL | Output | {e.g., actual volume dispensed} |
| {add parameters per phase} | | | |

**HMI binding:** Phase state is exposed to the HMI via the PlantPAx P_Phase AOI faceplate (SCR-FP-PHS per the HMS document). The phase tag is the InOut parameter of the P_Phase AOI.

---

## 11. Open Items and TBDs

Items that could not be confirmed during document preparation. All must be resolved before the Program Architecture is approved and coding begins.

| # | Section | Item Description | Owner | Target Date |
|---|---|---|---|---|
| TBD-001 | {section} | {description} | {Customer / HIS} | {date} |
| | | | | |

---

## 12. Glossary

| Term | Definition |
|---|---|
| AOI | Add-On Instruction — reusable function block in Studio 5000 (equivalent to IEC 61131-3 Function Block) |
| CIP | Common Industrial Protocol — EtherNet/IP application layer protocol |
| Control Module | ISA-88: lowest equipment hierarchy level — one physical device (motor, valve, transmitter) |
| CPU Budget | Maximum percentage of controller CPU time allocated to a task, leaving the remainder idle |
| DINT | Double Integer — 32-bit signed integer data type in Logix; used for BOOL packing |
| Equipment Module | ISA-88: group of Control Modules that form a functional piece of equipment (e.g., pump + valve + flow meter) |
| IEC 61131-3 | International standard for PLC programming languages and software model |
| InOut | AOI parameter passing method — passed by reference (pointer), not by value. Used for UDTs > 32 bytes. |
| IO_Mapping | The centralized program that reads all raw I/O addresses into buffer tags and writes buffer tags back to output addresses |
| ISA-88 | ANSI/ISA-88 batch control standard; the source of the equipment hierarchy model used in this document |
| ISA-5.1 | ANSI/ISA-5.1 instrumentation symbols standard; the source of the tag naming convention |
| PCMD | Phase Command instruction — accepts external commands (Start, Hold, Restart, Stop, Abort, Reset) into a phase |
| PIDE | Enhanced PID instruction in Studio 5000 — used for all closed-loop control |
| PlantPAx | Rockwell Automation process library — standard AOIs for motors, valves, PID, and phase management |
| PSC | Phase State Complete instruction — signals that the current phase state is done; triggers transition |
| RPI | Requested Packet Interval — the cyclic rate at which produced/consumed tags are transmitted |
| Safety Signature | Cryptographic hash of safety task logic and configuration; generated at commissioning; invalidated by any change |
| UDT | User-Defined Data Type — structured data type in Studio 5000, equivalent to IEC 61131-3 structure |
| Unit | ISA-88: the mandatory core of the hierarchy — a collection of Equipment Modules that can perform a batch operation independently |
```

---

### Step 5 — Finalize and Save the Document

1. Replace ALL `{placeholder}` fields with actual values from the interview and research.
2. Verify every area listed in the equipment interview (Round 2) has a corresponding program entry in the Program Hierarchy (Section 4) and at least one AOI instance in Section 6.
3. Verify every AOI instance in Section 6 references a UDT type defined in Section 7.
4. Verify every tag naming example in Sections 8.4 follows the suffix rules in Section 8.2 exactly.
5. Verify the task table (Section 3) has a CPU budget column that sums to ≤ 80% (simplex) or ≤ 60% (redundant), leaving the required idle time.
6. If GuardLogix: verify the safety task section is complete, safety zone count matches interview, and the non-editable safety task period matches the SIL/PL requirement.
7. Count all `[TBD]` items. List every one in Section 11 (Open Items).
8. **Always ask where to save — never silently default to Desktop.** Use `AskUserQuestion`:

   **"Where should the Program Architecture document be saved?"**
   - Header: `Save Location`
   - Options (single-select):
     - Active project folder — I'll type the path (Recommended)
     - Same folder as the FDS or I/O List for this project — I'll type the path
     - Current working directory
     - Desktop — `/mnt/c/Users/HIS Controls/Desktop/` (Windows Desktop via WSL — use only for one-off / scratch documents)

   If outputs from `/io-list`, `/bom`, `/fds`, or `/soo` exist in the same save folder, mention them in the document's Reference Documents table.

9. Save the document as: `PROG-ARCH-{project-slug}-Rev{rev}-DRAFT.md` in the location the user specified, using the user-supplied revision letter from the Round 1 follow-up.
10. Report the full file path, total AOI instance count, task count, and TBD count to the user.

---

### Step 6 — Present Next Steps Using AskUserQuestion

After saving, use `AskUserQuestion`:

**"The Program Architecture draft is saved. What would you like to do next?"**
- Header: `Next Step`
- Options (single-select):
  - Review the task architecture — walk through CPU budget and scan rate assignments
  - Review the AOI instance catalog — verify every equipment instance is listed
  - Review the UDT hierarchy — confirm data structure for a specific equipment type
  - Review the tag naming convention — walk through example tags for each area
  - Resolve open items — work through TBD items one by one
  - Generate the I/O List — run /io-list using this architecture as the basis
  - Begin coding — export a Studio 5000 L5X project shell with this structure pre-built
  - Done — this draft is ready for customer review

---

## Quality Rules

Before considering the Program Architecture complete, verify:

- [ ] Every process area from the interview has a program in the hierarchy (Section 4)
- [ ] Every program has all 7 standard routines (Alarms, Interlocks, Permissives, Modes, Sequencing, PID_Loops, Outputs) or "Not applicable" notation
- [ ] The IO_Mapping program exists in Periodic_50ms and is the only location where raw I/O addresses appear
- [ ] Every AOI instance in Section 6 has: instance name, AOI type, calling program, calling routine, UDT path, and tag prefix
- [ ] The AOI instance count matches (or is traceable to) the I/O list equipment count
- [ ] Every UDT referenced in Section 6 is defined in Section 7
- [ ] The suffix table in Section 8.2 covers every suffix used in the tag examples in Section 8.4
- [ ] The task CPU budget column sums to ≤ 80% total — leaving required idle time
- [ ] Watchdog values in the task table are 1.5× the nominal period
- [ ] If redundancy is declared: CPU idle target is ≥ 40%, no motion task, no safety task, no event tasks
- [ ] If produced/consumed tags are used: CIP connection budget table is filled and headroom is ≥ 20%
- [ ] If GuardLogix: safety task section is present, safety tag naming prefix is defined, and online edit prohibition is stated
- [ ] If Phase Manager is used: one Phase Manager instance per Equipment Module is confirmed
- [ ] All `[ASSUMED from Nexus]` items are listed in Section 11 (Open Items)
- [ ] Document reads clearly to a PLC programmer who is joining the project mid-stream
