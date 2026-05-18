---
name: plc-bom
description: Generate a standards-aligned Hardware BOM (Bill of Materials) template for industrial automation projects. Covers every hardware line item with catalog number, quantity, list price, net price, lead time, vendor, and PO-ready grouping. Follows ISA-20, ANSI/ISA-5.4, NEMA 250, IEEE 315, and procurement best practices. Produces an XLSX with live formulas plus a static CSV mirror, plus a PO-ready vendor summary.
version: 1.0.0
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [bom, hardware, plc, panel, isa-20, nema-250, ieee-315, his]
    ported_from: claude-code/bom
---
# /bom — Hardware BOM Template Generator

**Invocation:** `/bom [project name]`

**Examples:**
- `/bom "NFK-DRYER-01 — North Fork Community Power Dryer"`
- `/bom "Blending Skid — Area 300"`
- `/bom "Wastewater Lift Station No. 4"`

---

## Purpose

Generate a complete, procurement-ready Hardware Bill of Materials that serves four audiences simultaneously:

1. **The controls engineer** — every catalog number verified, I/O count matches the I/O List, no missing components.
2. **The purchasing agent** — line items grouped by vendor/distributor, list price and net price columns, PO-ready summary.
3. **The project manager** — extended prices roll up to a total hardware budget, lead times flag the critical path against the project's FAT date.
4. **The customer/end user** — section totals match the quote, no surprise add-ons at delivery.

Standards alignment:
- **ANSI/ISA-20** — Specification forms for process instruments (instrument procurement data requirements)
- **ISA-5.4-1991** — Instrument loop diagrams; minimum procurement data per instrument
- **NEMA 250-2018** — Enclosure type classification (enclosure spec fields)
- **IEEE 315 / ANSI Y32.2** — Graphic symbols for electrical & electronics diagrams (reference designation system)
- **NFPA 79** — Electrical standard for industrial machinery; wiring and component marking requirements
- **IEC 62443-2-4** — Security requirements for IACS service providers; asset inventory data (only claimed when the optional 62443 column add-on is selected)
- Distributor and Rockwell ordering conventions researched live via Scrapling, scoped to the platform the user actually selected.

---

## Execution Flow

### MANDATORY: `AskUserQuestion` for ALL Questions

Never ask questions in plain text. Every question to the user goes through `AskUserQuestion` so they get proper UI controls, selectable options, and a structured experience. **This is a formal interview, not a chat.** Never assume a value the user did not provide — if a value is needed and was not answered, ask. Defaults are only acceptable when explicitly labeled `(Recommended)` inside an `AskUserQuestion` option, never silently in code or in pre-populated rows. Pre-populated rows are templates — every TBD or PLACEHOLDER must be visible in the output so the engineer can resolve it before issuing the BOM.

The interview runs in **four sequential rounds**. Complete each round fully before proceeding to the next; later rounds depend on earlier answers.

---

### Step 1 — Interview Round 1: Project Identity, Schedule, and Revision

```
AskUserQuestion([
  {
    question: "What is the BOM revision letter?",
    header: "BOM Rev",
    multiSelect: false,
    options: [
      { label: "A — first issue (Recommended for new projects)", description: "Use Rev A for the first issue of the BOM. Increment the letter on every change after first issue." },
      { label: "B / C / D / later", description: "Continuing an existing BOM. I will ask which letter in a follow-up." },
      { label: "Draft / pre-revision", description: "Internal working copy; do not issue to purchasing." }
    ]
  },
  {
    question: "What is the project's required-on-site / FAT date? (Used to flag critical-path lead times)",
    header: "FAT Date",
    multiSelect: false,
    options: [
      { label: "Within 8 weeks — aggressive", description: "Anything > 6-week lead time is critical path. Flag aggressively." },
      { label: "8–16 weeks — typical", description: "Anything > 12-week lead time is critical path." },
      { label: "16+ weeks — long lead", description: "Standard lead times mostly fit. Flag > 16 weeks." },
      { label: "Unknown / not yet set", description: "Use absolute >12 wk threshold; flag in the BOM that critical-path detection is approximate." }
    ]
  },
  {
    question: "What is the overall project type?",
    header: "Project Type",
    multiSelect: false,
    options: [
      { label: "Greenfield — new panel, new hardware", description: "Full BOM from scratch. Every line item is new procurement." },
      { label: "Retrofit / Upgrade — reusing some existing hardware", description: "BOM covers only new hardware. Reused items listed with Status=EXISTING — $0." },
      { label: "Expansion — adding I/O or devices to a running system", description: "BOM focuses on delta hardware. Existing infrastructure noted for reference." },
      { label: "Migration — PLC-5 / SLC to Logix replacement", description: "BOM replaces all controller hardware. Field devices typically reused — flag each." }
    ]
  },
  {
    question: "Capture the document header info. Customer / project number / engineer-of-record.",
    header: "Doc Header",
    multiSelect: false,
    options: [
      { label: "I'll type all three", description: "You will be prompted via Other for customer name, HIS project number, and engineer-of-record initials. Today's date is auto-filled." },
      { label: "Use placeholders for now — I'll fill in later", description: "Customer / Project# / EOR set to {{TBD}} in the BOM header. Must be resolved before issuing." }
    ]
  }
])
```

If the user picks "I'll type all three," follow up with a single `AskUserQuestion` (Other) to capture customer, project number, and engineer initials.

---

### Step 2 — Interview Round 2: Platform, Architecture, Power, and Enclosure

```
AskUserQuestion([
  {
    question: "What is the primary control platform for this project?",
    header: "Platform",
    multiSelect: false,
    options: [
      { label: "Rockwell ControlLogix / GuardLogix (1756)", description: "Full rack, I/O modules, EtherNet/IP. Premium pricing, longest lead times on specialty cards." },
      { label: "Rockwell CompactLogix 5380 / Compact GuardLogix (5069) (Recommended for new mid-range)", description: "Modern recommended platform for new mid-range work. Right-sized for skids, machines, small process. Good availability." },
      { label: "Rockwell CompactLogix 5370 (1769) / MicroLogix / Micro820", description: "Legacy CompactLogix or fixed-I/O budget platforms. OEM equipment or replacement-in-kind." },
      { label: "Other / Siemens / Beckhoff", description: "Non-Rockwell platform. I will use placeholder cat numbers and flag for hand-resolution." }
    ]
  },
  {
    question: "Is the controller architecture redundant or simplex?",
    header: "Redundancy",
    multiSelect: false,
    options: [
      { label: "Simplex — single controller (Recommended for most machines and skids)", description: "Default for OEM machines, skids, and most discrete projects. Drives PSU choice to 1756-PA72 / 1756-PB72." },
      { label: "Redundant — paired chassis with 1756-RM2", description: "Process plants, utilities, pharma. Drives PSU choice to 1756-PA75R / 1756-PB75R; doubles chassis, controller, and EN module quantities." }
    ]
  },
  {
    question: "What is the panel power source / voltage system?",
    header: "Voltage",
    multiSelect: false,
    options: [
      { label: "480VAC, 3-phase (Recommended for industrial)", description: "Standard plant power. Drives transformer sizing and main breaker selection." },
      { label: "240VAC, 1-phase or 3-phase", description: "Light commercial / small machine. Smaller transformer, smaller main." },
      { label: "120VAC, 1-phase", description: "Single-phase 120V. No control transformer; direct branch breaker for 24VDC supply." },
      { label: "24VDC bus only — externally powered", description: "Panel receives 24VDC from upstream source. No AC distribution in scope." }
    ]
  },
  {
    question: "What is the enclosure NEMA / IP rating?",
    header: "Enclosure",
    multiSelect: false,
    options: [
      { label: "NEMA 12 — indoor, dust-tight (Recommended for plant interior)", description: "Standard plant-floor indoor enclosure. Mild steel painted." },
      { label: "NEMA 4 — indoor/outdoor, rain-tight", description: "Outdoor or wash-down indoor. Mild steel painted." },
      { label: "NEMA 4X — corrosion-resistant", description: "Stainless or fiberglass. Wet, corrosive, food-and-beverage, or marine." },
      { label: "NEMA 7 / 9 — hazardous location", description: "Class I or Class II hazardous. Drives flameproof or purged enclosure selection." }
    ]
  }
])
```

After Round 2, ask one targeted follow-up if `NEMA 7/9` was selected:

```
AskUserQuestion([
  {
    question: "Hazardous-area classification (drives every field instrument and seal)?",
    header: "Haz Area",
    multiSelect: false,
    options: [
      { label: "Class I, Div 2 (most common)", description: "Flammable gas/vapor, abnormal-condition only. Most instruments available off-the-shelf." },
      { label: "Class I, Div 1", description: "Flammable gas/vapor, normal operation. Requires explosion-proof or intrinsically safe instruments." },
      { label: "Class II (combustible dust)", description: "Grain, coal, metal dust. Drives dust-ignition-proof enclosures." },
      { label: "Zone 0 / 1 / 2 (IEC)", description: "European / IEC classification. Drives ATEX or IECEx instrument selection." }
    ]
  }
])
```

If `Other / Siemens / Beckhoff` was selected for platform, ask the user to type the platform name (Other) and proceed with placeholder cat numbers in Section 1.

---

### Step 3 — Interview Round 3: I/O Scope, Hardware Categories, and Spares

```
AskUserQuestion([
  {
    question: "What is the rough project I/O count? (Helps size the BOM sections)",
    header: "I/O Count",
    multiSelect: false,
    options: [
      { label: "Small — under 100 I/O points", description: "Single CompactLogix or MicroLogix. One panel. One HMI terminal." },
      { label: "Medium — 100 to 500 I/O points", description: "ControlLogix or CompactLogix with remote I/O drops. Multiple panels possible." },
      { label: "Large — 500+ I/O points", description: "Full ControlLogix rack(s), multiple remote I/O drops, distributed architecture." },
      { label: "Unknown — generate template with all sections", description: "Generate all BOM sections with placeholder rows for each category." }
    ]
  },
  {
    question: "Which major hardware categories are in scope for this BOM? (select all that apply)",
    header: "HW Categories",
    multiSelect: true,
    options: [
      { label: "PLC / Controller Hardware", description: "Controller, chassis, I/O modules, power supplies, communication modules." },
      { label: "HMI / Operator Interface", description: "PanelView terminals, industrial monitors, operator pushbutton stations." },
      { label: "Drives & Motor Control", description: "VFDs, soft starters, motor circuit protectors, contactors." },
      { label: "Field Instruments", description: "Transmitters, switches, analyzers, control valves, actuators." }
    ]
  },
  {
    question: "Which infrastructure / panel categories are in scope? (select all that apply)",
    header: "Panel / Infra",
    multiSelect: true,
    options: [
      { label: "Enclosure & Panel Hardware", description: "NEMA enclosures, back panels, cable duct, DIN rail, labels, hardware." },
      { label: "Power Distribution & Protection", description: "Transformers, circuit breakers, fuses, disconnect switches, surge suppressors, UPS." },
      { label: "Network & Communications Hardware", description: "Managed switches, fiber converters, EtherNet/IP taps, wireless APs, cellular routers." },
      { label: "Safety Devices", description: "E-stop buttons, safety relays, light curtains, interlocks, safety PLCs, guardlocking switches." }
    ]
  },
  {
    question: "Spare-parts coverage level?",
    header: "Spares",
    multiSelect: false,
    options: [
      { label: "Basic — 1 of each card type (Recommended for OEM machines)", description: "1 spare DI/DO/AI/AO module per type, 1 set of fuses, 2 spare E-stops, 4 spare pilot lights. Adequate for OEM machines." },
      { label: "Enhanced — basic + critical instruments + extra terminal bases", description: "Adds 1 spare per critical-loop instrument and 4 extra terminal bases. Recommended for process plants and 24/7 operations." },
      { label: "SIL-graded — enhanced + redundancy spares + safety relay spares", description: "Adds 1 spare safety relay per redundancy group and 1 spare safety I/O per channel-bank-of-16. Required for SIL-rated systems." },
      { label: "None / customer-specified", description: "Section 9 included as blank for customer to define quantities." }
    ]
  }
])
```

---

### Step 4 — Interview Round 4: Pricing, Distributor, Add-On Columns, and Output

```
AskUserQuestion([
  {
    question: "What is the pricing convention for this BOM?",
    header: "Pricing Mode",
    multiSelect: false,
    options: [
      { label: "Internal cost only — Net = our cost from distributor (Recommended for internal estimating)", description: "List_Price + Discount_Pct → Net_Unit_Price = our cost. No customer-facing markup. Pricing is HIS-internal." },
      { label: "Customer-facing — both cost and sell columns", description: "Adds Sell_Unit_Price and Sell_Ext_Price columns alongside Net. The same BOM can be filtered to internal or customer view." },
      { label: "Budget estimate — list × 0.65, marked ESTIMATE", description: "Rough order-of-magnitude only. Not for PO use. Status = ESTIMATE on every priced row." },
      { label: "No pricing — catalog numbers and quantities only", description: "Omit all price columns. BOM is a procurement specification only." }
    ]
  },
  {
    question: "Which Rockwell-authorized distributor will quote this project?",
    header: "Distributor",
    multiSelect: false,
    options: [
      { label: "Wesco / Rexel", description: "National distributor, multi-region coverage." },
      { label: "Border States / Stanley / Kendall", description: "Regional Rockwell distributor — common in central/midwest US." },
      { label: "Other — I'll type the name", description: "Type the distributor name. Will appear in Vendor column and PO Summary." },
      { label: "Not yet assigned", description: "Vendor column = TBD on every Rockwell line. Resolve before issuing PO." }
    ]
  },
  {
    question: "Optional column add-ons (select all that apply)?",
    header: "Add-On Cols",
    multiSelect: true,
    options: [
      { label: "IEC 62443 asset inventory — MAC, IP, Sec_Zone, Vendor_Support_Expiry", description: "Adds 4 columns to controller, HMI, drive, and switch lines. Required if BOM doubles as 62443 asset inventory." },
      { label: "Procurement lifecycle dates — PO_Date, Ack_Date, Promised_Ship, Actual_Ship, Received_Date", description: "Adds 5 columns. Use when the BOM doubles as the receiving log." },
      { label: "Trade compliance — COO, TAA_Compliant, HTS_Code", description: "Adds 3 columns. Required for federal projects (Buy America / TAA / DFARS) and EU exports (CE / UKCA)." }
    ]
  },
  {
    question: "What output files do you need?",
    header: "Output",
    multiSelect: false,
    options: [
      { label: "XLSX with live formulas + CSV mirror + PO Summary (Recommended)", description: "Primary deliverable is BOM-{project}-Rev{rev}.xlsx with =SUMIF formulas. Static CSV mirror with all values pre-computed. PO Summary in markdown." },
      { label: "XLSX + PO Summary only", description: "Skip CSV mirror." },
      { label: "Static CSV + PO Summary only", description: "No XLSX. All totals pre-computed by Claude — no formulas. Cheap and portable." }
    ]
  }
])
```

---

### Step 5 — Research (Scoped to the Platform Selected)

After the interview, fetch the product-family page for the platform answered in Round 2 — not a blind URL list. Cite at least one fact from the fetched page (current series letter, current firmware revision, lead-time advisory) in the generated BOM Notes column for the corresponding rows.

```
# ControlLogix selected →
mcp__scrapling__fetch("https://literature.rockwellautomation.com/idc/groups/literature/documents/sg/1756-sg001_-en-p.pdf")

# CompactLogix 5380 / 5069 selected →
mcp__scrapling__fetch("https://literature.rockwellautomation.com/idc/groups/literature/documents/sg/5069-sg001_-en-p.pdf")

# Other / Siemens / Beckhoff →
# skip Rockwell fetch; proceed with placeholders
```

If Scrapling errors or returns empty, proceed with built-in catalog reference. Do NOT block on failed fetches.

---

### Step 6 — Nexus + I/O List Data Pull (Optional, Best-Effort)

In order of preference:

1. **Local I/O list file.** Glob the project save location for `IO-LIST-*.csv`. If found, parse it to count modules per type (DI/DO/AI/AO/safety) and pre-fill Section 1 quantities. Cite the file path in the BOM Notes for the affected rows.
2. **Nexus.** If the project is indexed:
   ```
   mcp__nexus__plant_summary()
   mcp__nexus__list_documented_plcs()
   mcp__nexus__generate_io_inventory(plc_id: <selected PLC>)
   ```
   Use the inventory to estimate module quantities and flag safety I/O / Modbus / DeviceNet comm needs.
3. **Neither.** Pre-populate Section 1 with placeholder quantities `TBD`.

If both local I/O list AND Nexus exist, prefer the local file and flag any discrepancy in the BOM Notes (`⚠️ NEXUS / I/O LIST DISAGREE — VERIFY`).

Mark every Nexus- or I/O-list-derived quantity with status `PLACEHOLDER` and a note `Source: <io-list-filename>` or `Source: NEXUS` in the Notes column.

---

### Step 7 — Generate the BOM

#### BOM Sections

| Section | Typical Contents |
|---|---|
| **1 — PLC & Controller Hardware** | Controller, chassis, I/O modules (DI/DO/AI/AO/RTD/TC/HSC/Safety), comm modules, power supply, firmware media, terminal bases |
| **2 — HMI & Operator Interface** | PanelView Plus terminals, industrial PCs, monitors, operator stations, audio/visual alarm devices |
| **3 — Drives & Motor Control** | VFDs, soft starters, motor circuit protectors, contactors, overload relays, motor disconnects |
| **4 — Safety Devices** | E-stop buttons, safety relays, light curtains, laser scanners, gate switches, rope pulls, safety I/O modules, safety PLCs |
| **5 — Field Instruments** | Pressure / flow / level / temp transmitters, control valves, actuators, positioners, analyzers |
| **6 — Network & Communications** | Stratix managed switches, fiber media converters, wireless APs, cellular routers, EtherNet/IP taps, patch cables |
| **7 — Power Distribution & Protection** | Control transformer, main disconnect, branch breakers, 24VDC supplies, UPS, surge suppressors |
| **8 — Enclosure & Panel Hardware** | NEMA enclosures, back panels, cable duct, DIN rail, terminal blocks, ferrules, labels, hardware |
| **9 — Spare Parts** | Per the spare-parts level chosen in Round 3 (Basic / Enhanced / SIL-graded / None) |
| **10 — Miscellaneous & Consumables** | Conduit fittings, grounding lugs, wire loom, heat shrink, cable markers, panel labels, touch-up paint |

Each section is introduced by a header row (`Item_Type = SECTION`) and closed by a subtotal row (`Item_Type = SUBTOTAL`).

---

#### Column Set — Core (Always Present)

| # | Column | Notes |
|---|---|---|
| 1 | **Row#** | Sequential. Never renumber after issue. |
| 2 | **Section** | 1–10. `SECTION` for header rows. |
| 3 | **Item_Type** | `HARDWARE`, `SECTION`, `SUBTOTAL`, `TOTAL`, `ASSEMBLY` (parent kit), `KIT_ITEM` (child of an ASSEMBLY), `SPARE`. **Pure structural — does not encode procurement state.** |
| 4 | **Manufacturer** | Full mfr name: `Rockwell Automation`, `Phoenix Contact`, `Weidmuller`, `Pepperl+Fuchs`, `Cognex`. |
| 5 | **Cat_No** | Exact catalog/part number with all dashes and series codes (e.g., `1756-L85EP`). |
| 6 | **Series** | Series letter (e.g., `D`). Required on Logix controllers and major I/O modules; `--` on items without a series. |
| 7 | **FW_Required** | Firmware revision required for compatibility (e.g., `34.011`). Required on controllers and managed network gear. |
| 8 | **Description** | Plain-English description with project callout: `ControlLogix 5580 Controller, 20MB (Main PLC — Area 100)`. |
| 9 | **Qty** | Integer. **Must match UOM units.** If UOM=`PK`, Qty is in packs (not individual pieces). |
| 10 | **UOM** | `EA`, `FT`, `M`, `LT`, `PR`, `PK`. |
| 11 | **Pack_Qty** | Number of individual pieces per UOM. `1` for individually-priced items, `50` for terminal-block packs, `500` for ferrules, `2` for 2-meter DIN/duct sticks. **Quality check enforces Qty × Pack_Qty matches the engineer's intended count.** |
| 12 | **List_Price** | Mfr published list per UOM (USD). `TBD` if unknown. `0.00` if Status=`EXISTING`. |
| 13 | **Discount_Pct** | Distributor discount off list (e.g., `35.0`). `TBD` for purchasing. |
| 14 | **Net_Unit_Price** | `List_Price × (1 − Discount_Pct/100)`. Internal cost. `ESTIMATE` for budget mode. |
| 15 | **Ext_Price** | `Qty × Net_Unit_Price`. Auto-computed in XLSX; pre-computed numeric in CSV mirror. |
| 16 | **Lead_Time_Wks** | Weeks ARO. `⚠️ CRITICAL PATH` if `Lead_Time + 2` ≥ weeks-to-FAT (from Round 1). `⚠️ LONG LEAD` if absolute > 12 wk and FAT date unknown. |
| 17 | **Vendor** | Distributor / direct-buy. Pre-filled from Round 4 distributor answer for Rockwell lines. |
| 18 | **Quote_No** | Distributor quote number. `TBD` if unquoted. |
| 19 | **PO_Line** | PO line number. `TBD` until PO issued. |
| 20 | **Status** | **Procurement lifecycle only.** `PLACEHOLDER`, `ESTIMATE`, `QUOTED`, `ORDERED`, `ACKD`, `SHIPPED`, `BACKORDER`, `RECEIVED`, `EXISTING`, `RETURNED`. |
| 21 | **Sub_OK** | Substitution authority: `Y` (any equiv OK), `N` (exact only), `EOR` (engineer-of-record approval required). |
| 22 | **Rev** | `A`–`Z`. Increment when this row changes after first issue. |
| 23 | **Notes** | Free text: configurator decode for option-string cat#'s, alternate cat#, voltage option, data source citation, anything that doesn't fit a column. |

#### Column Set — Optional Add-Ons

Add the column groups the user selected in Round 4. Add columns only to applicable sections (e.g., 62443 columns appear on Section 1, 2, 3, 6 lines but not on terminal blocks).

**Customer-Facing Pricing (if "Customer-facing" selected in Round 4):**
- `Sell_Unit_Price` — our sell to customer (Net × margin).
- `Sell_Ext_Price` — `Qty × Sell_Unit_Price`.

**IEC 62443 Asset Inventory:**
- `MAC_Addr` — populated at commissioning.
- `IP_Addr` — populated at network-design phase.
- `Sec_Zone` — security zone per the project's zones-and-conduits diagram.
- `Vendor_Support_Expiry` — date support contract / TechConnect expires.

**Procurement Lifecycle Dates:**
- `PO_Date`, `Ack_Date`, `Promised_Ship`, `Actual_Ship`, `Received_Date`.

**Trade Compliance:**
- `COO` — country of origin (ISO-3166 alpha-2: `US`, `CN`, `MX`, `DE`).
- `TAA_Compliant` — `Y` / `N` / `Unknown`.
- `HTS_Code` — Harmonized Tariff Schedule code (10-digit US HTS).

---

### Step 8 — Pre-Populate Rows (Platform-Aware Defaults)

Pre-populate each in-scope section with the most common line items for the platform and architecture answered. Status = `PLACEHOLDER` until the engineer confirms cat# and quantity.

#### Section 1 — PLC & Controller Hardware (Rockwell ControlLogix example)

- 1756 chassis sized to estimated I/O count (4-slot / 7-slot / 10-slot / 13-slot / 17-slot).
- **Power supply (defaults by Round 2 redundancy answer):**
  - **Simplex:** `1756-PA72` (120/240VAC) or `1756-PB72` (24VDC). **These are the defaults.**
  - **Redundant:** `1756-PA75R` or `1756-PB75R` paired across both chassis.
- ControlLogix or GuardLogix controller sized to ~60% memory utilization at project start.
- 1756-EN2TR or 1756-EN4TR EtherNet/IP bridge module (DLR-capable).
- DI: `1756-IB16` (24VDC, sinking).
- DO: `1756-OB16E` (24VDC, sourcing, electronically fused).
- AI: `1756-IF16` (16-channel, 4-20mA / 0-10VDC).
- AO: `1756-OF8H` (8-channel, HART-enabled).
- Safety I/O if GuardLogix selected: `1791ES-IB16`.
- **`1756-TBCH` terminal base — one per I/O module.** Quality check enforces 1:1 match.
- Firmware media SD card if required for the controller series.

#### Section 2 — HMI & Operator Interface

- PanelView Plus 7 terminal sized 7" / 10" / 12" / 15" by scope.
- **Configurator decode required.** PanelView cat# format: `2711P-T{size}{C/W}{V/D}{D/A}{S/N}`. Decode each option position in Notes (e.g., `Size=10in, C=color, V=24VDC, D=Std, S=Stainless`). **Quality check:** any cat# starting with `2711P-T` must have a decoded option line in Notes.

#### Section 3 — Drives & Motor Control

- PowerFlex 525 (≤30HP): `25B-D{amps}N{frame}{options}` — fan/pump duty.
- PowerFlex 755 (≥30HP): `20G-{amps}-{options}` — heavy duty / high-performance.
- **Configurator decode required.** Decode each option position in Notes (`HP=5, V=480, Brake=N, Comm=ENet, Encl=NEMA1, Ctrl=Std, Coat=N`). **Quality check:** any cat# starting with `25B-` or `20G-` must have a decoded option line in Notes.
- Motor Circuit Protector: `140MG` series (size to motor FLA × 1.15 minimum).
- Bulletin 100 contactor (bypass contactor if VFD bypass is required).

#### Section 4 — Safety Devices

- E-stop button: `800F-MT44` (40mm, guarded, 2NC contacts) — one per E-stop location.
- Safety relay: `440R-S13R2` (Cat 3) or `440R-D22R2` (Cat 4).
- Light curtain: `GuardShield 450L` series if in scope.
- Safety I/O: `1791ES` modules if GuardLogix is the platform.

#### Section 5 — Field Instruments

Leave rows as `PLACEHOLDER` with description only. Pre-populate one row per instrument type mentioned in scope (pressure / flow / level / temp / analyzer / control valve). If hazardous-area was selected in Round 2, append a Notes flag: `⚠️ HAZ-AREA — VERIFY ENCLOSURE / I.S. RATING`.

#### Section 6 — Network & Communications

- Stratix 5400 (`1783-HMS{ports}...`) — preferred for EtherNet/IP DLR rings.
- Stratix 5700 (`1783-BMS{ports}...`) — non-DLR.
- **Configurator decode required.** Decode port count, fiber/copper, conformal coat, redundant power in Notes. **Quality check:** any cat# starting with `1783-` must have a decoded option line in Notes.

#### Section 7 — Power Distribution & Protection

- Control transformer sized to panel load + 25% spare (skip if Round 2 voltage = 24VDC bus only).
- 24VDC supply: `1606-XLP{wattage}E` (AB) or Phoenix `QUINT-PS`.
- Circuit breakers: list by circuit (PLC power, 24VDC PS, HMI, field power, spare).
- Main disconnect: rated to NEMA enclosure type from Round 2.
- UPS if 24VDC continuity required: `1606-XLS{wattage}` or APC industrial equivalent.

#### Section 8 — Enclosure & Panel Hardware

- Main enclosure cat# format keyed to NEMA rating from Round 2:
  - NEMA 12 / 4: Hoffman `A-{H}H{W}W{D}D{LP}` painted steel.
  - NEMA 4X: Hoffman `CSD` stainless or `LSF` fiberglass.
  - NEMA 7/9: appropriate flameproof / purged enclosure family — flag in Notes.
- DIN rail: `199-DR1` (AB) / `NS 35/7.5` (Phoenix). UOM=`M` or `EA`, `Pack_Qty=2` for 2m sticks.
- Cable duct: `1494U-{size}`. UOM=`EA`, `Pack_Qty=2` for 2m sticks.
- Terminal blocks: `1492-W{size}` / Phoenix `UK{size}`. UOM=`PK`, `Pack_Qty=50`.
- Wire ferrules: sized to wire gauge. UOM=`PK`, `Pack_Qty=500`.
- Panel labels: adhesive or engraved per NEMA rating.

#### Section 9 — Spare Parts (Conditional on Round 3 spares level)

| Item | Basic | Enhanced | SIL-graded | Rule |
|---|---|---|---|---|
| Spare DI/DO/AI/AO module per type | 1 EA | 1 EA | 1 EA | 1 spare per card type minimum |
| Spare terminal bases (1756-TBCH) | 0 EA | 4 EA | 4 EA | Match spare card count or buffer |
| Spare 24VDC PS fuses | 1 PK | 1 PK | 1 PK | Match installed rating |
| Spare 5A branch fuses | 1 PK | 1 PK | 1 PK | Standard instrument loop fuses |
| Spare E-stop buttons | 2 EA | 2 EA | 2 EA | 1 per 10 installed, min 2 |
| Spare pilot lights (24VDC) | 4 EA | 4 EA | 4 EA | Common consumable |
| Spare VFD I/O fuses | 1 PK | 1 PK | 1 PK | Per VFD model installed |
| Spare critical-loop instruments | — | 1 per critical tag | 1 per critical tag | Per project criticality matrix |
| Spare safety relay per redundancy group | — | — | 1 EA | SIL only |
| Spare safety I/O per channel-bank-of-16 | — | — | 1 EA | SIL only |

---

### Step 9 — Generate Output Files

#### Primary: XLSX with Live Formulas (if selected in Round 4)

Filename: `BOM-{project-slug}-Rev{rev}.xlsx`

Generate via Python `openpyxl` from a Bash heredoc. Requirements:

- Numeric-typed cells for `List_Price`, `Discount_Pct`, `Net_Unit_Price`, `Ext_Price`, `Sell_Unit_Price`, `Sell_Ext_Price`, `Qty`, `Pack_Qty`, `Lead_Time_Wks`.
- Live formulas:
  - `Net_Unit_Price = List_Price * (1 - Discount_Pct/100)`
  - `Ext_Price = Qty * Net_Unit_Price`
  - `Sell_Ext_Price = Qty * Sell_Unit_Price` (if customer-facing pricing selected)
  - Per-section subtotal: `=SUMIF(Section_column, <section_number>, Ext_Price_column)`
  - Grand total: `=SUM(<section subtotals>)`
- Conditional formatting: `Lead_Time_Wks` cells turn red when flagged `⚠️ CRITICAL PATH`; `Status` cells turn yellow when `PLACEHOLDER`.
- Header row frozen; column widths sized to typical content.

#### Secondary: Static CSV Mirror (if selected)

Filename: `BOM-{project-slug}-Rev{rev}.csv`

- RFC 4180 with double-quote wrapping on text fields.
- **Every numeric cell pre-computed by Claude — no formulas, no `=SUM` text in Notes, ever.**
- `--` for null/not-applicable.
- `TBD` for fields awaiting purchasing input.
- Section headers: `Item_Type=SECTION`, all numeric columns = `--`.
- Subtotal rows: `Item_Type=SUBTOTAL`, Qty=`--`, Ext_Price = pre-computed sum.
- Grand total row: `Item_Type=TOTAL` at the bottom.

#### PO Summary Markdown

Filename: `BOM-{project-slug}-Rev{rev}-PO-SUMMARY.md`

```markdown
# Purchase Order Summary — {Project Name}
**BOM Revision:** {rev} **Date:** {date} **Prepared by:** Hunt Integrative Solutions LLC
**Project FAT Date:** {fat_date} **Critical-path threshold:** {weeks-to-FAT − 2} weeks

## How to Use This Summary
Each section below corresponds to one Purchase Order. Copy the line items into your
PO system. The Cat_No and Description must appear exactly as listed — any variation
can cause the wrong item to ship. Verify lead times before issuing; long-lead and
critical-path items (⚠️) should be ordered first regardless of project schedule.

---

## PO 1 — Rockwell Automation (via {distributor from Round 4})

**Distributor Quote #:** {quote number or TBD}
**Estimated Lead Time:** {X–Y weeks ARO}

| PO Line | Cat No | Description | Qty | UOM | List Price | Net Price | Ext Price |
|---|---|---|---|---|---|---|---|
| 1 | ... | ... | ... | ... | ... | ... | ... |

**PO 1 Subtotal: ${subtotal}**

---

## PO 2 — {Next Vendor}
...

---

## Grand Total

| Line | Amount |
|---|---|
| Hardware subtotal (sum of POs) | ${hw_subtotal} |
| Freight allowance | TBD — purchasing fills |
| Sales tax (or tax-exempt cert on file) | TBD — purchasing fills |
| Contingency (recommend 5%) | ${contingency} |
| **PROJECT HARDWARE TOTAL** | **${total}** |

> Hardware-only BOM. Engineering, panel-build labor, programming, commissioning, and
> startup are estimated separately.

---

## Critical-Path Items ⚠️

Items where `Lead_Time + 2 weeks ≥ weeks-to-FAT` ({fat_date}) — ORDER THESE FIRST:

| Cat No | Description | Lead Time | Vendor | Action Required |
|---|---|---|---|---|
| ... | ... | ## wks | ... | Order immediately — drives FAT date |

---

## PO Checklist

Before issuing any PO:
- [ ] All `PLACEHOLDER` rows resolved to confirmed cat numbers
- [ ] All `TBD` prices replaced with quoted or list pricing
- [ ] Configurator-decoded option strings verified against distributor quote
- [ ] Distributor discount applied and net prices verified
- [ ] Ship-to confirmed: panel shop vs. site
- [ ] Delivery terms confirmed: FOB Origin vs. FOB Destination
- [ ] Tax-exempt certificate on file with distributor (if applicable)
- [ ] Critical-path items ordered first
- [ ] Spares section reviewed and approved by customer
- [ ] BOM revision matches project revision
```

---

### Step 10 — Choose Save Location (Always Ask — Never Default to Desktop)

Use `AskUserQuestion`. Project folder and user-typed path come first; current working directory next; Desktop last and never marked Recommended.

```
AskUserQuestion([
  {
    question: "Where should the BOM file(s) be saved?",
    header: "Save Location",
    multiSelect: false,
    options: [
      { label: "Active project folder — I'll type the path", description: "WSL path (e.g., /mnt/c/Users/HIS Controls/Projects/{project}/). Recommended for most projects so the BOM lives next to the I/O list, FDS, and other deliverables." },
      { label: "Same folder as other project documents — I'll type the path", description: "WSL path. Use when project documents are on a network share or customer-specific folder." },
      { label: "Current working directory", description: "Use the current shell working directory." },
      { label: "Desktop", description: "/mnt/c/Users/HIS Controls/Desktop/ — Windows Desktop, accessed via WSL. Use only for one-off / scratch BOMs." }
    ]
  }
])
```

If the user picks a "type the path" option, follow up with `AskUserQuestion` (Other) to capture the WSL path. Confirm the path resolves before writing.

---

### Step 11 — Mandatory Quality Check Before Saving

Before writing any file, verify:

1. **Cat# format check:** Every Rockwell cat# must include series dashes (`1756-IB16`, not `1756IB16`). Flag missing-dash rows: `⚠️ CAT# FORMAT — VERIFY`.
2. **Configurator decode check:** Every cat# starting with `25B-`, `20G-`, `2711P-T`, or `1783-` must have a decoded option line in Notes. Flag missing decodes: `⚠️ CONFIGURATOR DECODE MISSING`.
3. **Series / firmware presence check:** Every controller and major I/O module must have non-`--` `Series` and `FW_Required`. Flag gaps.
4. **Terminal base check:** Every 1756 I/O module must have a corresponding `1756-TBCH` (or `TBNH`/`TBNF`). Count modules vs. terminal bases. Mismatch: `⚠️ TERMINAL BASE MISSING FOR ROW #N`.
5. **Power supply check:** Every chassis section must have at least one PSU. **In simplex mode, flag any `1756-PA75R` / `1756-PB75R` as `⚠️ REDUNDANT PSU IN SIMPLEX BOM — OVER-SPEC`.**
6. **Pack-quantity check:** For every row with `Pack_Qty > 1`, verify `Qty` is in UOM units (`UOM=PK` and Qty is in packs, **or** `UOM=EA` and Qty is a multiple of `Pack_Qty`). Flag mismatches: `⚠️ PACK QTY MISMATCH`.
7. **Spare-parts coverage check:** Spare rows present per the level chosen in Round 3. Flag gaps.
8. **Critical-path check:** For every row, compute weeks-to-FAT (from Round 1) − 2. Items where `Lead_Time_Wks` ≥ that threshold get `⚠️ CRITICAL PATH`. Items > 12 weeks with no FAT date get `⚠️ LONG LEAD`.
9. **Pricing band check:** Flag `⚠️ UNUSUAL DISCOUNT — VERIFY` if `Net_Unit_Price` is outside the band `0.45 × List` to `0.75 × List` for Rockwell standard items. Tighter band than the previous rule, fewer false alarms.
10. **Distributor sanity check:** Every Rockwell-mfr line must have a non-generic Vendor (the Round 4 distributor or a quoted equivalent). `Rockwell Distributor` literal is a fail.

Report the quality check summary at the end of your response:

```
BOM Quality Check
─────────────────────────────────
Total line items:               ###
Sections populated:             # of 10
PLACEHOLDER rows:               ###  ← must be resolved before ordering
ESTIMATE prices:                ###  ← not for PO — require formal quotes
Critical-path items:            ###  ← listed in PO Summary
Long-lead items (>12 wk):       ###
Configurator decode gaps:       0    ← or list rows
Series / firmware gaps:         0    ← or list rows
Pack-qty mismatches:            0    ← or list rows
Missing terminal bases:         0    ← or list rows
Missing power supplies:         0    ← or list chassis
Redundant PSU in simplex BOM:   0    ← over-spec flag
Spare parts coverage:           OK   ← or list missing spares
Cat# format warnings:           0    ← or list rows
Pricing band warnings:          0    ← or list rows
Distributor not assigned:       0    ← or list rows
BOM Hardware Total:             $###,###  (ESTIMATE) or (QUOTED)
```

---

### Step 12 — Closing Statement

After saving all files, provide:

1. Exact output filename(s) and full WSL path.
2. The quality check summary block above.
3. **Next steps block:**

```
Next Steps
──────────────────────────────────────────────────────────────────────
1. RESOLVE PLACEHOLDERS — Every PLACEHOLDER row must be confirmed with
   a real catalog number before the BOM is issued to purchasing.

2. RESOLVE CONFIGURATOR DECODES — Every PowerFlex / PanelView / Stratix
   cat# must be confirmed against the manufacturer's configurator or
   the distributor's quote. A wrong digit ships the wrong drive.

3. GET FORMAL QUOTES — ESTIMATE pricing is ±30% and not suitable for
   customer quotes or POs. Send the cat# list to {distributor from Round 4}
   and request a formal written quote.

4. ORDER CRITICAL-PATH ITEMS FIRST — Items marked ⚠️ CRITICAL PATH or
   ⚠️ LONG LEAD drive the project schedule. Issue those POs before
   finalizing panel shop drawings.

5. CONFIRM SHIP-TO — Panel shop vs. site changes freight terms and
   insurance responsibility. Confirm before PO issuance.

6. RECONCILE WITH I/O LIST — Every I/O module in Section 1 should
   correspond to an assigned slot in the I/O List. If /io-list output
   was not auto-ingested in Step 6, run /io-list and reconcile.

7. REVISION CONTROL — This BOM is Rev {rev}. Any change to cat#, qty,
   or price after first issue must increment the Rev letter and be
   noted in the revision block. Purchasing must never order from a
   superseded revision.

8. CUSTOMER APPROVAL — Get customer sign-off on the BOM before
   ordering. Restocking fees for returned Rockwell hardware are
   typically 15–25% of list price.
```

---

## Built-In Catalog Number Reference (Rockwell ControlLogix — Common Items)

> **Series letters and firmware revisions in this table reflect typical-as-of-cutoff values.** Always cross-check against the Rockwell PCDC and the live distributor system before issuing. Cite the live source in the BOM Notes for controllers and managed network gear.

| Cat No | Description | Notes |
|---|---|---|
| **Controllers** | | |
| 1756-L81EP | ControlLogix 5580, 3MB | Entry-level; small systems |
| 1756-L83EP | ControlLogix 5580, 8MB | Medium systems; most common |
| 1756-L85EP | ControlLogix 5580, 20MB | Large systems; motion-capable |
| 1756-L8SP | GuardLogix 5580, 10MB | Safety controller; same chassis as CLX |
| 5069-L320ER | CompactLogix 5380, 2MB | Modern mid-range — recommended for new |
| 5069-L340ER | CompactLogix 5380, 4MB | Modern mid-range — recommended for new |
| **Chassis** | | |
| 1756-A4 / A7 / A10 / A13 / A17 | ControlLogix chassis sizes | A7 most common (1 PS + 6 modules) |
| **Power Supplies** | | |
| **1756-PA72** | ControlLogix PS, 120/240VAC, 75W | **Default for simplex chassis** |
| **1756-PB72** | ControlLogix PS, 24VDC, 75W | **Default for simplex chassis on 24VDC bus** |
| 1756-PA75R | ControlLogix redundant PS, 120/240VAC | **Use only with redundant chassis pairs** |
| 1756-PB75R | ControlLogix redundant PS, 24VDC | **Use only with redundant chassis pairs** |
| **EtherNet/IP Modules** | | |
| 1756-EN2TR | EtherNet/IP, 2-port, DLR | Required for DLR ring topology |
| 1756-EN4TR | EtherNet/IP, 2-port, DLR, CIP Security | Modern replacement for EN2TR on new projects |
| **DI Modules** | | |
| 1756-IB16 | 16-pt 24VDC DI, sinking | Standard discrete input |
| 1756-IB32 | 32-pt 24VDC DI, sinking | High-density; 2 terminal bases |
| 1756-IA16 | 16-pt 120VAC DI | AC input circuits |
| **DO Modules** | | |
| 1756-OB16E | 16-pt 24VDC DO, sourcing, efused | Most common; electronic fusing |
| 1756-OB32 | 32-pt 24VDC DO, sourcing | High-density; no e-fuse |
| 1756-OA16 | 16-pt 120VAC DO, triac | AC output circuits |
| **AI Modules** | | |
| 1756-IF16 | 16-ch analog input, 4-20mA / 0-10V | Standard analog input |
| 1756-IF8H | 8-ch analog input, HART | HART passthrough to asset management |
| **AO Modules** | | |
| 1756-OF8H | 8-ch analog output, HART | Standard AO with HART |
| 1756-OF4 | 4-ch analog output | Budget AO; no HART |
| **RTD / TC Modules** | | |
| 1756-IR6I | 6-ch RTD input | Pt100, Ni120, Cu10 |
| 1756-IT6I2 | 6-ch thermocouple input | J, K, T, E, R, S, B types |
| **Terminal Bases** | | |
| 1756-TBCH | 36-pin removable terminal base | Standard; one per I/O module |
| 1756-TBNH | 36-pin non-removable terminal base | Use where wiring must not be accidentally disconnected |
| **Safety Modules (GuardLogix)** | | |
| 1791ES-IB16 | 16-pt safety DI, CompactBlock | Remote safety I/O — most common |
| 1791ES-OB8EP | 8-pt safety DO, CompactBlock | Remote safety output |
| 1756-IB16S | 16-pt safety DI, ControlLogix | In-chassis safety input |
| **HMI Terminals** | | |
| 2711P-T7C22D8S | PanelView Plus 7, 7", color, 24VDC, stainless | Standard machine HMI |
| 2711P-T10C22D8S | PanelView Plus 7, 10", color, 24VDC, stainless | Process HMI — recommended minimum for process |
| 2711P-T12W22D8S | PanelView Plus 7, 12" wide, color, 24VDC | Wide-format; good for P&ID-style displays |
| **VFDs** | | |
| 25B-D010N104 | PowerFlex 525, 480VAC, 10A (5HP) | Fan/pump duty; most common |
| 20G-1P1-AA — | PowerFlex 755, 480VAC (size to motor) | Heavy duty, high-performance |
