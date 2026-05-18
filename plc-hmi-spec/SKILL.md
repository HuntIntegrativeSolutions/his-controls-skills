---
name: plc-hmi-spec
description: Generate a complete HMI Screen Specification for an industrial automation project. Covers screen inventory, navigation map, per-screen content definitions, security levels (ISA-101), color philosophy, alarm display (ISA-18.2), and faceplate templates. Works for FactoryTalk View SE/ME and Ignition SCADA. Researches applicable standards via Scrapling before generating the document.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [hmi, scada, factorytalk, ignition, isa-101, isa-18.2, his]
    ported_from: claude-code/hmi-spec
---
# /hmi-spec — HMI Screen Specification Generator

**Invocation:** `/hmi-spec [project-name]`

**Examples:**
- `/hmi-spec "Wastewater Lift Station No. 4 Control Upgrade"`
- `/hmi-spec "Reactor Area 200 — FactoryTalk View SE"`
- `/hmi-spec "Packaging Line 3 — Ignition Perspective"`

---

## Purpose

Generate a complete HMI Screen Specification document — the engineering blueprint that governs every screen, every navigation path, every user role, and every display element before a single pixel is drawn. This document defines WHAT appears on each screen and WHO can interact with it. HMI developers, PLC engineers, and customers all sign off on this document before HMI development begins.

Standards alignment:
- **ANSI/ISA-101.01-2015** — Human Machine Interfaces: Models and Terminology; Design, Implementation, and Evaluation
- **ISA-18.2-2016** — Management of Alarm Systems (alarm display, alarm summary, alarm banner)
- **ISA-5.1-2009** — Instrumentation Symbols and Identification (symbol conventions on process graphics)
- **IEC 62443-2-1 / -3-3** — Industrial cybersecurity: user authentication, role-based access control
- **NIST SP 800-82 Rev. 2** — Guide to ICS Security (HMI access control guidance)
- Platform-specific standards researched via Scrapling based on HMI platform selected

---

## Instructions

### MANDATORY: AskUserQuestion for ALL Questions

Never ask questions in plain text. Every question to the user goes through `AskUserQuestion`. This is a structured interview, not a chat.

---

### Step 1 — Conduct the Project Interview

Conduct the interview in **three sequential rounds**. Complete each round fully before proceeding. Answers from early rounds determine which questions in later rounds are relevant.

---

#### Interview Round 1 — Project Identity and Platform (ask all 4 together)

1. **"What is the project name and HIS project number?"**
   - Header: `Project ID`
   - Options: "I'll type it" (use Other)

2. **"What HMI platform is being used?"**
   - Header: `HMI Platform`
   - Options (single-select):
     - FactoryTalk View SE — server-based, multi-client (plant/area SCADA)
     - FactoryTalk View ME — standalone panel HMI (PanelView Plus)
     - Ignition 8.x — Perspective (web/mobile) or Vision (thick client)
     - Ignition Perspective (web-first, mobile-capable)
     - Ignition Vision (thick client, legacy Ignition)
     - Other / Multiple (I'll describe it)

3. **"What is the scope of the HMI — how many process areas does it cover?"**
   - Header: `HMI Scope`
   - Options (single-select):
     - Single machine or skid — one unit, one panel
     - Single process area — one area with multiple units
     - Multiple areas — plant-wide SCADA with area overview
     - Full plant — all areas plus utility and infrastructure screens

4. **"Is there an existing HMI project indexed in Nexus for this system?"**
   - Header: `Nexus HMI Data`
   - Options (single-select):
     - Yes — existing HMI screens are in Nexus
     - No — this is a new HMI, nothing in Nexus
     - Not sure — check Nexus and tell me what you find

---

#### Interview Round 2 — Process Areas and Equipment (ask all 4 together)

After Round 1 answers are received:

1. **"List every process area or unit that needs its own screens. Include the area name and the major equipment in each area (e.g., 'Area 100 — Feed Prep: two feed pumps, one strainer, one flow meter')."**
   - Header: `Process Areas`
   - Options: "I'll list them" (use Other)

2. **"What types of HMI screens does this project require? Select all that apply."**
   - Header: `Screen Types`
   - Options (multi-select):
     - Plant / Area Overview (top-level status, all areas visible)
     - Process Graphics / P&ID Views (equipment, valves, instruments on process background)
     - Equipment Faceplates (motor, valve, PID, phase — pop-up or dedicated screen)
     - Trend / History screens (real-time and historical trend displays)
     - Alarm Summary screen (active alarms, alarm history, acknowledgment)
     - Recipe / Parameter management screens
     - Reports / Totalization screens
     - Diagnostic / Maintenance screens (I/O status, comms health, tag browser)
     - Security / User Management screens
     - Navigation / Home screen

3. **"What control platform does the PLC run on? (Determines tag address format and faceplate data binding.)"**
   - Header: `PLC Platform`
   - Options (single-select):
     - Rockwell ControlLogix / CompactLogix (Studio 5000, UDT-based tags)
     - Rockwell PLC-5 or SLC 500 (legacy N-file / B-file addressing)
     - Ignition soft tag / OPC-UA (no direct PLC binding)
     - Siemens S7 (DB addressing)
     - Other / Mixed (I'll describe)

4. **"Who are the user roles that will interact with this HMI? Select all that apply."**
   - Header: `User Roles`
   - Options (multi-select):
     - Operator — runs the process, starts/stops equipment, acknowledges alarms
     - Supervisor — all operator rights plus setpoint changes and alarm shelving
     - Engineer — all supervisor rights plus tuning parameters, recipe editing, and forced overrides
     - Maintenance — specialized access to diagnostic and I/O force screens
     - Administrator — full system access including security configuration
     - Read-Only / Observer — view-only, no commands

---

#### Interview Round 3 — Security, Standards, and Constraints (ask all 4 together)

After Round 2 answers are received:

1. **"What security authentication method does the customer require for the HMI?"**
   - Header: `Authentication`
   - Options (single-select):
     - Windows-linked accounts (FactoryTalk Security integrated with AD)
     - Local HMI user accounts (username/password managed in HMI project)
     - Ignition role-based security (Identity Provider — Ignition native or AD/LDAP)
     - Badge/card reader (physical authentication integrated with HMI login)
     - No authentication required — open access (single-user kiosk or standalone machine)
     - Not yet determined

2. **"What display resolution and target screen size is the HMI running on?"**
   - Header: `Display Spec`
   - Options (single-select):
     - 1920×1080 (full HD) — 21"–27" industrial monitor
     - 1280×1024 — 19" standard industrial monitor
     - 1024×768 — 15" PanelView or legacy monitor
     - Mobile / Responsive — Ignition Perspective, varies by device
     - Multiple resolutions — I'll describe the mix
     - Not yet determined

3. **"Are there any customer HMI standards or existing style guides that must be followed?"**
   - Header: `Customer Standards`
   - Options (single-select):
     - Yes — customer has an HMI standard (I'll describe it or provide a file)
     - No — use HIS standards (ISA-101 compliant, gray background, alarm-only color)
     - Partially — customer has preferences but no formal standard
     - Not sure — use ISA-101 as baseline

4. **"What is the primary language for the HMI? Are multiple languages required?"**
   - Header: `Language`
   - Options (single-select):
     - English only
     - English primary with Spanish secondary
     - English primary with another language (I'll specify)
     - Multiple languages with runtime switching required

---

#### Round 1 Follow-Up — Document Revision and Schedule

After receiving the four Round 1 answers, ask one additional `AskUserQuestion` call to capture the document revision letter and project schedule.

1. **"Document revision letter for this issue?"**
   - Header: `Doc Rev`
   - Options (single-select):
     - A — first issue (Recommended for new projects)
     - B / C / D / later — continuing an existing HMI Spec, I'll specify in Other
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

- **If scope = "Full plant" or "Multiple areas":** Ask: "List all areas in the plant with their area name, area number/code, and the one-sentence description of what that area does. One area per line."
- **If Nexus HMI = "Not sure":** Run `mcp__nexus__list_documented_plcs` and `mcp__nexus__hmi_project_summary`. Report what HMI data exists before continuing.
- **If Nexus HMI = "Yes":** Run `mcp__nexus__screen_inventory` and `mcp__nexus__hmi_coverage_by_plc`. Surface the screen list as a follow-up question asking which screens to include.
- **If platform = "FactoryTalk View ME":** Ask: "What is the PanelView Plus catalog number and screen size? Is this running on a 1000, 1250, 1500, or 1750 series terminal?"
- **If platform = "Ignition Perspective":** Ask: "Is this using Ignition's built-in UDT-based Symbol Factory, a custom component library, or Sepasoft modules? Is the display target desktop browser, tablet, or phone?"
- **If user includes "Recipe":** Ask: "Describe the recipe structure — what parameters are stored in a recipe, how many recipes does the system need, and does recipe management require supervisor or engineer level access?"
- **If user includes "Maintenance" role:** Ask: "Should maintenance users have I/O force capability from the HMI? (ISA-101: forces should be visible plant-wide with a force indicator on all affected faceplates.)"

---

### Step 2 — Pull Nexus HMI Data (If Available)

If Nexus HMI data exists for this project:

Run these tools and collect the results. **Do not present raw Nexus output to the user.** Extract relevant facts for the screen inventory and tag binding tables.

| Tool | Purpose |
|---|---|
| `mcp__nexus__hmi_project_summary` | Get the top-level HMI project structure — number of screens, areas covered, platform version |
| `mcp__nexus__screen_inventory` | Get the flat list of all existing screens with names, descriptions, and display numbers |
| `mcp__nexus__screen_tag_bindings` | Pull what PLC tags each existing screen is bound to |
| `mcp__nexus__hmi_coverage_by_plc` | Identify which PLC tags currently have no HMI binding (coverage gaps) |
| `mcp__nexus__ftview_screen_layout` | Get the object layout of a specific screen |
| `mcp__nexus__ftview_object_catalog` | Catalog all display objects (buttons, indicators, faceplates) used in the project |
| `mcp__nexus__classify_signal_roles` | Identify which tags are commands, feedbacks, setpoints — determines what belongs on each screen type |
| `mcp__nexus__generate_io_inventory` | Pull the I/O list to ensure every field signal has an HMI binding |

**Nexus Skepticism Rules — Non-Negotiable:**

Every piece of data sourced from Nexus MUST be flagged `[ASSUMED from Nexus — verify with customer]` in the draft document. Specifically:

- Screen names and numbers from Nexus reflect the existing project — they may not represent the intended architecture
- Tag bindings from Nexus may include test tags, legacy tags, or tags from an adjacent project
- Coverage gaps from Nexus are based on indexed data only — the actual project may have tags not yet indexed

After collecting Nexus data, present a summary to the user with `AskUserQuestion` asking which screens to carry forward, which to discard, and whether the list is complete.

---

### Step 3 — Research Applicable Standards

Use Scrapling to research current HMI design standards and platform-specific best practices before generating the specification. This ensures every design decision in the document is defensible.

**Mandatory research (run regardless of project type):**

1. **ISA-101.01-2015 HMI Design** — research key requirements: color philosophy, display hierarchy (4 levels), navigation rules, alarm display requirements, operator effectiveness metrics. Search for "ISA-101 HMI design principles" and "ISA-101 display hierarchy."

2. **ISA-18.2 Alarm Display Requirements** — research what ISA-18.2 requires specifically on the HMI side: alarm banner, alarm summary screen, alarm history, acknowledgment workflow, shelving rules. Search for "ISA-18.2 HMI alarm display requirements."

**Platform-specific research (select based on Round 1 platform answer):**

| Platform | Research Topics |
|---|---|
| FactoryTalk View SE | Rockwell best practices for SE display hierarchy, global objects, parameter files, FactoryTalk Security role configuration, PlantPAx faceplate library |
| FactoryTalk View ME | PanelView Plus display limitations, memory budget, ME-specific navigation design, FactoryTalk ME Security |
| Ignition Perspective | Inductive Automation best practices for Perspective sessions, UDT-driven views, named query usage, Perspective security zones, mobile-responsive design |
| Ignition Vision | Vision window hierarchy, template repeaters, easy chart configuration, legacy Vision security model |

**Industry-specific research (based on process area descriptions from interview):**

| Industry | Research Topics |
|---|---|
| Water / Wastewater | AWWA M2 HMI guidance, EPA SCADA security for water utilities |
| Food and Beverage | 3-A / EHEDG HMI requirements, FDA 21 CFR Part 11 electronic records (if applicable) |
| Pharmaceutical / Biotech | GAMP 5 Category 4 (configured software) requirements for HMI validation, 21 CFR Part 11 audit trail |
| Oil and Gas | API RP 554 Part 2 (DCS HMI design), ISA-18.2 alarm rationalization for process |

Use `mcp__scrapling__fetch` with `disable_resources: true` and `main_content_only: true`. Use `mcp__scrapling__stealthy_fetch` only if `fetch` fails. Pull 2–3 sources maximum per topic. Capture the specific standard number and section supporting each design requirement.

---

### Step 4 — Generate the HMI Screen Specification Document

Write the complete document using the template below. Every section is mandatory. Use `[TBD — see Open Items]` for anything that cannot be determined from the interview and research. Never omit a section — write "Not applicable — [reason]" if a section genuinely does not apply.

**Formatting rules:**
- Screen IDs use the format `SCR-{AREA}-{NNN}` (e.g., `SCR-OVW-001`, `SCR-A100-001`, `SCR-ALM-001`)
- Security level numbers follow ISA-101: Level 0 (read-only) through Level 4 (administrator)
- Every screen in the Screen Inventory must appear in the Navigation Map and have a Screen Content Specification
- Color codes are specified as hex (#RRGGBB) for exact implementation
- Tag names follow the PLC naming convention established in the FDS or I/O list

---

## HMI Screen Specification Document Template

```markdown
# HMI Screen Specification
## {Project Name}
### {HIS Project Number} | Rev {X.X} — {Status: DRAFT / FOR REVIEW / APPROVED}

---

| Field | Value |
|---|---|
| Document Number | HMS-{project-slug} |
| Revision | {X.X} |
| Status | DRAFT |
| Prepared By | Timothy Hunt, Hunt Integrative Solutions LLC |
| Preparation Date | {date} |
| Customer | {customer name} |
| Site | {site location} |
| HMI Platform | {platform name and version} |
| PLC Platform | {PLC platform and Studio 5000 / firmware version} |
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
| Approved By (Customer Operations) | | | |
| Approved By (Customer Engineering) | | | |

**This document is not valid for HMI development until all approval signatures are obtained. Any change after approval requires a Management of Change (MOC) record.**

---

## 1. Purpose and Scope

### 1.1 Purpose

This HMI Screen Specification defines every screen in the {project name} human-machine interface. It specifies:
- The complete screen inventory with unique screen IDs
- The navigation hierarchy and permitted navigation paths between screens
- The content, layout, and interactive elements of each screen
- The security levels and user roles required for each screen and each action
- The display standards and color philosophy governing all screens

This document is the binding agreement between Hunt Integrative Solutions and the customer on what the HMI contains. HMI developers implement exactly what is specified here. Changes to screen content, navigation, or security after approval require a formal revision to this document.

### 1.2 Scope

**In scope:**
{List all areas, units, and equipment types that have HMI representation in this project}

**Out of scope:**
{Explicitly state what is NOT included — adjacent systems, standalone panels on separate networks, vendor-supplied OIT panels, etc.}

### 1.3 HMI System Overview

{2–3 paragraph description of the HMI system: platform, server/client architecture, number of operator stations, remote access capability, connection to the PLC network. Written so a new engineer can understand the physical HMI layout.}

**HMI Architecture:**

| Component | Quantity | Location | Purpose |
|---|---|---|---|
| {HMI Server / Ignition Gateway} | {qty} | {location} | {purpose} |
| {Operator Workstation} | {qty} | {location} | {purpose} |
| {PanelView / Remote Panel} | {qty} | {location} | {purpose} |
| {Engineering Workstation} | {qty} | {location} | {purpose} |

---

## 2. Reference Documents

| Document | Number / Revision | Description |
|---|---|---|
| Functional Design Specification | FDS-{slug} | System functional requirements — HMI mirrors PLC logic defined here |
| P&ID | {number and revision} | Process and Instrumentation Diagram — basis for process graphic layouts |
| I/O List | IOL-{slug} | Complete tag list — every field signal must have HMI binding |
| PLC Tag Export | {file name} | Exported tag database for binding verification |
| ANSI/ISA-101.01-2015 | — | Human Machine Interfaces for the Process Industries |
| ISA-18.2-2016 | — | Management of Alarm Systems for the Process Industries |
| ISA-5.1-2009 | — | Instrumentation Symbols and Identification |
| IEC 62443-2-1 | — | IACS Security: Security for Industrial Automation and Control Systems |
| {Platform guide} | {version} | {e.g., FactoryTalk View SE Application Guide, Pub. FTVIEW-RM002} |
| {Customer HMI Standard} | {revision} | Customer engineering standard for HMI design |

---

## 3. Display Standards and Design Philosophy

### 3.1 ISA-101 Compliance Summary

This HMI is designed in accordance with ANSI/ISA-101.01-2015. The key ISA-101 principles applied to this project:

1. **Display hierarchy:** All screens fall into one of four levels (see Section 3.2). Navigation between levels follows a defined path — no shortcuts that bypass intermediate levels except for alarm response.
2. **High-performance graphics:** Process graphics use a gray background with minimal color. Color communicates state and alarm condition — not equipment type or aesthetic preference.
3. **Operator effectiveness:** Every operator task must be completable within 3 screen transitions. An operator on the Plant Overview can reach any individual equipment faceplate in ≤ 3 clicks.
4. **Consistency:** Identical equipment types use identical faceplates. A motor faceplate looks the same on every screen in the project. No one-off designs.
5. **Alarm integration:** Alarms are always visible regardless of which screen is displayed. The alarm banner is persistent across all Level 1–3 screens.

### 3.2 Display Hierarchy (ISA-101 Level Model)

| Level | Name | Description | Examples in This Project |
|---|---|---|---|
| Level 1 | Plant / Area Overview | Highest-level view. Shows status of all major areas simultaneously. One screen or one set of overview screens. | {Plant Overview SCR-OVW-001} |
| Level 2 | Area / Unit Detail | Process graphic for a specific area or unit. Shows all equipment in that area with real-time status. | {Area 100 — Feed Prep: SCR-A100-001} |
| Level 3 | Equipment Faceplate | Individual device control interface. Pop-up or dedicated screen for a single motor, valve, PID loop, or phase. | {Motor Faceplate: SCR-FP-MTR} |
| Level 4 | Diagnostic / Detail | Engineering or maintenance screens. Device health, tag values, I/O forcing, comms status. Requires elevated security level. | {I/O Diagnostic: SCR-DGN-001} |

**Infrastructure screens** (alarm summary, trends, reports, security) are at Level 2 equivalent and accessible from persistent navigation.

### 3.3 Color Philosophy (ISA-101 / ISA-5.1)

**Background:**
- Process graphic background: **#808080** (medium gray) — ISA-101 recommended
- Pop-up / faceplate background: **#C0C0C0** (light gray)
- Navigation bar background: **#404040** (dark gray)

**Equipment state colors (consistent across ALL screens):**

| State | Color | Hex | Usage |
|---|---|---|---|
| Running / Open / Energized | Green | #00C000 | Motor running, valve open, output energized |
| Stopped / Closed / De-energized | White / Light Gray | #E0E0E0 | Motor stopped, valve closed, output off |
| Faulted / Alarm | Red | #FF0000 | Any fault or active alarm condition |
| Starting / Transitioning | Flashing Green/Gray | Animated | Equipment in startup or shutdown sequence |
| Unknown / No Comms | Magenta / Purple | #FF00FF | Signal quality bad, comms lost |
| Maintenance / Bypassed | Yellow | #FFFF00 | Permissive bypassed, tag forced, in maintenance mode |
| Out of Service / Inhibited | Orange | #FF8000 | Alarm inhibited, equipment locked out |

**Color rules:**
- **Color is reserved for communicating state and alarm.** No color used for decorative purposes, grouping, or equipment type differentiation.
- Gray equipment icons are in normal, non-alarm state. Color appears only when something changes.
- Flashing is reserved for unacknowledged alarms only. No other animated elements.
- Text on gray background: **#000000** (black). Text on colored backgrounds: evaluated per contrast — minimum 4.5:1 contrast ratio (WCAG AA).
- No color coding that relies on hue alone — operators with color vision deficiency must be able to read all states. Pair color with a text label or symbol.

### 3.4 Typography

| Element | Font | Size | Weight | Color |
|---|---|---|---|---|
| Screen title | Arial | 16pt | Bold | #000000 |
| Area / equipment label | Arial | 12pt | Regular | #000000 |
| Tag value (numeric) | Arial | 14pt | Bold | #000000 (normal) / #FF0000 (alarm) |
| Alarm message text | Arial | 11pt | Regular | #000000 |
| Navigation button text | Arial | 11pt | Bold | #FFFFFF on #404040 |
| Faceplate header | Arial | 13pt | Bold | #000000 |

### 3.5 Navigation Rules

1. **Persistent navigation bar:** A navigation bar appears at the top or left side of every Level 1–3 screen. It contains: Home button, Alarm Summary shortcut, Area navigation links, logged-in user name, current date/time, PLC communication status indicator.
2. **3-click rule:** From any screen, the operator can reach any equipment faceplate in ≤ 3 screen transitions: Overview → Area → Faceplate.
3. **Alarm navigation:** Clicking an alarm in the alarm banner or alarm summary navigates directly to the screen containing the alarming device. This navigation bypasses the 3-click hierarchy — it is a direct link.
4. **Back navigation:** Every screen has a Back button that returns to the calling screen. The back path is always deterministic — pressing Back always returns to the previous screen in the hierarchy, not the previously viewed screen.
5. **Pop-up faceplates:** Equipment faceplates launch as pop-up windows centered over the calling screen. The calling screen remains visible behind the pop-up. Multiple faceplates can be open simultaneously.
6. **No dead ends:** Every screen must have a visible path back to the plant overview. No screen can be navigated to without being navigated away from.

### 3.6 Alarm Banner (ISA-18.2 Requirement)

A persistent alarm banner is displayed on all Level 1–3 screens at all times. The alarm banner:

| Element | Specification |
|---|---|
| Position | Top of screen, full width, always on top |
| Height | Minimum 40px; expandable to show 3 alarms |
| Background | Red (#FF0000) when unacknowledged alarms are present; Yellow (#FFFF00) when all acknowledged but some active; Gray (#808080) when no active alarms |
| Content | Count of active alarms, count of unacknowledged alarms, highest-priority active alarm description |
| Interaction | Click banner to navigate to Alarm Summary screen (SCR-ALM-001) |
| Audio | Audible alert tone on new unacknowledged Priority 1 alarm (configurable volume) |
| Silence | Audible silence button on alarm banner. Silencing does NOT acknowledge the alarm. |

---

## 4. Security Levels and User Roles

### 4.1 ISA-101 Security Level Definitions

Per ANSI/ISA-101.01-2015 and IEC 62443-2-1, security levels control which HMI functions each user can perform. This project implements the following security level matrix:

| Security Level | Role Name | Description | Authentication |
|---|---|---|---|
| Level 0 | Read-Only / Observer | View all screens. No command capability. No setpoint changes. | {Auto-login / No login required} |
| Level 1 | Operator | Start/stop equipment, acknowledge alarms, change mode. Cannot change setpoints or tune parameters. | {Username + Password} |
| Level 2 | Supervisor | All Level 1 rights. Change process setpoints, shelf alarms (max 8 hours), view trend history. | {Username + Password} |
| Level 3 | Engineer / Maintenance | All Level 2 rights. Edit PID tuning parameters, edit recipes, access diagnostic screens, force I/O tags (logged). | {Username + Password — requires supervisor approval for I/O force} |
| Level 4 | Administrator | All Level 3 rights. Manage user accounts, modify security configuration, change HMI project parameters. | {Username + Password + MFA or physical key} |

**Session rules:**
- Automatic logout after {X} minutes of inactivity. Timer is visible to the user.
- Failed login attempts: account locked after {3} consecutive failures. {Administrator / auto-unlock after 15 minutes}.
- Login/logout events are logged with timestamp, user ID, and workstation ID in the audit log.
- A logged-in user's identity is displayed in the navigation bar at all times.
- Level 0 (Read-Only) access does not require login — the HMI defaults to this level when no user is logged in.

### 4.2 Permission Matrix — Screen Access

| Screen ID | Screen Name | Min. Security Level to View | Min. Level to Interact |
|---|---|---|---|
| SCR-OVW-001 | Plant Overview | 0 — Read-Only | 0 (view only — no commands on overview) |
| SCR-A{NNN}-001 | Area {N} Overview | 0 — Read-Only | 0 (view only) |
| SCR-FP-MTR | Motor Faceplate | 0 — Read-Only | 1 — Operator (start/stop) |
| SCR-FP-VLV | Valve Faceplate | 0 — Read-Only | 1 — Operator (open/close) |
| SCR-FP-PID | PID Faceplate | 0 — Read-Only | 2 — Supervisor (SP change) / 3 — Engineer (tuning) |
| SCR-ALM-001 | Alarm Summary | 0 — Read-Only | 1 — Operator (acknowledge) / 2 — Supervisor (shelf) |
| SCR-TRD-001 | Trend Display | 0 — Read-Only | 1 — Operator (time range, pen selection) |
| SCR-RCP-001 | Recipe Management | 2 — Supervisor (view) | 3 — Engineer (edit/download) |
| SCR-DGN-001 | Diagnostics / I/O | 3 — Engineer | 3 — Engineer (force requires logged entry) |
| SCR-USR-001 | User Management | 4 — Administrator | 4 — Administrator |
| {add all screens} | | | |

### 4.3 Sensitive Actions — Per-Action Security Override

The following individual actions require a security level higher than the general screen access level. These are enforced at the action level (button press), not the screen level:

| Action | Screen | Security Level Required | Audit Log Entry |
|---|---|---|---|
| I/O Force (set any tag to forced value) | SCR-DGN-001 | 3 — Engineer | Yes — logs tag name, forced value, user, timestamp, reason (required entry) |
| Alarm Shelving | SCR-ALM-001 | 2 — Supervisor | Yes — logs alarm tag, duration, user, timestamp |
| PID Tuning Parameter Change | SCR-FP-PID | 3 — Engineer | Yes — logs old value, new value, user, timestamp |
| Recipe Download to PLC | SCR-RCP-001 | 3 — Engineer | Yes — logs recipe name, version, user, timestamp |
| Safety Reset (post E-stop reset) | SCR-{area} | 1 — Operator | Yes |
| Manual Mode override (permissive bypass) | SCR-FP-{eq} | 2 — Supervisor or maintenance key switch | Yes |
| {Add additional sensitive actions} | | | |

### 4.4 Audit Log Requirements (IEC 62443 / FDA 21 CFR Part 11 if applicable)

The HMI audit log shall record the following events with timestamp (UTC + local), user ID, workstation ID, and the specific action taken:

- All login and logout events
- All command actions (start, stop, open, close, mode changes)
- All setpoint changes (old value → new value)
- All alarm acknowledgments and shelving events
- All I/O force applications and releases
- All recipe downloads
- All security configuration changes

**Audit log retention:** Minimum {90 days} online in the HMI. Archived to {historian / network share / SFTP} daily. Audit log data shall not be editable by any user, including Administrator.

{If FDA 21 CFR Part 11 applies:}
The audit log constitutes an electronic record per 21 CFR Part 11.10(e). Records shall be computer-generated and shall include the date and time of operator entries and actions that create, modify, or delete electronic records. System shall not allow retroactive editing of records.

---

## 5. Screen Inventory

Complete list of all screens in the {project name} HMI. Every screen has a unique Screen ID. Screen IDs are permanent — they do not change if the screen is renamed.

**Screen ID Convention:** `SCR-{AREA}-{NNN}`
- `OVW` = Overview / Plant-level
- `A{NNN}` = Process area (e.g., A100, A200)
- `FP` = Faceplate (pop-up templates)
- `ALM` = Alarm screens
- `TRD` = Trend screens
- `RCP` = Recipe screens
- `DGN` = Diagnostic screens
- `USR` = User/Security screens
- `NAV` = Navigation / Home

### 5.1 Overview Screens

| Screen ID | Screen Name | Description | HMI Level | Min. Security Level | Platform Display Number |
|---|---|---|---|---|---|
| SCR-OVW-001 | Plant Overview | Top-level status of all process areas. Shows area run/fault status, active alarm count, PLC comms health, and key process totals. | 1 | 0 | {DISP-001} |
| {add overview screens} | | | | | |

### 5.2 Area / Process Screens

{For each process area, list all screens. Group by area.}

**Area {NNN} — {Area Name}**

| Screen ID | Screen Name | Description | HMI Level | Min. Security Level | Platform Display Number |
|---|---|---|---|---|---|
| SCR-A{NNN}-001 | {Area Name} Overview | P&ID-style process graphic showing all equipment in {area name}: {list major equipment}. Real-time status of all devices. | 2 | 0 | {DISP-NNN} |
| SCR-A{NNN}-002 | {Area Name} Trends | Real-time and historical trends for key process variables in {area name}. Pre-configured pens for {list key PVs}. | 2 | 0 | {DISP-NNN} |
| {add area screens} | | | | | |

### 5.3 Faceplate Screens (Reusable Templates)

Faceplates are template screens linked to a specific equipment instance via parameter files (FactoryTalk) or UDT binding (Ignition). One template per equipment type. No one-off faceplates.

| Screen ID | Faceplate Name | Equipment Type | Description | HMI Level | Platform Display Number |
|---|---|---|---|---|---|
| SCR-FP-MTR | Motor Faceplate | Induction motor / DOL starter / VFD | Control and status for any motor. Launched from any area screen by clicking the motor symbol. | 3 | {POPUP-001} |
| SCR-FP-VLV | Valve Faceplate | On/off or modulating control valve | Control and status for any valve. Launched from any area screen by clicking the valve symbol. | 3 | {POPUP-002} |
| SCR-FP-PID | PID Faceplate | PID control loop | Full PID faceplate: PV, SP, Output, mode (Auto/Manual/Cascade), trend, tuning tab. | 3 | {POPUP-003} |
| SCR-FP-AI | Analog Input Faceplate | Transmitter / field instrument | Current value, engineering units, alarm limits, signal quality, calibration range. | 3 | {POPUP-004} |
| SCR-FP-PHS | Phase Faceplate | ISA-88 Phase / Rockwell Phase Manager | Current phase state, elapsed time, step number, phase commands (Start, Hold, Abort, Reset), parameter values. | 3 | {POPUP-005} |
| {add faceplates for additional equipment types} | | | | | |

### 5.4 Alarm Screens

| Screen ID | Screen Name | Description | HMI Level | Min. Security Level | Platform Display Number |
|---|---|---|---|---|---|
| SCR-ALM-001 | Alarm Summary | Real-time active alarm list. All active and unacknowledged alarms, sorted by priority then timestamp. Acknowledge and shelf from this screen. | 2 | 0 (view) / 1 (ack) / 2 (shelf) | {DISP-ALM-1} |
| SCR-ALM-002 | Alarm History | Historical alarm log with date/time filter, area filter, priority filter. Export to CSV available (Level 2+). | 2 | 0 | {DISP-ALM-2} |
| SCR-ALM-003 | Alarm Configuration | View alarm setpoints, deadbands, delays, and suppression rules. Edit requires Level 3. | 4 | 0 (view) / 3 (edit) | {DISP-ALM-3} |

### 5.5 Trend Screens

| Screen ID | Screen Name | Description | HMI Level | Min. Security Level | Platform Display Number |
|---|---|---|---|---|---|
| SCR-TRD-001 | Area {NNN} Trends | Pre-configured trend display for {area name}. {X} pens pre-loaded. Operator can add/remove pens from the available pen list. | 2 | 0 | {DISP-TRD-1} |
| SCR-TRD-002 | Custom Trend | Blank trend display. User selects any available tags to trend. Up to {8} pens simultaneously. | 2 | 1 | {DISP-TRD-2} |
| {add trend screens per area} | | | | | |

### 5.6 Diagnostic / Maintenance Screens

| Screen ID | Screen Name | Description | HMI Level | Min. Security Level | Platform Display Number |
|---|---|---|---|---|---|
| SCR-DGN-001 | I/O Diagnostic — {Area NNN} | Physical I/O status for all modules in {area}. Shows raw values, forced tags (highlighted), module fault status. | 4 | 3 | {DISP-DGN-1} |
| SCR-DGN-002 | PLC Communications Health | Status of all Ethernet/IP connections, CIP connection counts, task scan times (last, max, overlap count), controller fault code. | 4 | 3 | {DISP-DGN-2} |
| SCR-DGN-003 | Force Summary | Live list of all currently forced tags — tag name, forced value, user who applied the force, timestamp. Forces remain visible plant-wide per ISA-101. | 4 | 3 | {DISP-DGN-3} |
| {add diagnostic screens as needed} | | | | | |

### 5.7 Recipe / Parameter Screens

{Include only if recipe management was identified in interview}

| Screen ID | Screen Name | Description | HMI Level | Min. Security Level | Platform Display Number |
|---|---|---|---|---|---|
| SCR-RCP-001 | Recipe List | Library of available recipes. Shows recipe name, version, last modified date, last downloaded date. | 2 | 2 (view) / 3 (edit) | {DISP-RCP-1} |
| SCR-RCP-002 | Recipe Detail | Parameter view for a selected recipe. Edit mode allows parameter changes (Level 3). | 2 | 2 (view) / 3 (edit/download) | {DISP-RCP-2} |

### 5.8 Security / User Management Screens

| Screen ID | Screen Name | Description | HMI Level | Min. Security Level | Platform Display Number |
|---|---|---|---|---|---|
| SCR-USR-001 | User Management | Create, modify, disable user accounts. Assign security levels. View login history. | 4 | 4 | {DISP-USR-1} |
| SCR-USR-002 | Audit Log Viewer | View and search the HMI audit log. Filter by user, action type, date range. Export to CSV. | 4 | 3 (view) / 4 (export) | {DISP-USR-2} |

---

## 6. Navigation Map

### 6.1 Navigation Hierarchy Diagram

The navigation hierarchy follows ISA-101 display levels. Arrows indicate permitted navigation paths. Dotted arrows indicate alarm-driven navigation (bypasses normal hierarchy).

```
┌─────────────────────────────────────────────────────────────┐
│                    PERSISTENT NAVIGATION BAR                │
│  [Home] [Alarms] [Area 100] [Area 200] [Trends] [Diag]     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                  ┌─────────────────────┐
                  │   SCR-OVW-001       │  LEVEL 1
                  │   Plant Overview    │
                  └─────────────────────┘
                     │              │
              ┌──────┘              └───────┐
              ▼                            ▼
   ┌──────────────────┐         ┌──────────────────┐
   │ SCR-A100-001     │         │ SCR-A200-001     │  LEVEL 2
   │ Area 100 Overview│         │ Area 200 Overview│
   └──────────────────┘         └──────────────────┘
     │          │                  │          │
     ▼          ▼                  ▼          ▼
 [Motor]    [Valve]            [Motor]    [PID Loop]
 Faceplate  Faceplate          Faceplate  Faceplate   LEVEL 3
 SCR-FP-MTR SCR-FP-VLV        SCR-FP-MTR SCR-FP-PID

[Alarm navigation — from any screen]
 Alarm Banner ──────────────────────────────► SCR-ALM-001
 SCR-ALM-001 ─► (alarm source screen)
```

{Expand this diagram for all areas in the project. Add actual area names, screen IDs, and all navigation paths.}

### 6.2 Navigation Path Table

Every permitted navigation path is defined here. Navigation not listed in this table is not implemented.

| From Screen ID | Navigation Trigger | To Screen ID | Notes |
|---|---|---|---|
| Any screen | Alarm Banner click | SCR-ALM-001 | Always available |
| Any screen | [Home] button | SCR-OVW-001 | Always available |
| Any screen | [Alarms] button | SCR-ALM-001 | Always available |
| Any screen | [Trends] button | SCR-TRD-001 | Always available |
| SCR-OVW-001 | Click {Area 100} area tile | SCR-A100-001 | |
| SCR-OVW-001 | Click {Area 200} area tile | SCR-A200-001 | |
| SCR-A100-001 | Click motor symbol {M-101} | SCR-FP-MTR | Pop-up |
| SCR-A100-001 | Click valve symbol {XV-101} | SCR-FP-VLV | Pop-up |
| SCR-A100-001 | Click PID loop indicator | SCR-FP-PID | Pop-up |
| SCR-A100-001 | [Trends] tab | SCR-TRD-001 | |
| SCR-ALM-001 | Click alarm row | Alarm source screen | Direct navigation to screen containing the device |
| SCR-ALM-001 | [History] button | SCR-ALM-002 | |
| SCR-FP-PID | [Tuning] tab | SCR-FP-PID (tuning tab) | Level 3 required to edit |
| {Navigation bar} | [Diagnostics] button | SCR-DGN-001 | Level 3 required |
| {add all navigation paths} | | | |

### 6.3 Dead-End and Restricted Screen Access

The following screens have no navigation from standard operator screens and are accessed only via the navigation bar (with appropriate security level):

| Screen ID | Access Method | Required Security Level |
|---|---|---|
| SCR-DGN-001 through SCR-DGN-003 | Navigation bar [Diagnostics] button | 3 — Engineer |
| SCR-USR-001, SCR-USR-002 | Navigation bar [Admin] button (visible only when Level 4 logged in) | 4 — Administrator |
| SCR-ALM-003 | From SCR-ALM-001 [Config] button (visible only to Level 3+) | 3 — Engineer |

---

## 7. Screen Content Specifications

Each screen listed in Section 5 is fully defined here. Every element on every screen is specified.

---

### SCR-OVW-001 — Plant Overview

**Purpose:** Top-level status dashboard. Operator can see the health of the entire plant at a glance without navigating to any sub-screen.

**Display level:** Level 1  
**Minimum security to view:** Level 0  
**Platform display number:** {DISP-001}  
**Nominal resolution:** {1920×1080}

**Layout description:**

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  ALARM BANNER — persistent, full width, 40px tall                            │
├────────────┬─────────────────────────────────────────────────────────────────┤
│            │  PLANT OVERVIEW                     DATE/TIME   USER: {name}   │
│ NAV        ├────────────────────────┬───────────────────────────────────────┤
│ BAR        │  AREA 100              │  AREA 200                             │
│            │  {Area name}           │  {Area name}                          │
│ [Home]     │  Status: RUNNING ●     │  Status: STOPPED ○                    │
│ [Alarms]   │  Active Alarms: 0      │  Active Alarms: 2                     │
│ [A100]     │  {Key PV}: {value} {EU}│  {Key PV}: {value} {EU}              │
│ [A200]     │  [Click to navigate]   │  [Click to navigate]                  │
│ [Trends]   ├────────────────────────┴───────────────────────────────────────┤
│ [Diag]     │  PLC COMMUNICATIONS STATUS                                     │
│            │  {PLC Name}: ● ONLINE   {PLC Name 2}: ○ OFFLINE               │
│            ├───────────────────────────────────────────────────────────────│
│            │  PLANT TOTALS (shift / daily)                                  │
│            │  {Total 1}: {value} {EU}   {Total 2}: {value} {EU}            │
└────────────┴────────────────────────────────────────────────────────────────┘
```

**Elements:**

| Element | Tag / Data Source | Type | Update Rate | Security Level to Interact |
|---|---|---|---|---|
| Alarm Banner | HMI alarm system | Alarm banner component | Real-time | 1 (to navigate to ALM screen) |
| Area 100 status tile | {Area_100_Status tag} | Status indicator + label | 1 second | 0 (click to navigate) |
| Area 100 active alarm count | HMI alarm count query — Area 100 filter | Numeric indicator | Real-time | 0 |
| Area 100 key PV: {tag name} | {PLC tag path} | Numeric display with EU | 1 second | 0 |
| {Area 200 same pattern} | | | | |
| PLC communications status | CIP connection status tag | Online/Offline indicator | 2 seconds | 0 |
| Plant total — {name} | {Totalizer tag} | Numeric display | 5 seconds | 0 |
| Navigation bar | N/A | Persistent component | N/A | See Section 4.2 |
| Date/Time display | System clock | System time component | 1 second | 0 |
| Logged-in user display | HMI security system | Current user label | On login/logout | 0 |

**Navigation from this screen:** See Section 6.2

---

### SCR-A{NNN}-001 — Area {NNN} {Area Name} Overview

**Purpose:** Process graphic showing all equipment in {area name}. Primary working screen for the area operator. All field devices are visible with real-time status. Operator starts/stops equipment and calls faceplates from this screen.

**Display level:** Level 2  
**Minimum security to view:** Level 0  
**Minimum security to interact (commands):** Level 1 — Operator  
**Platform display number:** {DISP-NNN}

**Layout description:**

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  ALARM BANNER                                                                │
├────────────┬─────────────────────────────────────────────────────────────────┤
│  NAV BAR   │  AREA {NNN} — {AREA NAME}    [TREND TAB]  [MODE: AUTO]         │
│            ├─────────────────────────────────────────────────────────────────┤
│            │                                                                 │
│            │        {Process graphic — P&ID style layout}                   │
│            │                                                                 │
│            │   ┌──────────┐     ┌──────────┐    ╔═══════╗                  │
│            │   │ M-101    │──── │ XV-101   │────║TK-101 ║                  │
│            │   │ ●Running │     │ ○ Closed │    ║ 75%   ║                  │
│            │   │ 60 Hz    │     │          │    ║       ║                  │
│            │   └──────────┘     └──────────┘    ╚═══════╝                  │
│            │                                         │                      │
│            │                                    PIT-101                     │
│            │                                   65.2 PSI                     │
│            │                                                                 │
│            ├─────────────────────────────────────────────────────────────────┤
│            │  STATUS BAR: Area Run Status: RUNNING   Last Fault: {msg}      │
└────────────┴────────────────────────────────────────────────────────────────┘
```

**Process graphic elements:**

| Symbol | Tag | Display Value | Click Action | Status Color Rule |
|---|---|---|---|---|
| Motor symbol — M-101 | {M101_RunFB}, {M101_FaultSts}, {M101_SpeedFB} | Run status label, speed in Hz (if VFD) | Launch SCR-FP-MTR with M-101 binding | Green=Running, Gray=Stopped, Red=Fault, Flashing=Transitioning |
| Valve symbol — XV-101 | {XV101_OpenFB}, {XV101_CloseFB}, {XV101_FaultSts} | Open / Closed / Unknown | Launch SCR-FP-VLV with XV-101 binding | Green=Open, Gray=Closed, Red=Fault, Yellow=Mid-position/moving |
| Tank — TK-101 | {TK101_LevelPV} | Level % (analog fill on tank symbol) | Launch SCR-FP-AI with LT-101 binding | Gray=Normal, Yellow=High/Low warning, Red=HH/LL alarm |
| Pressure transmitter — PIT-101 | {PIT101_PV} | Numeric + EU (PSI) | Launch SCR-FP-AI with PIT-101 binding | Gray=Normal, Yellow=Warning, Red=Alarm |
| {Add all equipment in this area} | | | | |

**Non-process elements:**

| Element | Tag / Source | Purpose |
|---|---|---|
| Area mode display (Auto/Manual) | {Area_Mode_Sts} | Shows current area operating mode |
| Area fault summary | {Area_FaultSummary} | Number of active faults in this area |
| Status bar — last fault message | {Area_LastFault_Msg} | Shows description of most recent fault |
| Navigation bar | Persistent | Consistent with all Level 2 screens |

**Notes:**
- P&ID layout shall match the actual customer P&ID drawing {P&ID number and revision}. Equipment positions on screen mirror drawing layout.
- All pipe lines are drawn in gray (#808080) at 2px width. No color on pipe lines unless a fluid state indicator is required (e.g., hot/cold, product/CIP) — in that case, use standard ISA-5.1 line colors with a legend.
- Instrument symbols follow ISA-5.1 circle/square conventions.

---

### SCR-FP-MTR — Motor Faceplate (Template)

**Purpose:** Reusable template faceplate for any induction motor, DOL starter, or VFD-driven motor. One template serves all motor instances via parameter binding.

**Display level:** Level 3 (pop-up)  
**Minimum security to view:** Level 0  
**Minimum security to command (start/stop/mode):** Level 1 — Operator  
**Minimum security to change setpoints:** Level 2 — Supervisor  
**Platform display number / template name:** {POPUP-001}  
**Pop-up size:** 400px × 500px, centered over calling screen

**Faceplate tabs:**

#### Tab 1 — Control (default view)

| Element | Tag (parameterized) | Type | Security Level |
|---|---|---|---|
| Equipment tag and description | Parameter — tag name | Static label from binding | N/A |
| Mode selector (Auto / Manual) | {Equip.ModeCmd} | Toggle button group | 1 — Operator |
| Run status indicator | {Equip.RunFB} | Lighted indicator — Green/Gray | 0 |
| Fault status indicator | {Equip.FaultSts} | Lighted indicator — Red when fault | 0 |
| Start button | {Equip.StartCmd} | Momentary pushbutton — Green | 1 — Operator (Manual mode only; hidden/disabled in Auto) |
| Stop button | {Equip.StopCmd} | Momentary pushbutton — Red | 1 — Operator |
| Current speed (VFD only) | {Equip.SpeedFB} | Numeric — Hz | 0 |
| Speed setpoint (VFD only) | {Equip.SpeedSP} | Numeric entry | 2 — Supervisor |
| Run hours totalizer | {Equip.RunHours} | Numeric — hours | 0 |
| Permissive status (list) | {Equip.PermBits} | Multi-state list — Green/Red per permissive | 0 |
| Fault reset button | {Equip.FaultReset} | Momentary pushbutton | 1 — Operator |
| Fault description | {Equip.FaultMsg} | Text display | 0 |

#### Tab 2 — Details

| Element | Tag | Type | Security Level |
|---|---|---|---|
| Motor current (if available) | {Equip.CurrentFB} | Numeric — Amps | 0 |
| Motor thermal status (if OL monitoring) | {Equip.ThermalOL} | Percent bar graph | 0 |
| Last start timestamp | {Equip.LastStartTime} | DateTime | 0 |
| Last stop timestamp | {Equip.LastStopTime} | DateTime | 0 |
| Start count (total) | {Equip.StartCount} | Numeric | 0 |
| Minimum run time remaining | {Equip.MinRunTimer} | Countdown timer — visible when active | 0 |
| Restart lockout remaining | {Equip.RestartTimer} | Countdown timer — visible when active | 0 |

#### Tab 3 — Alarms (inline alarm list for this device)

| Element | Source | Type | Security Level |
|---|---|---|---|
| Active alarms for this device | HMI alarm system — filtered by device tag | Alarm list — matches SCR-ALM-001 format | 0 |
| Acknowledge button | HMI alarm acknowledge | Button — available for active/unack alarms | 1 — Operator |

---

### SCR-FP-VLV — Valve Faceplate (Template)

**Purpose:** Reusable template for on/off and modulating control valves.

**Display level:** Level 3 (pop-up)  
**Minimum security to view:** Level 0  
**Minimum security to command:** Level 1 — Operator  
**Pop-up size:** 350px × 450px

| Element | Tag (parameterized) | Type | Security Level |
|---|---|---|---|
| Tag and description | Parameter | Static label | N/A |
| Open status indicator | {Valve.OpenFB} | Lighted — Green when open | 0 |
| Closed status indicator | {Valve.CloseFB} | Lighted — Gray when closed | 0 |
| Fault indicator | {Valve.FaultSts} | Lighted — Red when fault | 0 |
| Position (modulating only) | {Valve.PositionFB} | Numeric % + horizontal bar graph | 0 |
| Command: Open | {Valve.OpenCmd} | Momentary button — Green | 1 — Operator |
| Command: Close | {Valve.CloseCmd} | Momentary button — Gray | 1 — Operator |
| Position setpoint (modulating only) | {Valve.PositionSP} | Numeric entry 0–100% | 1 — Operator |
| Mode selector (Auto / Manual) | {Valve.ModeCmd} | Toggle button group | 1 — Operator |
| Travel time display | {Valve.TravelTimer} | Countdown while moving | 0 |
| Fault description | {Valve.FaultMsg} | Text | 0 |
| Fault reset | {Valve.FaultReset} | Momentary button | 1 — Operator |

---

### SCR-FP-PID — PID Loop Faceplate (Template)

**Purpose:** Full PID faceplate per PlantPAx standard. Supports Auto, Manual, and Cascade modes.

**Display level:** Level 3 (pop-up)  
**Minimum security to view:** Level 0  
**Minimum security to change SP:** Level 2 — Supervisor  
**Minimum security to tune:** Level 3 — Engineer  
**Pop-up size:** 500px × 600px

#### Tab 1 — Control

| Element | Tag | Type | Security Level |
|---|---|---|---|
| Loop tag and description | Parameter | Static label | N/A |
| PV (Process Variable) | {Loop.PV} | Large numeric + EU + color (alarm state) | 0 |
| SP (Setpoint) | {Loop.SP} | Numeric entry | 2 — Supervisor |
| Output % | {Loop.Output} | Numeric + horizontal bar 0–100% | 0 |
| Mode selector (Auto / Manual / Cascade) | {Loop.ModeCmd} | Button group | 1 — Operator (Auto/Manual) / 3 — Engineer (Cascade) |
| PV + SP trend (5-minute rolling) | Historian query or in-memory pen | Trend chart — 2 pens | 0 |
| Alarm limits display (HH, H, L, LL) | {Loop.HHLimit}, {Loop.HLimit}, {Loop.LLimit}, {Loop.LLLimit} | Numeric labels overlaid on trend | 0 |

#### Tab 2 — Tuning (Level 3 required to edit)

| Element | Tag | Type | Security Level |
|---|---|---|---|
| Proportional Gain (Kp) | {Loop.PGain} | Numeric entry | 3 — Engineer |
| Integral Time (Ti / Ki) | {Loop.IGain} | Numeric entry | 3 — Engineer |
| Derivative Time (Td / Kd) | {Loop.DGain} | Numeric entry | 3 — Engineer |
| Output High Limit | {Loop.OutputHLimit} | Numeric entry | 3 — Engineer |
| Output Low Limit | {Loop.OutputLLimit} | Numeric entry | 3 — Engineer |
| Anti-windup enable | {Loop.AntiWindup} | Toggle | 3 — Engineer |
| Previous values (read-only) | Historian — last tuning change | Read-only numeric display | 3 — Engineer |

---

### SCR-ALM-001 — Alarm Summary

**Purpose:** Central alarm management screen. All active and unacknowledged alarms in the plant. ISA-18.2 compliant display. This is the primary response screen when an alarm occurs.

**Display level:** Level 2  
**Minimum security to view:** Level 0  
**Minimum security to acknowledge:** Level 1 — Operator  
**Minimum security to shelf:** Level 2 — Supervisor  
**Platform display number:** {DISP-ALM-1}

**Layout description:**

```
┌──────────────────────────────────────────────────────────────────────────────┐
│  ALARM BANNER                                                                │
├────────────┬─────────────────────────────────────────────────────────────────┤
│  NAV BAR   │  ALARM SUMMARY               [History ▶]  [Config ▶] (L3+)    │
│            ├──────────┬──────────────────────────────────────────────────────┤
│            │ FILTER:  │ [All Areas ▼]  [All Priorities ▼]  [Active ▼]       │
│            │          │ Active: {N}  Unacknowledged: {N}  Shelved: {N}       │
│            ├──────────┴──────────────────────────────────────────────────────┤
│            │ PRI │ TIME          │ TAG        │ DESCRIPTION    │ AREA │ ACK  │
│            ├─────┼───────────────┼────────────┼────────────────┼──────┼──────┤
│            │  1  │ 14:32:05      │ PSH-101    │ High Discharge │ A100 │ [ACK]│
│            │     │               │            │ Pressure       │      │      │
│            ├─────┼───────────────┼────────────┼────────────────┼──────┼──────┤
│            │  2  │ 14:28:41      │ M-201-OL   │ Motor Overload │ A200 │ ✓    │
│            │     │               │            │ Trip           │      │      │
│            ├─────┼───────────────┼────────────┼────────────────┼──────┼──────┤
│            │     │               │            │                │      │      │
│            ├─────────────────────────────────────────────────────────────────┤
│            │  [ACK ALL VISIBLE]  [SHELF SELECTED] (L2+)  [NAVIGATE TO]      │
└────────────┴────────────────────────────────────────────────────────────────┘
```

**Alarm list columns:**

| Column | Content | Width | Color Coding |
|---|---|---|---|
| Priority | 1–4 per ISA-18.2 | 40px | 1=Red, 2=Orange, 3=Yellow, 4=Blue |
| Time | Timestamp of alarm occurrence (local time) | 120px | |
| Tag | PLC alarm tag name | 120px | |
| Description | Alarm message text (from alarm rationalization register) | Fill | |
| Area | Process area where alarm originated | 80px | |
| State | Active/Acknowledged/Shelved | 80px | Red=Unack, Yellow=Ack+Active, Gray=Cleared |
| Ack | Acknowledge button (if unacknowledged) or checkmark (if acknowledged) | 60px | |

**Row color rules (ISA-18.2 visual convention):**
- Unacknowledged + Active: Red background, flashing
- Acknowledged + Active: Red background, steady
- Unacknowledged + Cleared: Yellow background, flashing
- Acknowledged + Cleared: Gray background (auto-removes from active list)
- Shelved: Orange background

**Actions available on this screen:**

| Action | Security Level | Behavior |
|---|---|---|
| Acknowledge single alarm | 1 — Operator | Marks alarm acknowledged. Requires operator to see alarm detail before acknowledging (detail popup on click). |
| Acknowledge all visible | 1 — Operator | Acknowledges all currently filtered alarms. Logs as bulk acknowledge with user ID. |
| Shelf selected alarm | 2 — Supervisor | Shelves alarm for operator-specified duration (max 8 hours). Requires reason entry. Logs user, duration, reason. |
| Navigate to alarm source | 0 (click alarm row) | Navigates to the process screen containing the alarming device. Alarm summary remains accessible via nav bar. |
| Filter by area | 0 | Filters list to selected area. Does not navigate — stays on alarm summary screen. |
| Filter by priority | 0 | Filters to selected priority level. |

---

### SCR-TRD-001 — {Area Name} Trends

**Purpose:** Pre-configured trend display for {area name} key process variables. Operator can view real-time and historical data without configuring pens.

**Display level:** Level 2  
**Minimum security to view:** Level 0  
**Minimum security to modify pen config:** Level 1  
**Platform display number:** {DISP-TRD-1}

**Pre-configured pens:**

| Pen # | Tag | Description | EU | Color | Y-Axis Scale |
|---|---|---|---|---|---|
| 1 | {PIT-101_PV} | Discharge Pressure | PSI | Blue | 0–100 |
| 2 | {PIT-101_SP} | Discharge Pressure SP | PSI | Blue (dashed) | 0–100 |
| 3 | {FIT-101_PV} | Flow Rate | GPM | Green | 0–500 |
| 4 | {LIT-101_PV} | Tank Level | % | Orange | 0–100 |
| {add pens per area} | | | | | |

**Trend display parameters:**
- Default time range: Last 1 hour (real-time scrolling)
- Operator-selectable time ranges: 15 min, 1 hour, 4 hours, 8 hours, 24 hours, custom date range
- Historian source: {FactoryTalk Historian tag / Ignition history / SQL database}
- Data resolution: Raw (1 second) for ≤ 4-hour views; 1-minute average for longer views

---

### SCR-DGN-001 — I/O Diagnostic — {Area NNN}

**Purpose:** Raw I/O status for all modules in {area NNN}. Used by maintenance and engineering to verify field wiring, check module health, and apply or remove tag forces during commissioning.

**Display level:** Level 4  
**Minimum security to view:** Level 3 — Engineer  
**Minimum security to force a tag:** Level 3 — Engineer (with required reason entry)

**Force indicator rule (ISA-101):** Any forced tag on ANY screen in the project shall display a "FORCED" indicator overlay on the corresponding symbol. The Force Summary screen (SCR-DGN-003) provides a plant-wide list of all active forces.

**I/O table elements:**

| Element | Content | Forced State Indicator |
|---|---|---|
| Card slot indicator | Module catalog number, slot number, fault status | Red if module faulted |
| Channel row | Channel number, tag name, current raw value, current EU value, engineering units | Yellow border + "F" flag if forced |
| Force control (per channel) | Current value display + editable force value + [Apply Force] / [Remove Force] buttons | |
| Force reason entry | Required text entry before any force is applied | |

---

### SCR-USR-001 — User Management

**Purpose:** Create and manage HMI user accounts, assign security levels, and view login history.

**Display level:** Level 4  
**Minimum security:** Level 4 — Administrator  
**Note:** This screen is not visible in the navigation bar for users below Level 4.

**Elements:**

| Element | Content |
|---|---|
| User list | All configured users: username, display name, security level, last login, account status (active/locked/disabled) |
| [Add User] | Creates new user account — requires username, display name, initial password (force-change at first login), security level |
| [Edit User] | Modify security level, reset password, enable/disable account |
| [Audit Log] | Link to SCR-USR-002 |
| Login history for selected user | Last 10 login/logout events for highlighted user |

---

## 8. HMI Tag Binding Plan

### 8.1 Tag Binding Architecture

All HMI-to-PLC tag bindings use the following architecture. No hardcoded raw I/O addresses (e.g., `Local:1:I.Data.0`) shall appear in HMI displays.

| Platform | Binding Method |
|---|---|
| FactoryTalk View SE | FactoryTalk Linx (RSLinx Enterprise) — OPC server. Tags bound by controller-scoped tag name. Parameter files pass tag names to faceplate templates. |
| FactoryTalk View ME | FactoryTalk Linx (device shortcut). Direct tag binding in display elements. |
| Ignition | OPC-UA connection to ControlLogix. Tags browsed from server. UDT-driven views bind via template parameter. |

### 8.2 Tag Naming in HMI

HMI tag names shall mirror PLC tag names exactly. No translation tables. No HMI-only aliases for PLC tags except:
- Calculated values (e.g., runtime totals computed in the HMI if not available in PLC)
- HMI-local navigation and security tags

### 8.3 Coverage Verification

Before FAT, every field signal in the I/O list (document IOL-{slug}) shall have at least one HMI binding. Coverage is verified by:
1. Run `mcp__nexus__hmi_coverage_by_plc` after HMI development is complete
2. Any I/O tag with no HMI binding is a coverage gap requiring resolution
3. Acceptable exceptions: spare I/O channels (no tag required until used), internal PLC-only tags not intended for operator visibility

---

## 9. Faceplate Parameter Binding Reference

Faceplate templates are bound to specific equipment instances via parameter files (FactoryTalk) or UDT template parameters (Ignition). The following table maps each equipment tag to its faceplate template and binding path.

| Equipment Tag | Description | Faceplate Template | Binding Parameter / UDT Instance Path |
|---|---|---|---|
| M-101 | {Motor description} | SCR-FP-MTR | {Controller}/{Program}.M101 |
| XV-101 | {Valve description} | SCR-FP-VLV | {Controller}/{Program}.XV101 |
| PID-101 | {PID loop description} | SCR-FP-PID | {Controller}/{Program}.PID101 |
| {add all equipment} | | | |

---

## 10. Open Items and TBDs

Items that could not be confirmed during document preparation. All must be resolved before the HMI Screen Specification is approved for development.

| # | Section | Item Description | Owner | Target Date |
|---|---|---|---|---|
| TBD-001 | {section} | {description} | {customer / HIS} | {date} |
| | | | | |

---

## 11. Glossary

| Term | Definition |
|---|---|
| ACK | Acknowledge — operator action confirming they have seen an alarm |
| DI / DO | Digital Input / Digital Output |
| EU | Engineering Units — the physical unit of measure for an analog signal (PSI, GPM, °F, %) |
| Faceplate | A standardized pop-up or sub-screen that controls and displays status for one equipment instance |
| HMI | Human-Machine Interface — the operator display and control interface |
| ISA-101 | ANSI/ISA-101.01-2015: Human Machine Interfaces for the Process Industries |
| ISA-18.2 | ANSI/ISA-18.2-2016: Management of Alarm Systems for the Process Industries |
| ISA-5.1 | ANSI/ISA-5.1-2009: Instrumentation Symbols and Identification |
| Navigation map | The defined set of permitted paths between screens |
| OPC-UA | Open Platform Communications — Unified Architecture. Industry-standard protocol for HMI-to-PLC data exchange. |
| Parameter file | FactoryTalk View mechanism for passing tag paths and configuration to a display template |
| PV | Process Variable — the measured value of a process parameter |
| Screen ID | Unique identifier for each HMI screen (format: SCR-{AREA}-{NNN}) |
| Security level | ISA-101 user access tier (0=Read-Only through 4=Administrator) |
| Shelf / Suppress | Temporarily remove an alarm from the active alarm list for a defined duration |
| SP | Setpoint — the desired or target value for a process variable |
| UDT | User-Defined Tag (data type in Studio 5000) / User-Defined Type (Ignition) |
| VFD | Variable Frequency Drive |
```

---

### Step 5 — Finalize and Save the Document

1. Replace ALL `{placeholder}` fields with actual values from the interview and research.
2. Verify every screen in the Screen Inventory (Section 5) has a Screen Content Specification in Section 7. If a screen spec is not yet written, add it with `[TBD — spec to be developed before HMI development begins]`.
3. Verify every screen in Section 5 appears in the Navigation Map (Section 6.2).
4. Verify every faceplate type referenced in Section 5 has a complete faceplate spec in Section 7.
5. Count all `[TBD]` items. List them all in Section 10 (Open Items).
6. Verify the Permission Matrix in Section 4.2 includes every screen from Section 5.
7. **Always ask where to save — never silently default to Desktop.** Use `AskUserQuestion`:

   **"Where should the HMI Screen Specification be saved?"**
   - Header: `Save Location`
   - Options (single-select):
     - Active project folder — I'll type the path (Recommended)
     - Same folder as other project documents — I'll type the path
     - Current working directory
     - Desktop — `/mnt/c/Users/HIS Controls/Desktop/` (Windows Desktop via WSL — use only for one-off / scratch documents)

   If outputs from `/io-list`, `/bom`, `/fds`, `/soo`, or `/alarm-philosophy` exist in the same save folder, mention them in the document's Reference Documents table.

8. Save the document as: `HMS-{project-slug}-Rev{rev}-DRAFT.md` in the chosen location, using the user-supplied revision letter from the Round 1 follow-up.
9. Report the full file path, total screen count, and TBD count to the user.

---

### Step 6 — Present Next Steps Using AskUserQuestion

After saving, use `AskUserQuestion`:

**"The HMI Screen Specification draft is saved. What would you like to do next?"**
- Header: `Next Step`
- Options (single-select):
  - Review and resolve open items — walk through each TBD together
  - Add missing screen specifications — write specs for screens marked TBD
  - Add more areas — the project has more areas we haven't covered yet
  - Review the navigation map — trace the 3-click path for a specific operator task
  - Review security levels — confirm the permission matrix with the customer
  - Export as Word/PDF — convert the Markdown to a customer-ready format
  - Done — this draft is ready for customer review

---

## Quality Rules

Before considering the HMI Screen Specification complete, verify:

- [ ] Every screen in Section 5 (Screen Inventory) has a unique Screen ID in the correct format
- [ ] Every screen in Section 5 appears in the Navigation Map (Section 6.2) with at least one defined navigation path TO it and at least one FROM it (no dead ends)
- [ ] Every screen in Section 5 has a complete Screen Content Specification in Section 7
- [ ] Every faceplate template in Section 5.3 is a reusable template — no one-off faceplates for individual equipment instances
- [ ] The Permission Matrix (Section 4.2) includes every screen from Section 5
- [ ] The alarm banner is specified on every Level 1–3 screen
- [ ] The 3-click rule is satisfied: from Plant Overview to any equipment faceplate in ≤ 3 transitions
- [ ] All colors are specified as hex codes (#RRGGBB)
- [ ] All tag bindings reference controller-scoped PLC tag names — no raw I/O addresses
- [ ] Every I/O force action triggers an audit log entry with reason field
- [ ] The audit log section covers all required events (login, commands, setpoint changes, alarms, forces, recipe downloads)
- [ ] Security levels are consistent between the Screen Inventory table (Section 5) and the Permission Matrix (Section 4.2)
- [ ] All `[ASSUMED from Nexus]` items are listed in Section 10 (Open Items)
- [ ] Document reads clearly to a customer who has no HMI development experience
