---
name: plc-io-list
description: Generate a standards-compliant I/O List template (CSV + definitions reference) for industrial automation projects. Covers every signal with tag, card/slot/channel, wire numbers, terminal blocks, cable numbers, signal type, scaling, fail-safe state, safety classification, and IS entity parameters for hazardous-area loops. Follows ISA-5.1-2024, ISA-TR5.1.02, ISA-5.4-1991, ISA-12.06, IEC 62443-3-2, IEC 60079-25, NEC 504, and NFPA 79. Designed as the electrician's roadmap and the commissioning engineer's checklist.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [io-list, plc, isa-5.1, iec-62443, nec-504, ieee-90, his]
    ported_from: claude-code/io-list
---
# /io-list — I/O List Template Generator

**Invocation:** `/io-list [project name]`

**Examples:**
- `/io-list "Blending Skid — Area 300"`
- `/io-list "Wastewater Lift Station No. 4"`
- `/io-list "Conveyor System — Building 7"`

---

## Purpose

Generate a complete, standards-aligned I/O List template that serves three audiences:

1. **The electrician** — wire numbers, terminal blocks, cable numbers, field device location. Every physical connection traceable without opening a drawing.
2. **The controls engineer** — chassis/rack/slot/channel, module catalog number, series, firmware, PLC tag address, scaling, fail-safe behavior.
3. **The commissioning team** — loop drawing reference, P&ID reference, calibration LRV/URV, normal state, safety classification, IS entity parameters, security zone, test status.

Standards alignment:
- **ANSI/ISA-5.1-2024** — Instrumentation Symbols and Identification (tag-numbering structure: Process Identifier + Variable Letter + Modifier + Loop Number + Suffix)
- **ISA-TR5.1.02** — Instrumentation and Control Identification System Guidelines (companion)
- **ISA-5.4-1991** — Instrument Loop Diagrams (loop-component identification minimum content)
- **ISA-20** — Specification forms for process instruments (LRV/URV, sensor span fields)
- **ISA-12.06.01 / IEC 60079-25 / NEC 504** — Intrinsic safety entity parameters (Voc, Isc, Po, Lo, Co)
- **IEC 62443-3-2** — Security zone/conduit classification (Sec_Zone column add-on)
- **NFPA 79** — Wire numbering for industrial machinery
- **NEC Article 501/502** — Hazardous location classification fields
- Platform-specific module references researched live via Scrapling, scoped to the platform actually selected.

---

## Execution Flow

### MANDATORY: `AskUserQuestion` for ALL Questions

Never ask in plain text. Every question goes through `AskUserQuestion` so the user gets proper UI controls, selectable options, and a structured experience. **This is a formal interview, not a chat.** Never assume a value the user did not provide — if a value is needed and was not answered, ask. Defaults are only acceptable when explicitly labeled `(Recommended)` inside an `AskUserQuestion` option, never silently in code or pre-populated rows.

The interview runs in **three sequential rounds plus targeted follow-ups**. Complete each round before proceeding.

---

### Step 1 — Interview Round 1: Project Identity, Schedule, Platform, and Tag Convention

```
AskUserQuestion([
  {
    question: "What is the I/O list revision letter?",
    header: "IO List Rev",
    multiSelect: false,
    options: [
      { label: "A — first issue (Recommended for new projects)", description: "Use Rev A for the first issue. Increment on every change after first issue." },
      { label: "B / C / D / later", description: "Continuing an existing list. I will ask which letter in a follow-up." },
      { label: "Draft / pre-revision", description: "Internal working copy; do not issue to field." }
    ]
  },
  {
    question: "What is the project's required-on-site / FAT date?",
    header: "FAT Date",
    multiSelect: false,
    options: [
      { label: "Within 8 weeks — aggressive", description: "Aggressive schedule; flag any TBD field that blocks panel fab." },
      { label: "8–16 weeks — typical", description: "Typical schedule." },
      { label: "16+ weeks — long lead", description: "Long lead; standard schedule pressure." },
      { label: "Unknown / not yet set", description: "Skip schedule-relative checks." }
    ]
  },
  {
    question: "What is the control platform?",
    header: "Platform",
    multiSelect: false,
    options: [
      { label: "Rockwell ControlLogix / GuardLogix (1756)", description: "Full rack. Chassis = Local or RIO-N. 0-based channel by raw-address convention." },
      { label: "Rockwell CompactLogix 5380 / Compact GuardLogix (5069) (Recommended for new mid-range)", description: "5069 backplane. Chassis = Local. 0-based channel by raw-address convention." },
      { label: "Rockwell legacy CompactLogix (1769) / MicroLogix / Micro800", description: "1769 chassis: leave Chassis blank. MicroLogix: Slot=onboard / 1 / 2." },
      { label: "Other (Siemens / Beckhoff / Modicon / etc.)", description: "Raw_Addr column accepts any platform's address syntax. I will use placeholder examples." }
    ]
  },
  {
    question: "What is the project type?",
    header: "Project Type",
    multiSelect: false,
    options: [
      { label: "Greenfield — new system from scratch", description: "Full I/O list from scratch." },
      { label: "Retrofit / Upgrade", description: "Some signals carried over from existing system. Status=EXISTING for unchanged rows." },
      { label: "Migration — PLC-5 / SLC to Logix", description: "Old address → new address columns both populated." },
      { label: "Brownfield addition — adding I/O to running system", description: "Spare-channel assignment from existing modules required." }
    ]
  }
])
```

After Round 1 main answers, ask three targeted follow-ups in a single `AskUserQuestion` call:

```
AskUserQuestion([
  {
    question: "What tag-numbering convention does this project use?",
    header: "Tag Convention",
    multiSelect: false,
    options: [
      { label: "ISA-5.1-2024 — FunctionLetters + LoopNumber, no area prefix (Recommended for single-skid)", description: "Format: FIC-101, LSH-204A. Used on standalone machines and skids." },
      { label: "ISA-5.1-2024 with Process Identifier prefix (Recommended for multi-area)", description: "Format: 100-FIC-101, 200-LSH-204A. The PID prefix is the area / unit number per ISA-TR5.1.02 hierarchy." },
      { label: "Customer-corporate / equipment-numbered", description: "Customer-defined format (e.g., P101A-FT-01). I will accept any string and skip the format regex check." },
      { label: "Other / I'll define", description: "Free format; format regex check disabled." }
    ]
  },
  {
    question: "Channel-numbering base on this project?",
    header: "Channel Base",
    multiSelect: false,
    options: [
      { label: "0-based — matches Rockwell raw address (Recommended for Rockwell projects)", description: "Channel 0..15 on a 16-pt module. Matches Local:s:I.Data.0..15." },
      { label: "1-based — matches terminal-block silkscreen", description: "Channel 1..16 on a 16-pt module. Matches the labels stamped on the module faceplate." },
      { label: "Customer-defined", description: "Free entry; channel-base quality check disabled." }
    ]
  },
  {
    question: "Capture document header info — customer / project number / engineer-of-record",
    header: "Doc Header",
    multiSelect: false,
    options: [
      { label: "I'll type all three", description: "You will be prompted for customer, HIS project number, and engineer-of-record initials." },
      { label: "Use placeholders for now — I'll fill in later", description: "Header fields set to {{TBD}}. Must be resolved before issuing." }
    ]
  }
])
```

If the user picked "I'll type all three," follow up via Other to capture the strings.

---

### Step 2 — Interview Round 2: Signal Types and Wire Numbering

```
AskUserQuestion([
  {
    question: "Which signal types are present? (select all that apply)",
    header: "Signal Types",
    multiSelect: true,
    options: [
      { label: "DI — 24VDC / 120VAC Discrete Input", description: "Pushbuttons, limit switches, proximity sensors, dry contacts." },
      { label: "DO — 24VDC / 120VAC Discrete Output", description: "Solenoids, pilot lights, relay coils, motor starters." },
      { label: "AI — 4-20mA Analog Input", description: "Transmitters: pressure, flow, level, temperature." },
      { label: "AO — 4-20mA Analog Output", description: "Control valves, VFD speed references, positioners." }
    ]
  },
  {
    question: "Any special signal types? (select all that apply)",
    header: "Special Signals",
    multiSelect: true,
    options: [
      { label: "RTD / Thermocouple (TC)", description: "Direct temperature input — RTD/TC module required." },
      { label: "High-Speed Counter (HSC)", description: "Encoder, pulse meter, turbine flow — HSC module required." },
      { label: "Safety I/O (CIP Safety / FSoE)", description: "GuardLogix 1791ES / 1734 POINT Guard — dual-channel wiring." },
      { label: "Serial / Modbus RTU / Ethernet field devices", description: "Drives, instruments, meters — no I/O channel; comm-port mapping." }
    ]
  },
  {
    question: "Wire numbering convention?",
    header: "Wire Numbering",
    multiSelect: false,
    options: [
      { label: "NFPA 79 sequential by panel (Recommended for OEM machines)", description: "Sequential numbers per panel section (101, 102, 103…). NFPA 79 §13. Simple; no information encoded in the number." },
      { label: "ISA-5.4 device-point (Recommended for process plants)", description: "Wire = instrument tag + signal designator (FT101-+, FT101--, MV301-OPN). Matches ISA-5.4 loop-diagram practice." },
      { label: "Source-destination (panel-terminal format)", description: "PNL01-TB1:23 — every wire labeled with its termination point. Common at large EPC panel shops." },
      { label: "Customer-defined / I'll fill in later", description: "Wire_No column = WN-TBD." }
    ]
  }
])
```

---

### Step 3 — Interview Round 3: Safety, Hazardous Area, Security Zone, Output

```
AskUserQuestion([
  {
    question: "Safety Instrumented Functions or safety-rated I/O?",
    header: "Safety I/O",
    multiSelect: false,
    options: [
      { label: "Yes — SIL-rated functions present", description: "Add Safety_Zone, Dual_Ch, SIF_ID, SIL_Level, DC_Coverage columns. IEC 61511 / ISO 13849-1." },
      { label: "Yes — E-stop / light curtain only (no SIL calc)", description: "Add Safety_Zone and Dual_Ch only. No SIL columns." },
      { label: "No — standard process interlocks only", description: "Omit safety columns." }
    ]
  },
  {
    question: "Field devices in classified (hazardous) locations?",
    header: "Haz. Area",
    multiSelect: false,
    options: [
      { label: "Yes — NEC Class/Division (North America)", description: "Add Class/Div, Group, T-rating, IS Barrier, and full IS entity parameters per NEC 504 / ISA-12.06." },
      { label: "Yes — IECEx / ATEX Zones (international)", description: "Add Zone, Gas Group, T-rating, IS Barrier, and full IS entity parameters per IEC 60079-25." },
      { label: "No — general purpose only", description: "Omit haz-area columns." }
    ]
  },
  {
    question: "Add IEC 62443-3-2 security zone column?",
    header: "Sec Zone",
    multiSelect: false,
    options: [
      { label: "Yes — add Sec_Zone column", description: "Required if the I/O list doubles as the 62443 asset inventory at the channel level." },
      { label: "No — omit", description: "Skip the Sec_Zone column. Standards-alignment claim downgraded accordingly." }
    ]
  },
  {
    question: "Output format?",
    header: "Output",
    multiSelect: false,
    options: [
      { label: "CSV + Markdown definitions (Recommended)", description: "IO-List-{project}-Rev{rev}.csv plus IO-List-{project}-Rev{rev}-DEFINITIONS.md. Definitions file carries the example row." },
      { label: "CSV only", description: "Single CSV. Definitions inline as a header comment block." },
      { label: "Markdown table only", description: "Single .md file — useful for embedding in a Git repo or wiki." }
    ]
  }
])
```

---

### Step 4 — Research (Scoped to Selected Platform)

After the interview, fetch a *standards-relevant* page scoped to the platform answered. Cite at least one fact from the fetched page in the I/O list Notes for affected rows.

```
# ControlLogix selected →
mcp__scrapling__fetch("https://literature.rockwellautomation.com/idc/groups/literature/documents/sg/1756-sg001_-en-p.pdf")

# CompactLogix 5380 / 5069 selected →
mcp__scrapling__fetch("https://literature.rockwellautomation.com/idc/groups/literature/documents/sg/5069-sg001_-en-p.pdf")

# Other platform → skip Rockwell fetch; proceed with placeholders.
```

If Scrapling errors or returns empty, proceed with the built-in Signal Type Reference. Do NOT block on failed fetches.

---

### Step 5 — Nexus + Local Cross-Skill Data Pull (Best-Effort)

In order of preference:

1. **Local cross-skill artifacts.** Glob the project save location for:
   - `WIRE-SCHEDULE-*.csv` and `TB-SCHEDULE-*.csv` (from `/wire-diagram`) → pre-fill `Wire_No`, `TB_ID`, `TB_Term`, `Cable_No`, `Cable_Type` for matching tags. Cite source: `Source: WIRE-SCHEDULE-…`.
   - `BOM-*-Rev*.csv` (from `/bom`) → cross-check `Module_CatNo`, `Series`, `FW_Required` against the BOM Section 1.
2. **Nexus** (if user indicated Nexus is loaded):
   ```
   mcp__nexus__plant_summary()
   mcp__nexus__list_documented_plcs()
   mcp__nexus__generate_io_inventory(plc_id: <selected PLC>)
   mcp__nexus__classify_signal_roles(plc_id: <selected PLC>)
   mcp__nexus__group_io_by_area(plc_id: <selected PLC>)
   ```
   Pre-populate rows; mark Status = `IMPORT-UNVERIFIED` with Notes `Source: NEXUS`.
3. **Neither** — generate a blank template (25 blank data rows with `--` placeholders).

If both local cross-skill files AND Nexus exist, prefer the local file and flag any discrepancy in Notes.

---

### Step 6 — Generate the I/O List

#### Column Set — Core (Always Present)

| # | Column | Notes |
|---|---|---|
| 1 | **Row#** | Sequential. Never renumber after issue. |
| 2 | **Process_Id** | Area / unit prefix per ISA-TR5.1.02. Empty (`--`) if convention from Round 1 has no prefix. |
| 3 | **Tag** | ISA-5.1-2024 tag (FunctionLetters + LoopNumber + optional Suffix). Format per Round 1 convention. |
| 4 | **Description** | Plain-English description (`Feed Pump P-101 Run Status`). |
| 5 | **IO_Type** | `DI`, `DO`, `AI`, `AO`, `RTD`, `TC`, `HSC`, `HSI`, `SAFE-DI`, `SAFE-DO`, `COMM`. |
| 6 | **Loop_Type** | ISA-5.1 function letter: `I` (Indicator), `R` (Recorder), `C` (Controller), `S` (Switch), `Q` (Integrator), `T` (Transmitter), `Y` (Computing/Relay), `--` for non-instrument. |
| 7 | **Signal_Char** | `24VDC Source/Sink`, `120VAC`, `4-20mA 2-wire/3-wire/4-wire`, `0-10VDC`, `Pt100 RTD`, `J/K/T TC`, `NPN`, `PNP`, `Dry Contact`. |
| 8 | **Loop_Power** | `2-wire loop`, `4-wire ext`, `Self`, `--`. Required for AI/AO. |
| 9 | **HART** | `Y` / `N` / `--`. AI/AO only. |
| 10 | **Controller** | PLC name (e.g., `PLC-AREA100`, `CLX-01`). |
| 11 | **Chassis** | `Local`, `RIO-N`. CompactLogix 5380/5069: `Local`. Legacy 1769: blank. |
| 12 | **Slot** | Module slot number. |
| 13 | **Channel ({base})** | Channel within module. Header reflects 0-based or 1-based per Round-1 follow-up. |
| 14 | **Module_CatNo** | Catalog number with all dashes (`1756-IB16`). |
| 15 | **Series** | Series letter (`D`). Required on Logix-family modules; `--` otherwise. |
| 16 | **FW_Required** | Firmware revision required (`34.011`). Required on controllers and managed network gear. |
| 17 | **PLC_Tag** | Buffered tag name (`IO_Mapping:Area100:FIC101_PV`). |
| 18 | **Raw_Addr** | Direct controller address. Format per platform: Rockwell `Local:3:I.Data.5`; Siemens `I 0.0` or `%I0.0`; Beckhoff `%IX0.0`; Modicon `100001`. |
| 19 | **Wire_No** | All wire numbers for this point. Multi-wire separated by `/`. |
| 20 | **TB_ID** | Terminal block strip ID (`TB1`, `TB-FIELD-1`). |
| 21 | **TB_Term** | Terminal positions (`23/24`). |
| 22 | **Cable_No** | Cable identifier matching cable schedule. |
| 23 | **Cable_Type** | UL/CSA: `18AWG/2C SHLD`, `TC-ER`, `MTW`, `CAT6`, `FO-MM-4C`. IEC: `1.5mm²/2C SHLD`, `H05V-K`. |
| 24 | **Field_Loc** | Physical location (`AREA-100`, `MCC-1`, `FIELD-JB-3`). |
| 25 | **PID_Ref** | P&ID drawing + zone (`P-100 SH2 C3`). |
| 26 | **Loop_Ref** | Loop-diagram number per ISA-5.4 (`L-FIC-101`). |
| 27 | **Eng_Units** | `GPM`, `PSI`, `°F`, `%`, `IN`, `AMPS`, `VDC`, `RPM`. `--` for discrete. **Required for AI/AO/RTD/TC.** |
| 28 | **Sensor_LRV** | Sensor minimum (raw transducer). `--` for discrete. |
| 29 | **Sensor_URV** | Sensor maximum (raw transducer). `--` for discrete. |
| 30 | **Calib_LRV** | Calibrated lower range value (engineered span lower limit). `--` for discrete. |
| 31 | **Calib_URV** | Calibrated upper range value. `--` for discrete. |
| 32 | **Normal_State** | DI only: `NO` / `NC`. `--` for analog. |
| 33 | **Fail_Safe** | DO: `DE-ENRG` / `ENRGZD`. AO: `HOLD` / `LO` / `HI` / `{mA value}`. AI: `LO` / `HI`. DI: `--`. |
| 34 | **Status** | `ACTIVE`, `SPARE`, `HOT-SPARE`, `FUTURE`, `TEMP`, `QUARANTINE`, `DELETED`, `EXISTING`, `IMPORT-UNVERIFIED`. |
| 35 | **Rev** | `A`–`Z`. Increment when row changes after issue. |
| 36 | **Notes** | Free text. Source citations (`Source: WIRE-SCHEDULE-…`), wiring deviations, marshalling notes. |

#### Column Set — Optional Add-Ons (Per Round 3 Selections)

**Safety (if Safety I/O selected):**

| Column | Notes |
|---|---|
| Safety_Zone | `SZ1`, `SZ2`. Matches zone drawing. |
| Dual_Ch | `CH-A` / `CH-B` / `SINGLE`. |
| SIF_ID | (SIL-rated only) `SIF-101`. |
| SIL_Level | (SIL-rated only) `SIL 1`–`SIL 3`, or `PL c/d/e`. |
| DC_Coverage | (SIL-rated only) `None`, `Low (60–90%)`, `Medium (90–99%)`, `High (≥99%)`. |

**Hazardous Area (if Haz. Area selected):**

| Column | Notes |
|---|---|
| HazLoc_Class | `Class I Div 1/2`, `Zone 0/1/2`, `Class II Div 1/2`, `GP`. |
| Gas_Group | `A`/`B`/`C`/`D` (NEC) or `IIA`/`IIB`/`IIC` (IEC). |
| T_Rating | `T1`–`T6`. |
| IS_Barrier | `Zener`, `Galvanic`, `Fieldbus`, or `NONE`. |
| Barrier_CatNo | Barrier mfr + cat# (`MTL7787+`). |
| Voc_V | Maximum open-circuit voltage at barrier output (V). |
| Isc_mA | Maximum short-circuit current at barrier output (mA). |
| Po_mW | Maximum output power (mW). |
| Lo_mH | Maximum permitted loop inductance (mH). |
| Co_uF | Maximum permitted loop capacitance (μF). |
| IS_Calc_Ref | Loop calculation drawing/document reference (per ISA-12.06 / IEC 60079-25). |

**Security (if Sec_Zone selected):**

| Column | Notes |
|---|---|
| Sec_Zone | IEC 62443-3-2 zone identifier (e.g., `Z-CTRL-100`, `Z-SAFETY-A`). |

---

### Step 7 — Choose Save Location (Always Ask — Never Default to Desktop)

```
AskUserQuestion([
  {
    question: "Where should the I/O List file(s) be saved?",
    header: "Save Location",
    multiSelect: false,
    options: [
      { label: "Active project folder — I'll type the path", description: "WSL path (e.g., /mnt/c/Users/HIS Controls/Projects/{project}/). Recommended so the I/O list lives next to the BOM, FDS, and wire schedule." },
      { label: "Same folder as other project documents — I'll type the path", description: "WSL path. Use when project documents are on a network share." },
      { label: "Current working directory", description: "Use the current shell working directory." },
      { label: "Desktop", description: "/mnt/c/Users/HIS Controls/Desktop/ — Windows Desktop, accessed via WSL. Use only for one-off / scratch lists." }
    ]
  }
])
```

If the user picks a "type the path" option, follow up via Other to capture the WSL path. Confirm the path resolves before writing.

---

### Step 8 — Generate Output File(s)

#### CSV File

Filename: `IO-List-{project-slug}-Rev{rev}.csv`

**Row 1:** Column headers (short names below) in the order Core → Safety → Haz Area → Sec Zone, including only the optional add-on groups selected in Round 3.
**Rows 2–N:** Data rows.
- If local cross-skill or Nexus data was found, pre-populate with that data; Status = `IMPORT-UNVERIFIED`; Notes cites source.
- Otherwise, 25 blank data rows with `--` in every cell except `Status=PLACEHOLDER` and `Rev=A`.

The CSV must:
- Use comma delimiters with double-quote wrapping on text fields (RFC 4180).
- Use `--` (two dashes) for null/not-applicable. **Never blank.**
- Use `TBD` for fields awaiting engineering input.
- **No example row inside the CSV.** The example row lives in the Definitions file.

#### Markdown Definitions File

Filename: `IO-List-{project-slug}-Rev{rev}-DEFINITIONS.md`

Contents:
1. **Document header** — customer, project number, engineer-of-record, FAT date, Rev, date issued.
2. **Tag-numbering convention** — the convention selected in Round 1, with a regex and three valid examples for that convention.
3. **Column Definitions table** — every column included in this CSV, its purpose, valid values, example.
4. **Signal Type Reference** — every IO_Type code with full expansion and typical Rockwell module catalog numbers.
5. **Wire numbering convention** — the convention selected in Round 2, with worked examples for a 4-20mA loop, a discrete input, and a discrete output.
6. **Status code definitions** — every Status value explained.
7. **Fail-safe state guidance** — table from this skill's Fail-Safe Definitions section.
8. **Example row** — one fully populated AI signal (`100-FT-101 / Flow Transmitter — Cooling Water Supply / AI / 4-20mA 2-wire / Loop_Power=2-wire loop / HART=Y / …`). Clearly marked as illustrative; never paste into the CSV.
9. **How to use this list** — fill TBD as engineering progresses; increment Rev letter per row when a row changes; electrician signs off Notes at loop check; commissioning changes Status `IMPORT-UNVERIFIED` → `ACTIVE` after live test.

---

### Step 9 — Mandatory Quality Check Before Saving

Before writing any file, run all rules. Report results in the summary block at the end.

1. **Tag duplicate check.** Scan `Tag` for duplicates. Flag as `*** DUPLICATE TAG — RESOLVE ***` in Notes.
2. **Channel duplicate check.** Scan `Controller + Chassis + Slot + Channel` combinations for duplicates. Flag as `*** DUPLICATE CHANNEL — WIRING CONFLICT ***`.
3. **Wire number uniqueness.** Flag any `Wire_No` appearing on more than one row.
4. **Tag format check.** Verify `Tag` matches the convention chosen in Round 1 via regex (e.g., `^[A-Z]{2,4}-\d{1,4}[A-Z]?$` for ISA-5.1-2024 no-prefix; `^\d{2,4}-[A-Z]{2,4}-\d{1,4}[A-Z]?$` for prefixed). Skip on Customer/Other conventions.
5. **Module/IO_Type consistency.** Maintain a built-in lookup `Module_CatNo → expected IO_Type` (from the Signal Type Reference). For every row whose Module_CatNo is in the lookup, verify `IO_Type` matches. Flag mismatches as `⚠️ MODULE MISMATCH: {Module_CatNo} expects {expected}, row has {actual}`.
6. **Eng_Units required.** Every `AI` / `AO` / `RTD` / `TC` row must have non-`--` `Eng_Units`. Flag as `⚠️ ENG_UNITS MISSING`.
7. **Loop_Power required.** Every `AI` / `AO` row must have non-`--` `Loop_Power`. Flag as `⚠️ LOOP_POWER MISSING`.
8. **HART consistency.** Any row with a HART-capable module (`1756-IF8H`, `1756-OF8H`, `1734-…H`, `5069-…H`) must have `HART=Y`. Flag mismatches.
9. **Channel-base consistency.** Verify every Channel value lies within the platform-typical range for its Module_CatNo (`[0, 15]` for 16-pt 0-based; `[1, 16]` for 16-pt 1-based; per Round 1 follow-up). Flag out-of-range as `⚠️ CHANNEL OUT OF RANGE`.
10. **Spare-physical check.** For each module, count `ACTIVE + HOT-SPARE + FUTURE + TEMP` channels. Free physical channels = `Module total − that count`. Flag if free < 20% of module total: `⚠️ SPARE DEFICIT: {Module} slot {S} has {N} free of {T} channels`.
11. **Haz-area IS check.** Every row with `HazLoc_Class != GP` and `IS_Barrier != NONE` must have `Voc_V`, `Isc_mA`, `Po_mW`, `Lo_mH`, `Co_uF` populated (numeric or `TBD`) and a non-`--` `IS_Calc_Ref`. Flag gaps as `⚠️ IS ENTITY PARAMS MISSING`.
12. **Series presence check.** Every row whose Module_CatNo is a Logix-family module must have non-`--` `Series`.

Summary block:

```
I/O List Quality Check
────────────────────────
Total I/O rows:                ###
Active signals:                ###
Spare/Hot-Spare/Future/Temp:   ### / ### / ### / ###
Free physical channels:        ###  ← 20% rule per module
Duplicate tags:                0    ← or list them
Duplicate channels:            0    ← or list them
Duplicate wires:               0    ← or list them
Tag-format violations:         0    ← per Round 1 convention
Module / IO_Type mismatches:   0    ← cat# vs IO_Type lookup
Eng_Units missing:             0    ← AI/AO/RTD/TC rows
Loop_Power missing:            0    ← AI/AO rows
HART flag inconsistencies:     0
Channel out-of-range:          0    ← per channel base
Module spare deficits:         0    ← list modules below 20% free
IS entity params missing:      0    ← haz-area rows
Series missing on Logix:       0
```

---

### Step 10 — Closing Statement

After saving the file(s), provide:

1. The exact output filename(s) and full WSL path.
2. The quality check summary block.
3. **Next steps block:**

```
Next Steps
──────────────────────────────────────────────────────────────────────
1. RESOLVE TBD FIELDS — Assign rack/slot/channel once panel layout
   is finalized. I/O assignment should precede writing PLC tag names.
2. ASSIGN WIRE NUMBERS — Per the convention chosen in the interview.
   Wire numbers must appear on terminal block schedules and field
   wiring diagrams before panel build starts.
3. LOOP DRAWINGS — Generate ISA-5.4 loop diagrams for every AI/AO
   signal; cross-reference via Loop_Ref column.
4. 20% SPARE CHECK — Verify each module has spare physical channels.
   Adding a module after panel fab requires a panel door change
   at minimum.
5. IMPORT-UNVERIFIED VERIFICATION — Every IMPORT-UNVERIFIED row
   must be field-verified (signal type, wiring, terminal) before
   FAT and Status changed to ACTIVE.
6. SAFETY I/O VERIFICATION — Every SAFE-DI / SAFE-DO row must be
   cross-checked against the Safety Architecture document and the
   SISTEMA PL calculation before the safety signature is generated.
7. IS LOOP CALCULATION — Every haz-area row with an IS barrier must
   have a signed-off ISA-12.06 / IEC 60079-25 loop calculation
   referenced in IS_Calc_Ref before energization.
8. CROSS-CHECK WITH /bom — Every Module_CatNo here should appear in
   the BOM Section 1 with matching Series and FW_Required.
```

---

## Column Short Names for CSV Header Row

Core (always present):
```
Row#, Process_Id, Tag, Description, IO_Type, Loop_Type, Signal_Char,
Loop_Power, HART, Controller, Chassis, Slot, Channel, Module_CatNo,
Series, FW_Required, PLC_Tag, Raw_Addr, Wire_No, TB_ID, TB_Term,
Cable_No, Cable_Type, Field_Loc, PID_Ref, Loop_Ref, Eng_Units,
Sensor_LRV, Sensor_URV, Calib_LRV, Calib_URV, Normal_State, Fail_Safe,
Status, Rev, Notes
```

Safety add-on: `Safety_Zone, Dual_Ch, SIF_ID, SIL_Level, DC_Coverage`
Haz Area add-on: `HazLoc_Class, Gas_Group, T_Rating, IS_Barrier, Barrier_CatNo, Voc_V, Isc_mA, Po_mW, Lo_mH, Co_uF, IS_Calc_Ref`
Security add-on: `Sec_Zone`

---

## Signal Type Reference (Built-In)

| IO_Type | Expansion | Typical Rockwell Module |
|---|---|---|
| DI | 24VDC Discrete Input | 1756-IB16, 1756-IB32, 1734-IB4, 5069-IB16 |
| DO | 24VDC Discrete Output (sourcing) | 1756-OB16E, 1756-OB32, 1734-OB4E, 5069-OB16 |
| AI | 4-20mA Analog Input | 1756-IF16, 1756-IF8H, 1734-IE2C, 5069-IF8 |
| AO | 4-20mA Analog Output | 1756-OF8H, 1756-OF4, 1734-OE2C, 5069-OF8 |
| RTD | RTD Temperature Input | 1756-IR6I, 1734-IR2, 5069-IY4 |
| TC | Thermocouple Input | 1756-IT6I2, 1734-IT2I, 5069-IY4 |
| HSC | High-Speed Counter | 1756-HSC |
| HSI | High-Speed Input | 1734-HSC, 5069-HSC2xOB4 |
| SAFE-DI | Safety Discrete Input (CIP Safety) | 1791ES-IB16, 1734-IB8S, 1756-IB16S, 5069-IB16S |
| SAFE-DO | Safety Discrete Output (CIP Safety) | 1791ES-OB8EP, 1734-OB8S, 1756-OB16IS |
| COMM | Serial / Modbus / Ethernet field device — no I/O channel | N/A — comm port or EtherNet/IP adapter |

This table is the source-of-truth lookup for the Module/IO_Type quality-check rule.

---

## Wire Numbering Convention Reference (Built-In)

### Convention 1 — NFPA 79 Sequential by Panel
- **Standards basis:** NFPA 79 §13 (industrial machinery wire identification).
- **Format:** Wires numbered 101, 102, 103… within each panel section. Sub-panel or junction box starts a new range (e.g., JB1: 201–299).
- **Best for:** Machine builders, single-panel equipment, OEM builds.
- **Example for a 4-20mA AI loop:** `+` wire = `121`, `−` wire = `122`.

### Convention 2 — ISA-5.4 Device-Point (Recommended for Process Plants)
- **Standards basis:** ISA-5.4 (loop diagrams) device-point identification practice. Not a formal NFPA-79 strict scheme.
- **Format:** `{TagNumber}-{SignalDesignator}` — `FT101-+`, `FT101--`, `LSH204A-SIG`, `MV301-OPN`.
- **Best for:** Process plants and any project with > 50 field devices.
- **Why:** The wire label alone tells the electrician which instrument and which conductor, even with the loop sheet closed.

### Convention 3 — Source-Destination
- **Standards basis:** EPC and panel-shop convention; not codified in a single standard.
- **Format:** `{Panel ID}-{TB}:{Terminal}` — `PNL01-TB1:23`.
- **Best for:** Multi-panel projects with complex cable schedules.

---

## Fail-Safe State Definitions

| Signal | Fail-Safe Options | Guidance |
|---|---|---|
| DO (valve, solenoid) | `DE-ENRG` — de-energize to fail safe | Use when loss of signal = safe (valve closes, brake engages). |
| DO (valve, solenoid) | `ENRGZD` — must stay energized to fail safe | Unusual; document the reason. |
| AO (control valve) | `HOLD` — hold last value | Use only where safe; not recommended for flow-to-process. |
| AO (control valve) | `LO` — drive to 4 mA (0%) | Typical: fail closed on loss of signal. |
| AO (control valve) | `HI` — drive to 20 mA (100%) | Typical: fail open (cooling water, quench). |
| AO (control valve) | `{mA value}` — specific position | Bypass valves, blends with a specific fail position. |
| DI | `--` | Inputs do not have a fail-safe state; document the Normal State (NO/NC) instead. |
| AI | `LO` or `HI` | Instrument fail direction. Many transmitters fail LOW (4 mA); document if different. |
