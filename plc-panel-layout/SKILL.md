---
name: plc-panel-layout
description: Generate a panel layout specification package for industrial control panels. Produces a DIN rail loading plan (ASCII art + CSV component schedule), power budget spreadsheet (breaker sizing, wire gauge, PSU loading), and a UL 508A compliance checklist. Works for MCC panels, PLC control panels, and combination power+control enclosures. Follows UL 508A, NFPA 79, NEC Article 409, NEC 110.26 working clearance (with Cond 1/2/3 differentiation), NEC Table 430.250 motor FLA, and NEMA 250 enclosure ratings.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [panel-layout, power-budget, ul-508a, nfpa-79, nec-409, his]
    ported_from: claude-code/panel-layout
---
# /panel-layout — Panel Layout Specification Generator

**Invocation:** `/panel-layout [project name]`

**Examples:**
- `/panel-layout "Pump P-101 Motor Control Panel"`
- `/panel-layout "Blending Skid — Area 300"`
- `/panel-layout "Wastewater Lift Station No. 4"`

---

## Purpose

A complete panel layout documentation package that serves four audiences:

1. **The panel builder** — DIN rail loading plan with every component's left-to-right position, dimensions, and clearances. Enough to build the panel without a CAD drawing.
2. **The controls engineer** — power budget table with branch circuit sizing, wire gauge per NEC 310.16 ampacity tables, PSU loading, transformer kVA.
3. **The project manager / customer** — UL 508A compliance checklist covering spacing, marking, SCCR, grounding, and enclosure type.
4. **The drafter** — a structured CSV component schedule ready to import into AutoCAD Electrical or EPLAN.

Standards alignment:
- **UL 508A** — Industrial control panel construction; spacing, marking, SCCR (Supplement SB combination ratings)
- **NFPA 79-2021** — Industrial machinery; panel wiring and component requirements
- **NEC Article 409** — Industrial control panel installation; branch protection
- **NEC 110.26(A)(1)** — Working clearance, conditional on voltage and condition (Cond 1 / 2 / 3)
- **NEC 310.16 / 310.12** — Conductor ampacity (75°C column for industrial)
- **NEC Table 430.250** — Motor FLA, full table by voltage column
- **NEMA 250** — Enclosure type selection (1, 12, 12K, 4, 4X, 7, 9)
- **IEC 60529 / IP ratings** — International equivalent
- **NFPA 70E §130.5(H)** — Arc-flash warning labels on panels > 50 V nominal

---

## Output Files

Save location is **always asked** (Step 1 — never defaults to Desktop). Files emitted:

| File | Description |
|---|---|
| `hunt-panel-layout-{slug}-Rev{rev}.md` | Panel layout specification document (main deliverable) |
| `hunt-panel-layout-{slug}-Rev{rev}-component-schedule.csv` | Component schedule with position, dimensions, catalog numbers |
| `hunt-panel-layout-{slug}-Rev{rev}-power-budget.csv` | Branch circuit sizing, wire gauge, PSU load |

---

## Execution Flow

### MANDATORY: `AskUserQuestion` for ALL Questions

Never ask questions in plain text. Every question to the user goes through `AskUserQuestion`. **This is a formal interview, not a chat.** Never assume a value the user did not provide. Defaults are only acceptable when explicitly labeled `(Recommended)` inside an `AskUserQuestion` option, never silently in code.

The interview runs in **four sequential rounds**. Complete each round before proceeding.

---

### Step 1 — Interview Round 1: Project Identity, Schedule, Supply, and Working Condition

```
AskUserQuestion([
  {
    question: "What is the panel-layout revision letter?",
    header: "Doc Rev",
    multiSelect: false,
    options: [
      { label: "A — first issue (Recommended for new projects)", description: "Use Rev A for first issue. Increment after every change." },
      { label: "B / C / D / later", description: "Continuing an existing layout. I'll ask which letter." },
      { label: "Draft", description: "Internal working copy; do not issue." }
    ]
  },
  {
    question: "Project's required-on-site / FAT date?",
    header: "FAT Date",
    multiSelect: false,
    options: [
      { label: "Within 8 weeks — aggressive", description: "Long-lead enclosures (Hoffman/Rittal stock variants) are critical-path." },
      { label: "8–16 weeks — typical", description: "Standard schedule." },
      { label: "16+ weeks — long lead", description: "Custom enclosures and large VFDs feasible." },
      { label: "Unknown / not yet set", description: "Skip schedule-relative checks." }
    ]
  },
  {
    question: "What is the supply voltage for this panel?",
    header: "Supply Voltage",
    multiSelect: false,
    options: [
      { label: "480VAC / 3-Phase (Recommended for industrial)", description: "Service-entrance disconnect required. Branch protection per NEC 430." },
      { label: "208VAC / 3-Phase", description: "Common in commercial / light industrial." },
      { label: "240VAC / 1-Phase or 3-Phase", description: "Drives branch breaker selection accordingly." },
      { label: "120VAC / 1-Phase only", description: "Small panels, control-only enclosures, no main 3-phase." }
    ]
  },
  {
    question: "NEC 110.26(A)(1) working clearance condition? (Determines required depth in front of panel)",
    header: "NEC Cond",
    multiSelect: false,
    options: [
      { label: "Condition 1 — exposed live parts on one side, no live or grounded parts opposite (Recommended for most panels)", description: "≤600V: 3 ft / 914 mm depth required." },
      { label: "Condition 2 — exposed live parts on one side, grounded parts opposite", description: "≤600V: 3.5 ft / 1067 mm depth required." },
      { label: "Condition 3 — exposed live parts on both sides of working space", description: "≤600V: 4 ft / 1219 mm depth required." }
    ]
  }
])
```

After Round 1, ask one targeted follow-up to capture customer / project number / engineer-of-record:

```
AskUserQuestion([
  {
    question: "Capture document header info — customer / project number / engineer-of-record",
    header: "Doc Header",
    multiSelect: false,
    options: [
      { label: "I'll type all three", description: "You'll be prompted for customer, HIS project number, EOR initials. Today's date is auto-filled." },
      { label: "Use placeholders for now", description: "Header set to {{TBD}}. Must be resolved before issuing." }
    ]
  }
])
```

---

### Step 2 — Interview Round 2: Enclosure and Size

```
AskUserQuestion([
  {
    question: "What NEMA enclosure type?",
    header: "Enclosure Type",
    multiSelect: false,
    options: [
      { label: "NEMA 12 (Recommended — indoor industrial)", description: "IP54 equiv. Dust, falling dirt, dripping. Standard plant floor." },
      { label: "NEMA 12K (with knockouts) / NEMA 4 / NEMA 4X", description: "12K = factory knockouts. 4 = rain-tight. 4X = stainless or fiberglass corrosion-resistant." },
      { label: "NEMA 7 / NEMA 9 (Hazardous Location)", description: "7 = Class I gas/vapor (flameproof). 9 = Class II dust. Drives explosion-proof or purged construction." },
      { label: "NEMA 1 (indoor general purpose)", description: "Office / server room only. NOT for factory floor." }
    ]
  },
  {
    question: "Approximate panel dimensions?",
    header: "Panel Size",
    multiSelect: false,
    options: [
      { label: "Small — 24\"W × 24\"H × 8\"D", description: "Up to ~4 DIN rails. Single-door. Small skid or standalone machine." },
      { label: "Medium — 36\"W × 36\"H × 12\"D or 24\"W × 48\"H × 12\"D", description: "Up to ~8 DIN rails. Most common MCC or PLC panel." },
      { label: "Large — 48\"W × 72\"H × 16\"D or larger", description: "Free-standing. Multiple sections. VFDs, large chassis, multiple starters." },
      { label: "Custom — I'll specify dimensions", description: "Enter H × W × D in inches in a follow-up." }
    ]
  }
])
```

If NEMA 7 / 9 was selected, follow up with hazardous-area class question (Class I Div 1/2 / Class II Div 1/2 / Zone 0/1/2) — same option set as `/io-list` Step 3 follow-up.

---

### Step 3 — Interview Round 3: Components, PLC Platform, and Door-Mounted Devices

```
AskUserQuestion([
  {
    question: "Major power components in this panel? (select all that apply)",
    header: "Power Components",
    multiSelect: true,
    options: [
      { label: "Motor starter / contactor + overload relay", description: "IEC contactor 45–90 mm wide depending on HP." },
      { label: "Variable Frequency Drive(s) (VFD)", description: "1–5 HP: ~80–120 mm. 10–25 HP: ~150–200 mm. 50+ HP: usually separate enclosure or MCC bucket." },
      { label: "Control transformer (480→120VAC)", description: "CPT 50–500 VA typical. Panel-mount." },
      { label: "24VDC power supply", description: "1606-XLP / Phoenix QUINT. 10–20 A typical for PLC I/O." }
    ]
  },
  {
    question: "PLC / controller platform in this panel?",
    header: "PLC Platform",
    multiSelect: false,
    options: [
      { label: "Rockwell ControlLogix (1756 chassis)", description: "4-slot: 306 mm. 7-slot: 395 mm. 10-slot: 486 mm. 13-slot: 577 mm." },
      { label: "Rockwell CompactLogix 5380 (5069 chassis) (Recommended for new mid-range)", description: "Modular backplane. Common for medium machine control." },
      { label: "Rockwell legacy CompactLogix 5370 (1769) / MicroLogix", description: "Right-expand modules. Each module ~25 mm wide." },
      { label: "No PLC — relay logic / hardwired", description: "Terminal strips and relays only." }
    ]
  },
  {
    question: "Door-mounted devices? (select all that apply)",
    header: "Door Devices",
    multiSelect: true,
    options: [
      { label: "Operator pushbuttons (Start / Stop / E-stop / Reset)", description: "30 mm round cutouts; door mount." },
      { label: "Pilot lights / indicators", description: "Run, Fault, Power, Mode indicators." },
      { label: "Selector switch / keyswitch / mode switch", description: "2 / 3-position rotary selectors." },
      { label: "HMI terminal", description: "PanelView Plus / Industrial monitor; cutout sized to terminal." }
    ]
  }
])
```

---

### Step 4 — Interview Round 4: Motor / Load Data (Conditional)

If motor starters or VFDs were selected in Round 3:

```
AskUserQuestion([
  {
    question: "Motor count and HP rating?",
    header: "Motor Data",
    multiSelect: false,
    options: [
      { label: "1–2 motors, ≤ 10 HP each", description: "Small panel. NEMA starter or IEC contactor with manual motor protector." },
      { label: "3–5 motors, ≤ 25 HP each", description: "Medium panel. Section power on bottom rails, control on top." },
      { label: "6+ motors or any motor > 25 HP", description: "Large panel. VFDs strongly recommended for > 10 HP." },
      { label: "No motors — control / instrumentation only", description: "No motor branch circuits." }
    ]
  }
])
```

If 6+ motors or > 25 HP selected, follow up via Other to capture each motor's HP and FLA so the power-budget computation can use real values rather than NEC table defaults.

---

### Step 5 — Companion-Skill Ingestion + Optional Nexus Pull

In order of preference:

1. **Local cross-skill artifacts.** Glob the project save location for:
   - `IO-LIST-*.csv` → count modules per type, derive PLC chassis loading; cite source.
   - `BOM-*-Rev*.csv` → cross-check component cat#'s and net pricing; pre-fill component schedule.
   - `WIRE-SCHEDULE-*.csv` / `TB-SCHEDULE-*.csv` → cross-check terminal block widths and counts.
2. **Nexus** (if requested by the user via follow-up):
   ```
   mcp__nexus__plant_summary()
   mcp__nexus__list_documented_plcs()
   mcp__nexus__generate_io_inventory(plc_id: <selected>)
   mcp__nexus__group_io_by_area(plc_id: <selected>)
   ```
3. **Neither** — proceed with the interview answers only. Quantities marked `TBD` in the component schedule.

If both local cross-skill files AND Nexus exist, prefer the local file and flag any discrepancy.

---

### Step 6 — Research (Scoped)

After the interview, fetch the manufacturer datasheet relevant to the selected platform and components, *not* a list of blog posts. Use Scrapling to fetch the *PowerFlex 525 / 755 user manual* (for VFD efficiency), the *1606-XLP datasheet* (for PSU efficiency at load), and the *control transformer datasheet* (for no-load + load loss). Cite the actual efficiency numbers in the heat-dissipation calc rather than rule-of-thumbs. If a fetch fails, fall back to the Built-In ROT table below and tag the values `[ROT — verify with datasheet]`.

---

### Step 7 — Build the Internal Data Model

```
PANEL MODEL:
  project_name, slug, rev, fat_date, customer, project_number, engineer_of_record
  enclosure: { type, width_in, height_in, depth_in, nema_rating, manufacturer_suggestion }
  supply: { voltage, phases, hz, condition (NEC 110.26 Cond 1/2/3) }
  din_rail_usable_width_mm: (panel_width_in × 25.4) − 80   ← 40 mm margin each side
  din_rail_count: estimated from component heights and total length needed

  COMPONENTS: [ { id, label, type, cat_no, manufacturer, width_mm, height_mm, din_rail_section, position_from_left_mm, voltage, current_A, power_W, sccr_kAIC, source } ]

  POWER_BUDGET: {
    main_supply_voltage, main_breaker_A, sccr_panel_kAIC,
    motor_branch_circuits: [ { motor_id, hp, fla_nameplate_A, fla_nec_table_A, breaker_A, wire_AWG, wire_type, sizing_method } ],
    control_transformer: { kva, primary_fla, secondary_fla, secondary_fuse_A },
    psu_24vdc: { input_W, output_A_24v, load_W_estimated, margin_pct },
    total_panel_load_kVA, total_internal_heat_W
  }

  DOOR_DEVICES: [ { tag, type, cutout_size, location_h_in, location_w_in, label } ]
```

---

### Step 8 — Built-In Reference Tables

#### NEC Table 430.250 — Motor FLA (Three-Phase AC Induction)

| HP | 230 V 3PH | 460 V 3PH | 575 V 3PH |
|---|---|---|---|
| 1 | 4.2 A | 2.1 A | 1.7 A |
| 1.5 | 6.0 A | 3.0 A | 2.4 A |
| 2 | 6.8 A | 3.4 A | 2.7 A |
| 3 | 9.6 A | 4.8 A | 3.9 A |
| 5 | 15.2 A | 7.6 A | 6.1 A |
| 7.5 | 22 A | 11 A | 9 A |
| 10 | 28 A | 14 A | 11 A |
| 15 | 42 A | 21 A | 17 A |
| 20 | 54 A | 27 A | 22 A |
| 25 | 68 A | 34 A | 27 A |
| 30 | 80 A | 40 A | 32 A |
| 40 | 104 A | 52 A | 41 A |
| 50 | 130 A | 65 A | 52 A |
| 60 | 154 A | 77 A | 62 A |
| 75 | 192 A | 96 A | 77 A |
| 100 | 248 A | 124 A | 99 A |
| 125 | 312 A | 156 A | 125 A |
| 150 | 360 A | 180 A | 144 A |
| 200 | 480 A | 240 A | 192 A |

Read the column matching the supply voltage answered in Round 1. **Do not mix columns.**

#### NEC 110.26(A)(1) — Working Clearance (≤ 600 V)

| Condition | Description | Required depth |
|---|---|---|
| Cond 1 | Exposed live on one side, no live/grounded opposite | 3 ft / 914 mm |
| Cond 2 | Exposed live on one side, grounded parts opposite | 3.5 ft / 1067 mm |
| Cond 3 | Exposed live on both sides of working space | 4 ft / 1219 mm |

Use the value matching the answer to the Round 1 NEC Cond question.

#### Component DIN Rail Widths

| Component | Width (mm) | Notes |
|---|---|---|
| 1756-A4 / A7 / A10 / A13 / A17 chassis | 306 / 395 / 486 / 577 / 668 | DIN mount |
| 5069 CompactLogix 16-slot | 450 | Modular backplane |
| 1734-AENTR + power bus | 60 + 22 per module | Right-expandable |
| IEC contactor NEMA 0–1 (≤ 10 A) | 45 | 100-C09, 100-C23 |
| IEC contactor NEMA 2 (18–25 A) | 55 | 100-C37, 100-C43 |
| IEC contactor NEMA 3 (30–45 A) | 70 | 100-C60, 100-C85 |
| Manual Motor Protector (≤ 12 A) | 45 | 140M-C2E / 140MT |
| Circuit breaker 1-pole 15–30 A | 18 | 1492-SPM1 |
| Circuit breaker 3-pole 30–100 A | 53–80 | 140G series |
| 24VDC PSU 10 A (110 W) | 65 | 1606-XLP95E |
| 24VDC PSU 20 A (240 W) | 95 | 1606-XLP240E |
| Control transformer 100 VA | 100 | 1497-A |
| Control transformer 500 VA | 140 | 1497-B |
| PowerFlex 525 1 / 5 / 10 HP | 85 / 110 / 140 | 25B-D2P3 / 8P0 / 012N114 |
| PowerFlex 755 25 HP | 195 | 20F-D027 |
| Terminal block (TS-35, 2.5 mm²) | 6 | 1492-W3 per terminal |
| Terminal block (TS-35, 6 mm²) | 8 | 1492-W6 per terminal |
| End bracket | 10 | one per rail end |

#### Heat-Dissipation Rules of Thumb (use only if datasheet fetch fails)

`[ROT]` tags must remain in the output if these are used.

| Source | Rule |
|---|---|
| PLC chassis | 30 W per ControlLogix module slot (active); 5 W idle |
| VFD | ~3 % × rated motor HP × 746 W = thermal dissipation at full load |
| Control transformer | ~5 % × rated kVA × 1000 W |
| 24VDC PSU | ~15 % × rated output W |
| Contactor coil | ~5 W each |

#### NEC 310.16 — Conductor Ampacity (75 °C column, copper)

| Wire AWG | Ampacity |
|---|---|
| 14 | 20 A (limited to 15 A for branch protection per NEC 240.4(D)) |
| 12 | 25 A (limited to 20 A) |
| 10 | 35 A (limited to 30 A) |
| 8 | 50 A |
| 6 | 65 A |
| 4 | 85 A |
| 3 | 100 A |
| 2 | 115 A |

Motor branch wire = NEC 430.22: ≥ 125 % × motor FLA, then round up to next AWG.
Motor branch breaker = NEC 430.52: 250 % × FLA max for inverse-time circuit breaker; 150 % typical for HACR.
Control circuit conductor = NEC 725.49: 16 AWG minimum (Class 1).
I/O signal wire = 18 AWG minimum, shielded for analog (ISA-5.4).

#### Wire Duct Fill (UL 508A §15.2 / NEC 376.22)

Fill ≤ 40 % of internal cross-section area. Compute as:

```
Sum_of_wire_areas = Σ ( Wire_OD² × π / 4 )  for every wire in the duct
Fill_pct = 100 × Sum_of_wire_areas / (Duct_W × Duct_H)
PASS if Fill_pct ≤ 40
```

Typical wire ODs: 18 AWG MTW = 2.5 mm; 16 AWG = 2.8 mm; 14 AWG = 3.2 mm; 12 AWG = 3.5 mm.

#### SCCR — UL 508A Combination Rating (Supplement SB)

Panel SCCR = min(component SCCR_kAIC, …). When a low-SCCR component is the bottleneck:
1. Check the manufacturer's UL-listed combination rating with an upstream protective device (e.g., 100-C09 contactor + 140M-C2E MMP gives a higher combo rating than the contactor alone).
2. Substitute the bottleneck component with a higher-SCCR equivalent.
3. Add an upstream current-limiting fuse to raise the let-through to within the downstream's SCCR.

Reference: UL 508A Supplement SB Tables SB4.1 / SB4.2 for series ratings.

---

### Step 9 — Choose Save Location (Always Ask — Never Default to Desktop)

```
AskUserQuestion([
  {
    question: "Where should the panel-layout files be saved?",
    header: "Save Location",
    multiSelect: false,
    options: [
      { label: "Active project folder — I'll type the path", description: "WSL path (e.g., /mnt/c/Users/HIS Controls/Projects/{project}/). Recommended so the layout lives next to the BOM, I/O list, and wire schedule." },
      { label: "Same folder as other project documents — I'll type the path", description: "WSL path. Use when project documents are on a network share." },
      { label: "Current working directory", description: "Use the current shell working directory." },
      { label: "Desktop", description: "/mnt/c/Users/HIS Controls/Desktop/ — Windows Desktop via WSL. Use only for one-off / scratch layouts." }
    ]
  }
])
```

If the user picks a "type the path" option, follow up via Other to capture the WSL path.

---

### Step 10 — Generate Output Files

#### Component Schedule CSV

Filename: `hunt-panel-layout-{slug}-Rev{rev}-component-schedule.csv`

Columns:
```
Row#, Section, DIN_Rail, Position_mm, Tag, Type, Cat_No, Manufacturer, Width_mm,
Height_mm, Voltage, Current_A, Power_W, SCCR_kAIC, Source, Notes
```

Rows: every power component, every PLC module, every TB strip, every door-mounted device. Use `--` for non-applicable cells; `TBD` for awaiting input.

#### Power Budget CSV

Filename: `hunt-panel-layout-{slug}-Rev{rev}-power-budget.csv`

Columns:
```
Row#, Circuit, Load_Type, Voltage, FLA_or_Continuous_A, Sizing_Factor,
Breaker_A, Wire_AWG, Wire_Type, NEC_Article, Notes
```

Rows: every motor branch, control transformer primary and secondary, 24VDC PSU input, every branch loading the PSU, control circuit branches.

#### Panel Layout Specification Document

Filename: `hunt-panel-layout-{slug}-Rev{rev}.md`

Document structure (each section populated from the data model):

1. **Header** — customer, project number, EOR, FAT date, Rev, date issued.
2. **Enclosure Specification** — type, dimensions, manufacturer suggestion, supply voltage / phase / Hz, NEC 110.26 Cond, working clearance required, SCCR required, arc-flash label requirement, UL listing.
3. **DIN Rail Layout Plan** — one ASCII section per rail with components left-to-right; live computation of net DIN rail per section: `(panel_W_in × 25.4) − 80 − 120 = net_mm` (margins + duct allowance).
4. **Door Layout** — one row per Round-3 door device; device, cutout size, location, label.
5. **Wire Duct Sizing & Routing** — actual fill percentage computed per the §15.2 formula; each duct pass / fail.
6. **Grounding & Bonding** — NFPA 79 §8 / UL 508A §17.3 checklist.
7. **UL 508A Compliance Checklist** — spacing per §5/§11 with the right NEC 110.26 depth value pulled from the Cond answer; component marking per §6; SCCR per Supplement SB with combination-rating walkthrough; overcurrent protection per NEC 409.21.
8. **Thermal Management** — heat dissipation table populated from datasheet fetches (or `[ROT]` tagged fallback). If total > 200 W in NEMA 12 (or > 100 W in NEMA 4X), recommend cooling.
9. **Recommended Next Steps** — manufacturer-quote step, SCCR verification, motor nameplate FLA reconciliation, run `/wire-diagram` to feed component schedule into wiring docs, run `/io-list` if not done.

---

### Step 11 — Mandatory Quality Check Before Saving

Run all rules; report results in the summary block.

1. **Cat# completeness.** Every COMPONENT has a non-`TBD` Cat_No or status `PLACEHOLDER`. Flag any HARDWARE row missing both.
2. **Motor branch sizing.** For each motor branch: breaker between `1.50 × FLA` and `2.50 × FLA` per NEC 430.52 Table; wire AWG ampacity ≥ `1.25 × FLA` per NEC 430.22. Flag mismatches.
3. **DIN rail width.** Sum of component widths plus inter-component spacing (6 mm per gap) ≤ Net DIN rail per section. Flag overflows.
4. **Voltage labeling.** Every panel section ≥ 50 V triggers an arc-flash warning label requirement (NFPA 70E §130.5(H)). Flag missing.
5. **NEMA-vs-supply sanity.** NEMA 1 + supply ≥ 480 V → `⚠️ NEMA 1 NOT FOR HIGH-VOLTAGE INDUSTRIAL`. NEMA 7/9 selected with no haz-class follow-up answer → `⚠️ HAZ CLASS NOT CAPTURED`.
6. **SCCR floor.** Lowest component SCCR_kAIC < 5 → `⚠️ COMPONENT SCCR < 5 kAIC; LIKELY UNDER PANEL REQUIREMENT — verify combination rating`.
7. **Wire duct fill.** Computed fill > 40 % → `⚠️ DUCT OVERSIZE — UL 508A §15.2`.
8. **Working clearance.** Required depth (from NEC Cond answer) printed in the doc. Flag if customer-provided panel-room depth (asked via Other in Round 1 follow-up) < required.

Summary block:

```
Panel Layout Quality Check
──────────────────────────
Components specified:           ###
TBD / PLACEHOLDER components:   ###
Motor branch sizing errors:     0  ← or list
DIN rail overflows:             0  ← or list
Voltage labels missing:         0  ← or list
NEMA-vs-supply warnings:        0  ← or list
SCCR floor warnings:            0  ← or list
Duct fill overflows:            0  ← or list
Working clearance violations:   0  ← or list
Total internal heat:            ### W  ← cooling recommended above 200 W
Panel SCCR (calculated):        ### kAIC
```

---

### Step 12 — Closing Statement

After all files are written:

1. List exact output filenames and full WSL path.
2. Quality Check summary block.
3. **Next Steps:**

```
Next Steps
──────────────────────────────────────────────────────────────────────
1. CONFIRM PANEL DIMENSIONS — Get a firm panel quote from a vendor
   (Hoffman, Rittal, Hammond) before finalizing DIN rail layout.
   Component widths may require upsizing.

2. SCCR VERIFICATION — Get available fault current from the customer's
   electrical engineer. Verify every component's kAIC. Apply UL 508A
   Supplement SB combination ratings where a single component is the
   bottleneck.

3. MOTOR NAMEPLATE FLA — NEC Table 430.250 values are sizing defaults.
   Once nameplate FLA is known, reconcile overload relay setting per
   NEC 430.32 (115–125 % × nameplate FLA for ≤ 40 °C SF).

4. WIRING DIAGRAMS — Run /wire-diagram. The component schedule CSV
   from this skill feeds directly into it.

5. COMPONENT SCHEDULE → CAD — Import the component schedule CSV into
   AutoCAD Electrical or EPLAN to auto-place components.

6. THERMAL REVIEW — If total heat > 200 W in NEMA 12 (sealed) or
   > 100 W in NEMA 4X, add cooling: fan + filter (Hoffman ASYSTAT)
   for NEMA 12; heat exchanger (Hoffman MFPHE) for NEMA 4X.

7. REVISION CONTROL — Increment Rev letter on any change after first
   issue. Panel shop must never build from a superseded revision.
```

---

## Companion Skills

| Skill | Relationship |
|---|---|
| `/io-list` | I/O list — module quantities feed PLC chassis sizing on the appropriate DIN rail |
| `/bom` | Hardware BOM — catalog numbers and net pricing cross-checked against the component schedule |
| `/wire-diagram` | Wiring diagram — TB widths and counts verified against this layout |
| `/fds` | Functional spec — defines what the panel must control |
