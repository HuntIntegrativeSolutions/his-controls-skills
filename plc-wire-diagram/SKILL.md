---
name: plc-wire-diagram
description: Generate a panel wiring diagram package for industrial automation projects. Produces an interactive Graphviz HTML wiring diagram (terminal-block-level connections using HTML table nodes and PORT routing), a point-to-point wire schedule CSV, and a terminal block schedule CSV. Follows NFPA 79 §13 wire numbering, ISA-5.4 loop identification at termination points, UL 508A marking, and NEC Article 409 panel wiring. Optional automatic input from /io-list output.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [wiring, panel, plc, nfpa-79, isa-5.4, ul-508a, his]
    ported_from: claude-code/wire-diagram
---
# /wire-diagram — Panel Wiring Diagram Generator

**Invocation:** `/wire-diagram [project name]`

**Examples:**
- `/wire-diagram "Pump P-101 Motor Control Panel"`
- `/wire-diagram "Blending Skid — Area 300"`
- `/wire-diagram "Wastewater Lift Station No. 4"`

---

## Purpose

A complete panel wiring documentation package serving three audiences:

1. **The panel builder / electrician** — interactive Graphviz HTML wiring diagram showing every terminal connection, wire number, and device label. Opens in any browser; no CAD license required.
2. **The controls engineer** — point-to-point wire schedule CSV (generic format that imports into AutoCAD Electrical via the Wire Numbers Spreadsheet adapter, or converts to EPLAN via XML adapter).
3. **The commissioning team** — terminal block schedule CSV mapping every TB position to its wire number, signal description, and field device.

Standards alignment:
- **NFPA 79-2021 §13** — Wire identification and marking for industrial machine wiring (including the codified color rules for ungrounded, grounded, equipment grounding, and out-of-disconnect conductors)
- **ISA-5.4-1991** — Wire identification at instrument loop termination points
- **UL 508A** — Component and wire marking for industrial control panels
- **NEC Article 409** — Industrial control panel wiring and marking
- **IEC 60446** — International color identification convention (for export projects)

> **Wire-color note.** This skill cites NFPA 79 §13 as the codified source. The common Black/Red/Blue = L1/L2/L3 assignment is *industry convention plus customer spec*, not an NFPA 79 mandate — NFPA 79 §13.2 requires identification (by markers or color) but does not specify which color goes on which phase. The skill carries this convention as a default and expects the project's wire-color spec to confirm or override. NFPA 70E is workplace electrical safety and is **not** the source of wire-color rules.

---

## Output Files

Save location is **always asked** (Step 9 — never defaults to Desktop). Files emitted:

| File | Description |
|---|---|
| `hunt-wiring-{slug}-Rev{rev}-diagram.html` | Interactive Graphviz HTML wiring diagram |
| `hunt-wiring-{slug}-Rev{rev}-wire-schedule.csv` | Point-to-point wire schedule |
| `hunt-wiring-{slug}-Rev{rev}-tb-schedule.csv` | Terminal block schedule |

Filename includes `Rev{rev}` so re-running the skill never silently overwrites a prior version.

---

## Execution Flow

### MANDATORY: `AskUserQuestion` for ALL Questions

Never ask in plain text. Every question goes through `AskUserQuestion`. **This is a formal interview, not a chat.** Never assume a value the user did not provide. Defaults are only acceptable when explicitly labeled `(Recommended)` inside an `AskUserQuestion` option.

The interview runs in **four sequential rounds**.

---

### Step 1 — Interview Round 1: Project Identity, Schedule, Panel, Voltage

```
AskUserQuestion([
  {
    question: "What is the wiring-diagram revision letter?",
    header: "Doc Rev",
    multiSelect: false,
    options: [
      { label: "A — first issue (Recommended for new projects)", description: "Rev A for first issue. Increment after every change." },
      { label: "B / C / D / later", description: "Continuing an existing diagram." },
      { label: "Draft", description: "Internal working copy; do not issue." }
    ]
  },
  {
    question: "Project's required-on-site / FAT date?",
    header: "FAT Date",
    multiSelect: false,
    options: [
      { label: "Within 8 weeks — aggressive", description: "Aggressive schedule." },
      { label: "8–16 weeks — typical", description: "Typical schedule." },
      { label: "16+ weeks — long lead", description: "Standard schedule pressure." },
      { label: "Unknown / not yet set", description: "Skip schedule-relative checks." }
    ]
  },
  {
    question: "Panel type for this wiring diagram?",
    header: "Panel Type",
    multiSelect: false,
    options: [
      { label: "Combination — Power + PLC (Recommended for skids)", description: "480VAC power section AND PLC/I/O section in same panel." },
      { label: "Motor Control Panel (MCC-style)", description: "480VAC power + 120VAC or 24VDC control." },
      { label: "PLC Control Panel", description: "24VDC I/O + PLC chassis + power supplies + TB strips." },
      { label: "Junction Box / Marshalling Panel", description: "Terminal strips only — no active components." }
    ]
  },
  {
    question: "Control voltage?",
    header: "Control Voltage",
    multiSelect: false,
    options: [
      { label: "24VDC (Recommended for I/O)", description: "NFPA 79 §13.2.5 light-blue convention. Common Blue/Blue+White on +24V/COM in industry practice." },
      { label: "120VAC", description: "NFPA 79 §13.2.4 — Black=Hot (L), White=Grounded (N) for the supply circuit." },
      { label: "Both 120VAC and 24VDC", description: "Power contactor coils at 120 V; PLC I/O at 24 V. Two transformer secondary circuits." }
    ]
  }
])
```

After Round 1, ask one targeted follow-up to capture customer / project number / engineer-of-record (same pattern as `/io-list` and `/panel-layout`).

---

### Step 2 — Interview Round 2: I/O Counts (skip if /io-list ingested)

If Step 5 ingestion (below) finds an `IO-LIST-*.csv`, skip this round and use the parsed counts. Otherwise:

```
AskUserQuestion([
  {
    question: "Approximate DI count?",
    header: "DI Count",
    multiSelect: false,
    options: [
      { label: "1–16", description: "Single 16-pt module. One TB strip." },
      { label: "17–32", description: "Two modules or one 32-pt module. Two TB strips." },
      { label: "33–64", description: "Multiple modules. Separate TB section per module." },
      { label: "None", description: "Output-only or analog-only panel." }
    ]
  },
  {
    question: "Approximate DO count?",
    header: "DO Count",
    multiSelect: false,
    options: [
      { label: "1–16", description: "Single output module." },
      { label: "17–32", description: "Two modules." },
      { label: "33–64", description: "Multiple modules." },
      { label: "None", description: "Input-only or analog-only panel." }
    ]
  },
  {
    question: "Analog I/O present?",
    header: "Analog I/O",
    multiSelect: false,
    options: [
      { label: "Yes — both AI and AO", description: "Shielded 2-wire pairs to TB strips. Shield-drain TB required." },
      { label: "Yes — AI only", description: "4–20 mA inputs only." },
      { label: "Yes — AO only", description: "4–20 mA outputs only (rare; usually paired with AI)." },
      { label: "No — discrete only", description: "No analog I/O." }
    ]
  }
])
```

---

### Step 3 — Interview Round 3: Wire Numbering and Key Devices

```
AskUserQuestion([
  {
    question: "Wire-numbering convention?",
    header: "Wire Numbers",
    multiSelect: false,
    options: [
      { label: "NFPA 79 sequential by section (Recommended for OEM machines)", description: "Power 1–99, 120VAC 100–199, 24VDC 200–299, DI 300–499, DO 500–699, AI 700–899, AO 900–999." },
      { label: "ISA-5.4 device-point (Recommended for process plants)", description: "FT101-+, FT101--, MV301-OPN. Wire label tells the electrician which instrument and which conductor." },
      { label: "Source-destination (PNL01-TB1:3)", description: "Origin panel-TB-terminal. EPC and large-panel-shop convention." },
      { label: "I will assign later — use TBD placeholders", description: "Wire_No = WN-TBD; assign in CAD tool." }
    ]
  },
  {
    question: "Major powered devices in this panel? (select all that apply)",
    header: "Key Devices",
    multiSelect: true,
    options: [
      { label: "Motor starter / contactor + overload relay", description: "480VAC or 208VAC power circuit." },
      { label: "Variable Frequency Drive (VFD)", description: "L1/L2/L3 input, T1/T2/T3 output, control terminals." },
      { label: "Control transformer (480→120VAC)", description: "H1/H2/H3/H4 primary, X1/X2/X3 secondary, fused secondary." },
      { label: "24VDC power supply", description: "PSU: 120VAC input, +24VDC and 24V-COM output bus." }
    ]
  }
])
```

---

### Step 4 — Interview Round 4: Diagram Packaging Options

```
AskUserQuestion([
  {
    question: "Diagram-asset packaging?",
    header: "HTML Packaging",
    multiSelect: false,
    options: [
      { label: "Online — load Graphviz WASM from CDN (Recommended for typical use)", description: "Smaller HTML; needs internet on the rendering machine." },
      { label: "Offline — embed WASM as base64", description: "Larger HTML (~2 MB) but air-gap-portable. Use for panel-shop laptops without internet." }
    ]
  }
])
```

---

### Step 5 — Companion-Skill Ingestion (Primary Input Path)

Glob the project save location for `IO-LIST-*.csv`. If found:
- Parse it. Use `Tag`, `IO_Type`, `Wire_No`, `TB_ID`, `TB_Term`, `Cable_No`, `Cable_Type`, `Field_Loc` columns directly.
- This is the **canonical input path**. The free-text I/O paste below is the fallback.
- Cite source in Notes per row: `Source: IO-LIST-{filename}`.
- Status = `IMPORT-UNVERIFIED` until field-verified.

Also glob for:
- `BOM-*-Rev*.csv` (from `/bom`) → cross-check Module_CatNo for I/O modules and any panel-internal devices referenced in this diagram.
- Optional Nexus pull (only if requested via follow-up): `mcp__nexus__plant_summary`, `list_documented_plcs`, `generate_io_inventory`, `classify_signal_roles`, `group_io_by_area`.

If no `IO-LIST-*.csv` exists and the user has not run `/io-list` yet, fall through to Step 6.

---

### Step 6 — Free-Text I/O Paste (Fallback Only)

Only used if Step 5 found no `IO-LIST-*.csv` and Nexus was not selected.

> "Please paste your I/O list — one signal per line in the format:
> `Tag | Description | IO_Type | Wire# | TB_Strip | TB_Terminal | Field_Device_Location`
>
> Example:
> `FT-101 | Flow Transmitter | AI | 301/302 | TB-FIELD | 1/2 | Field JB-1`
> `LSH-204 | Level Switch High | DI | 201 | TB-FIELD | 5 | Tank T-200`
> `SV-301 | Solenoid Valve | DO | 401 | TB-FIELD | 8 | Valve SV-301`"

Parse tolerantly. Ask follow-up `AskUserQuestion` only on ambiguous entries. Status = `PLACEHOLDER` for parsed rows; `IMPORT-UNVERIFIED` is reserved for ingested rows.

---

### Step 7 — Build the Internal Data Model

```
PANEL MODEL:
  project_name, slug, rev, fat_date, customer, project_number, engineer_of_record
  control_voltage: "24VDC" | "120VAC" | "Both"
  panel_type: <Round 1>
  wire_convention: <Round 3>
  packaging: "Online" | "Offline"

  POWER_CIRCUIT: { devices: [...], power_voltage: "480VAC" | "208VAC" | "240VAC" | "—" }

  TERMINAL_BLOCKS:
    TB-POWER: power circuit terminals (L1/L2/L3/T1/T2/T3/PE)
    TB-CTRL: control circuit (transformer secondary, PSU output, E-stop chain)
    TB-DI / TB-DO / TB-AI / TB-AO: field-wiring TBs

  IO_POINTS: [ { tag, description, io_type, wire_numbers, tb_strip, tb_terminals, module_slot, module_channel, field_location, status, source } ]

  DEVICES: [ { id, label, type, cat_no, terminals: [ { name, wire } ] } ]
```

Apply wire numbering convention to assign wire numbers if not already provided:
- Sequential: per the NFPA 79-style ranges in Round 3.
- Device-point: `{tag}-+` / `{tag}--` for analog; `{tag}-SIG` for DI/DO; `{tag}-A` / `{tag}-B` for dual-channel safety.
- Source-destination: `{panel_id}-{TB}:{terminal_number}`.

---

### Step 8 — Generate Wire Schedule CSV

Filename: `hunt-wiring-{slug}-Rev{rev}-wire-schedule.csv`

Columns:

```
Wire_No, From_Panel, From_Device, From_Terminal, To_Panel, To_Device, To_Terminal,
Wire_Gauge, Wire_Color, Signal_Function, Signal_Type, Status, Source, Notes
```

Row rules:
- Every wire gets a row. A 4-20 mA loop = 2 rows (+ wire and − wire) plus 1 shield-drain row.
- Dual-channel safety = 3 rows (CH-A, CH-B, SHD).
- Power circuit: one row per conductor (L1, L2, L3, PE).
- Wire number must match Round-3 convention.
- `Status` per the lifecycle enum below.
- **No example row inside the CSV.** The example lives in the Definitions block in the HTML sidebar.

#### Status Lifecycle Enum (parity with `/io-list`)

`PLACEHOLDER`, `IMPORT-UNVERIFIED`, `ACTIVE`, `EXISTING`, `RETURNED`.

#### Wire-Color Reference (NFPA 79-codified vs industry convention)

| Circuit | Color | Standards basis |
|---|---|---|
| Equipment Ground (PE) | Green or Green/Yellow | **NFPA 79 §13.2.4(a) — codified** |
| Grounded conductor (neutral) | White or Gray | **NFPA 79 §13.2.4(c) — codified** |
| Out-of-disconnect (energized when main is OFF) | Orange | **NFPA 79 §13.2.4(b) — codified** |
| AC control circuit from a source other than the main supply | Yellow | **NFPA 79 §13.2.4(d) — codified** |
| 24VDC ungrounded | Light blue (where used) | **NFPA 79 §13.2.5 — codified light-blue suggestion** |
| 480/208 VAC L1 / L2 / L3 ungrounded | Black / Red / Blue (industry convention) | NFPA 79 §13.2 requires identification but does NOT codify these specific colors. Confirm against the project's wire-color spec. |
| 120 VAC ungrounded (Hot) | Black (industry convention) | Same — requires identification, color choice not codified. |
| 4-20 mA (+) / (−) | Blue / White-Blue pair (typical ISA-5.4 shielded pair) | Industry convention; ISA-5.4 requires identification at termination. |
| Shield drain | Bare or White | ISA-5.4 §5.4.1 — ground at panel end only. |

If the project has a customer wire-color spec, the **spec wins** over these defaults.

---

### Step 9 — Generate Terminal Block Schedule CSV

Filename: `hunt-wiring-{slug}-Rev{rev}-tb-schedule.csv`

One row per TB position:

```
TB_ID, Position, Wire_No_Left, Wire_No_Right, Signal_Tag, Signal_Description,
IO_Type, Field_Device, Status, Source, Notes
```

Rules:
- Group rows by TB_ID (TB-POWER / TB-CTRL / TB-DI / TB-DO / TB-AI / TB-AO).
- Include a `SPARE` row for every 5th unused terminal (20 % spare rule).
- Jumper links: mark both positions with the same wire number; `Notes = "JUMPER"`.
- Shield grounds: dedicated shield terminal row with `Wire_No = {TAG}-SHD`, `Signal_Tag = "SHIELD"`, `Notes = "Ground at panel end only — ISA-5.4"`.

---

### Step 10 — Choose Save Location (Always Ask — Never Default to Desktop)

```
AskUserQuestion([
  {
    question: "Where should the wiring-diagram files be saved?",
    header: "Save Location",
    multiSelect: false,
    options: [
      { label: "Active project folder — I'll type the path", description: "WSL path. Recommended so wiring lives next to BOM, I/O list, and panel layout." },
      { label: "Same folder as other project documents — I'll type the path", description: "WSL path. Use for network-share project folders." },
      { label: "Current working directory", description: "Use the current shell working directory." },
      { label: "Desktop", description: "/mnt/c/Users/HIS Controls/Desktop/ — Windows Desktop via WSL. Use only for one-off / scratch diagrams." }
    ]
  }
])
```

If the user picks a "type the path" option, follow up via Other.

---

### Step 11 — Generate Graphviz HTML Wiring Diagram

This is the primary deliverable. Render with the `@hpcc-js/wasm` Graphviz renderer in a self-contained HTML file. Packaging follows the Round-4 answer (Online / Offline base64-embed).

#### Color Palette (HIS Standard — same as /hunt-diagram)

| Layer | Background | Text | Border |
|---|---|---|---|
| Power Circuit (480VAC) | `#1a0a0a` | `#ffa657` | `#d9730d` |
| Control Circuit (120VAC) | `#1a1200` | `#e3b341` | `#bb8009` |
| PLC / I/O | `#0d1a2a` | `#79c0ff` | `#388bfd` |
| Terminal Blocks | `#0d2112` | `#3fb950` | `#238636` |
| Field Devices | `#1c1a21` | `#d2a8ff` | `#8957e5` |
| Safety / E-stop | `#1a0a0a` | `#f85149` | `#da3633` |
| PSU / 24VDC | `#162032` | `#a5d6ff` | `#388bfd` |
| Background | `#0d1117` | — | — |

#### Font Stack

Use a web-safe stack so non-Windows render hosts don't fall back to Times:

```
fontname="Inter, Segoe UI, Arial, sans-serif"
```

(Graphviz accepts a comma-separated list; the renderer picks the first available.)

#### HTML-Escape Rule (mandatory)

Every tag, description, and free-text field inserted into a DOT HTML label must be HTML-entity-escaped:

| Char | Entity |
|---|---|
| `&` | `&amp;` |
| `<` | `&lt;` |
| `>` | `&gt;` |
| `"` | `&quot;` |

Apply before substitution into the DOT label. A descriptive field with `<` or `&` un-escaped breaks the diagram.

#### DOT Code Structure

```dot
digraph WiringDiagram {
    label="Hunt Integrative Solutions LLC\n{project_name} Rev {rev} — Wiring Diagram — {YYYY-MM-DD}"
    labelloc="t"
    fontname="Inter, Segoe UI, Arial, sans-serif"
    fontsize=13
    fontcolor="#8b949e"
    bgcolor="#0d1117"
    rankdir=LR
    splines=ortho
    nodesep=0.5
    ranksep=1.0
    node [fontname="Inter, Segoe UI, Arial, sans-serif" fontsize=10 style="filled" penwidth=1.5]
    edge [fontname="Inter, Segoe UI, Arial, sans-serif" fontsize=9 penwidth=1.2]

    subgraph cluster_power { label="Power Circuit — 480VAC" style=filled fillcolor="#1a0a0a" color="#d9730d" fontcolor="#ffa657" /* ... */ }
    subgraph cluster_ctrl  { label="Control Circuit — 120VAC / 24VDC" style=filled fillcolor="#111208" color="#bb8009" fontcolor="#e3b341" /* ... */ }
    subgraph cluster_plc   { label="PLC / I/O" style=filled fillcolor="#0d1a2a" color="#388bfd" fontcolor="#79c0ff" /* ... */ }
    subgraph cluster_tb    { label="Terminal Blocks" style=filled fillcolor="#061410" color="#238636" fontcolor="#3fb950" /* ... */ }
    subgraph cluster_field { label="Field" style=filled fillcolor="#12101a" color="#8957e5" fontcolor="#d2a8ff" /* ... */ }
}
```

#### Terminal Block Nodes — HTML Table with PORT Routing

```dot
TB_DI [shape=none margin=0 label=<
  <TABLE BORDER="1" CELLSPACING="0" CELLPADDING="3" BGCOLOR="#0d2112">
    <TR><TD COLSPAN="4" BGCOLOR="#1a4228" ALIGN="CENTER">
      <FONT COLOR="#3fb950" POINT-SIZE="11"><B>TB-DI — Discrete Input Field Wiring</B></FONT>
    </TD></TR>
    <TR>
      <TD BGCOLOR="#111d17"><FONT COLOR="#6e7681" POINT-SIZE="9">Term</FONT></TD>
      <TD BGCOLOR="#111d17"><FONT COLOR="#6e7681" POINT-SIZE="9">Wire</FONT></TD>
      <TD BGCOLOR="#111d17"><FONT COLOR="#6e7681" POINT-SIZE="9">Tag</FONT></TD>
      <TD BGCOLOR="#111d17"><FONT COLOR="#6e7681" POINT-SIZE="9">Function</FONT></TD>
    </TR>
    <TR>
      <TD PORT="t1" BGCOLOR="#0d2112"><FONT COLOR="#3fb950">1</FONT></TD>
      <TD BGCOLOR="#0d2112"><FONT COLOR="#3fb950">201</FONT></TD>
      <TD BGCOLOR="#0d2112"><FONT COLOR="#3fb950">FT-101</FONT></TD>
      <TD BGCOLOR="#0d2112"><FONT COLOR="#3fb950">Flow Xmtr (+)</FONT></TD>
    </TR>
    <!-- ... -->
  </TABLE>
>]
```

#### I/O Module Nodes — HTML Table with PORT routing per channel

```dot
DI_Module [shape=none margin=0 label=<
  <TABLE BORDER="1" CELLSPACING="0" CELLPADDING="3" BGCOLOR="#0d1a2a">
    <TR><TD COLSPAN="3" BGCOLOR="#1c2d4f" ALIGN="CENTER">
      <FONT COLOR="#79c0ff" POINT-SIZE="11"><B>1756-IB16 — Slot 3</B></FONT>
    </TD></TR>
    <TR>
      <TD BGCOLOR="#161b22"><FONT COLOR="#6e7681" POINT-SIZE="9">Ch</FONT></TD>
      <TD BGCOLOR="#161b22"><FONT COLOR="#6e7681" POINT-SIZE="9">PLC Tag</FONT></TD>
      <TD BGCOLOR="#161b22"><FONT COLOR="#6e7681" POINT-SIZE="9">Signal</FONT></TD>
    </TR>
    <TR>
      <TD PORT="ch0" BGCOLOR="#0d1a2a"><FONT COLOR="#79c0ff">0</FONT></TD>
      <TD BGCOLOR="#0d1a2a"><FONT COLOR="#79c0ff">IO_Map:FT101_PV</FONT></TD>
      <TD BGCOLOR="#0d1a2a"><FONT COLOR="#79c0ff">FT-101 Flow</FONT></TD>
    </TR>
  </TABLE>
>]
```

#### Edges — Wire Connections with Labels

```dot
TB_DI:t1 -> DI_Module:ch0 [label="201" color="#3fb950" penwidth=1.5 fontcolor="#3fb950"]
FDS1 -> Contactor_M1 [label="L1/L2/L3" color="#ffa657" penwidth=2.5 fontcolor="#ffa657"]
EStop_PB -> Safety_Relay [label="101" color="#f85149" penwidth=2.0 fontcolor="#f85149"]
TB_DI:t1 -> FT101_Field [label="201↗ Field" color="#d2a8ff" penwidth=1.2 style=dashed fontcolor="#d2a8ff"]
```

#### DOT Code Rules

1. Every I/O point gets a TB row AND a module channel row. The edge connects the TB PORT to the module PORT.
2. Spare TB rows use muted color `#3d6640`.
3. Power circuit edges: `penwidth=2.5`, `#ffa657`.
4. Safety / E-stop edges: `penwidth=2.0`, `#f85149`.
5. 24 V logic: `#3fb950`, `penwidth=1.5`.
6. Field cables (panel TB → field stub): dashed, `#d2a8ff`.
7. Analog pairs: `#79c0ff` for `+`, dashed blue/white for `−`.
8. Wire-number labels appear on every edge.

#### Scale Guidance

- ≤ 32 I/O points: full diagram with field-stub nodes.
- 33–64 I/O points: cluster per DIN rail section; omit individual field stubs.
- > 64 I/O points: emit one diagram per I/O section (`-di-section.html`, `-do-section.html`, etc.).

---

### Step 12 — HTML Shell

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>{project_name} Rev {rev} — Wiring Diagram | Hunt Integrative Solutions</title>
<style>
  /* HIS dark theme — identical to /hunt-diagram */
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { height: 100%; background: #0d1117; color: #e6edf3; font-family: 'Inter', 'Segoe UI', Arial, sans-serif; }
  /* header, toolbar, canvas, sidebar — copy from MCC demo pattern */
</style>
</head>
<body>
  <!-- Header: HIS badge, project name, Rev, date -->
  <!-- Toolbar: Zoom In / Out / Fit / Export PNG / Export SVG -->
  <!-- Canvas div for Graphviz SVG -->
  <!-- Sidebar tabs: Wire Schedule / Legend / Standards / Definitions+Example -->

  {% if packaging == "Online" %}
  <script src="https://cdn.jsdelivr.net/npm/@hpcc-js/wasm@2/dist/graphviz.umd.js"></script>
  {% else %}
  <script>/* base64-embedded WASM module loader for offline use */</script>
  {% endif %}
  <script>
    const dot = `{DOT_CODE}`;
    // graphviz.instance().then(g => render(g.dot(dot)))
    // Zoom / pan / export
  </script>
</body>
</html>
```

Sidebar `Definitions+Example` tab carries the canonical example row (in place of the in-CSV example row that previous versions of this skill included — that was a Rev-A defect mode and has been removed).

---

### Step 13 — Mandatory Quality Check Before Writing Files

1. **Wire-number uniqueness.** No two different signals share a wire number.
2. **Terminal balance.** Every TB terminal that appears as `From` in the wire schedule has a `To`. Open-ended terminals = wiring error.
3. **Module / TB pairing.** Every module channel row has exactly one TB row connected. Channels with no TB → open input. TBs with no channel → disconnected signal.
4. **Ground continuity.** Every panel section has at least one PE terminal run to chassis ground.
5. **Spare TB coverage.** Each TB strip has ≥ 20 % spare terminals.
6. **Wire-color / voltage consistency.** Every wire whose `Wire_Color = Blue` is on a 24VDC or analog circuit (not on 120VAC or 480VAC). Every Black on Hot/L1. Every Green or Green/Yellow on PE. Every White on Neutral. Flag mismatches.
7. **Duplicate TB position.** No two rows share `TB_ID + Position`.
8. **Dual-channel safety symmetry.** Every row whose Wire_No suffix = `-A` has a corresponding `-B` row on the same tag.
9. **Jumper-link consistency.** Rows marked `JUMPER` appear in pairs with the same wire number on adjacent terminals.
10. **HTML-escape compliance.** Every tag, description, and free-text field destined for the DOT HTML label has been entity-escaped.

Summary block:

```
Wire Diagram Quality Check
──────────────────────────
Total I/O points:                ###
Wires in schedule:               ###
Duplicate wire numbers:          0  ← or list
Open terminals:                  0  ← or list
Channel/TB pairing errors:       0  ← or list
Missing PE grounds:              0  ← or list
TB spare coverage:               ##%  ← flag if < 20 %
Wire-color / voltage violations: 0  ← or list
Duplicate TB positions:          0  ← or list
Dual-channel asymmetries:        0  ← or list
Jumper-link mismatches:          0  ← or list
HTML-escape gaps:                0  ← must be 0
```

---

### Step 14 — Closing Statement

After all files are written:

1. List exact output filenames and full WSL paths.
2. Quality Check summary block.
3. **Next Steps:**

```
Next Steps
──────────────────────────────────────────────────────────────────────
1. OPEN THE HTML — The diagram is interactive. Zoom, pan, export PNG/SVG
   from the toolbar.

2. IMPORT WIRE SCHEDULE TO CAD — AutoCAD Electrical: Import → Wire
   Numbers → From Spreadsheet. EPLAN: convert CSV to EPLAN XML via
   adapter. Note: this CSV is a generic format — ACE / EPLAN may
   require column re-mapping.

3. VERIFY IMPORT-UNVERIFIED ROWS — Any row with that status must be
   field-confirmed before panel build. Update Status to ACTIVE after
   loop check.

4. ASSIGN TBD WIRE NUMBERS — Replace WN-TBD before panel fab. Wire
   numbers must appear on TB labels per UL 508A.

5. TERMINAL BLOCK LABELS — Print TB schedule and attach to DIN rail
   before shipping. Include TB_ID, Position, Wire_No.

6. UL 508A MARKING — Every TB strip labeled with panel ID, TB
   designation, and voltage rating. Cross-reference /panel-layout
   for the full 508A checklist.

7. REVISION CONTROL — Increment Rev on any change after first issue.
   Filename includes Rev — re-running the skill will not overwrite
   previous revisions.
```

---

## Wire Numbering Reference (Built-In)

### NFPA 79-style Sequential by Section (Recommended for OEM machines)

| Range | Circuit |
|---|---|
| 1–99 | 480VAC / 208VAC power circuit |
| 100–199 | 120VAC control circuit |
| 200–299 | 24VDC bus (PSU output, I/O commons) |
| 300–499 | Discrete Input field signals |
| 500–699 | Discrete Output field signals |
| 700–899 | Analog Input field signals |
| 900–999 | Analog Output field signals |
| PE | Protective Earth / Ground |

### ISA-5.4 Device-Point Convention

Format: `{ISA-Tag}-{Function}`

| Code | Meaning | Example |
|---|---|---|
| `+` | Positive conductor (DC+, AI+, AO+) | `FT-101-+` |
| `-` | Negative / return conductor | `FT-101--` |
| `SIG` | Single-conductor discrete signal | `LSH-204-SIG` |
| `COM` | Common / return for discretes | `DI-COM` |
| `A`, `B` | Dual-channel safety conductors | `SZ1-ES01-A`, `SZ1-ES01-B` |
| `SHD` | Shield drain | `FT-101-SHD` |
| `OPN` / `CLS` | Open / Close command | `MV-301-OPN` |
| `FB-O` / `FB-C` | Open / Close feedback | `MV-301-FB-O` |
| `RUN` / `FLT` | Run / Fault feedback | `P-101-RUN` |

---

## Companion Skills

| Skill | Relationship to /wire-diagram |
|---|---|
| `/io-list` | **Primary input.** `IO-LIST-*.csv` is auto-ingested in Step 5. Run `/io-list` first. |
| `/bom` | Cross-check: every Module_CatNo and panel-internal device should appear in the BOM Section 1. |
| `/panel-layout` | TB widths and counts in this diagram should match the panel-layout component schedule. |
| `/fds` | Functional spec — defines what this panel must do. |

> **Out of scope:** ISA-5.4 *per-loop* diagrams (field instrument → IS barrier → I/O channel → controller → HMI) are a different deliverable and not produced by this skill. Use a CAD tool or generate them manually from the I/O List + this wiring diagram.
